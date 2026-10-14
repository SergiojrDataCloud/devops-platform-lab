# Troubleshooting — K3s, NGINX, Traefik e Ingress

## 1. Estado do cluster

```bash
systemctl status k3s --no-pager
k3s kubectl get nodes -o wide
k3s kubectl get pods -A
```

O nó deve estar `Ready`; os pods principais, incluindo Traefik, devem estar saudáveis.

## 2. Deployment, Service e endpoints

```bash
k3s kubectl get deployment nginx
k3s kubectl get pods -o wide
k3s kubectl get svc nginx
k3s kubectl get endpoints nginx
k3s kubectl get endpointslice -l kubernetes.io/service-name=nginx
k3s kubectl describe service nginx
k3s kubectl describe deployment nginx
```

No laboratório validado, o Service tinha um endpoint do Pod na porta 80 e o acesso direto ao IP do Pod retornou `HTTP/1.1 200 OK`.

## 3. Ingress e classe

```bash
k3s kubectl get ingress nginx-ingress -o wide
k3s kubectl get ingressclass
k3s kubectl describe ingress nginx-ingress
```

Confirme classe `traefik`, host `nginx.lab`, path `/`, backend `nginx:80` e ausência de eventos de erro.

## 4. Testes em camadas

### A. Pod diretamente na VM

```bash
curl -i http://IP_DO_POD:80/
```

### B. Ingress localmente na VM

```bash
curl -i -H 'Host: nginx.lab' http://127.0.0.1/
```

### C. Porta 80 pelo Windows

```powershell
Test-NetConnection IP_REAL_DA_VM -Port 80
curl.exe -v -i --resolve nginx.lab:80:IP_REAL_DA_VM http://nginx.lab/
```

Esperado: `TcpTestSucceeded : True` e `HTTP/1.1 200 OK`. Use o IPv4 da interface da VM que o Windows alcança.

## 5. Erros comuns

| Sintoma | Verificações |
|---|---|
| Pod não está `Running` | `k3s kubectl describe pod ...` e `k3s kubectl logs ...` |
| Service sem endpoint | selector/labels do Service e estado do Deployment |
| `404` do Traefik | Host enviado, classe do Ingress, regra/path e IP/porta de destino |
| `404` de outro servidor | confirmar IP/porta e cabeçalho `Server` |
| Timeout ou conexão recusada | IP, conectividade, firewall e porta 80 |
| Funciona na VM, mas não no Windows | IPv4 correto da VM e `Test-NetConnection` |

## 6. Logs do Traefik

```bash
k3s kubectl logs -n kube-system deployment/traefik --tail=100
```

Avisos sobre `aliasHeadersStrategy` tratam de segurança de cabeçalhos HTTP; isoladamente, não comprovam que sejam a causa de um `404`.

## 7. Resultado deste laboratório

O Pod NGINX respondeu `200 OK` diretamente; o Ingress local com `Host: nginx.lab` também respondeu `200 OK`; o acesso pelo Windows foi finalmente validado com `HTTP/1.1 200 OK`.
