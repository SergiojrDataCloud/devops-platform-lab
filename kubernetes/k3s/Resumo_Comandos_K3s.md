# K3s & Kubernetes — Resumo de Comandos

**Objetivo:** consulta rápida dos comandos usados no laboratório K3s com NGINX, Service e Ingress Traefik.

> Execute os comandos no nó K3s, salvo quando indicado. Ajuste nomes, namespaces, portas e endereços conforme o ambiente.

## 1. Estado do cluster

```bash
# Verificar o serviço K3s
sudo systemctl status k3s

# Consultar os nós
sudo kubectl get nodes -o wide

# Verificar informações do cluster
sudo kubectl cluster-info

# Listar namespaces
sudo kubectl get namespaces
```

## 2. Deployments e Pods

```bash
# Listar Deployments
sudo kubectl get deployments

# Listar Pods em todos os namespaces
sudo kubectl get pods -A -o wide

# Listar Pods no namespace default
sudo kubectl get pods -o wide

# Descrever um Deployment
sudo kubectl describe deployment nginx

# Descrever um Pod específico
sudo kubectl describe pod <nome-do-pod>

# Consultar logs de um Pod
sudo kubectl logs <nome-do-pod>

# Acompanhar logs em tempo real
sudo kubectl logs -f <nome-do-pod>
```

**Quando usar:** `get` para visão geral; `describe` para eventos e configuração; `logs` para investigar o comportamento da aplicação.

## 3. Services e endpoints

```bash
# Listar Services
sudo kubectl get services

# Listar Services e portas
sudo kubectl get svc -o wide

# Detalhar o Service NGINX
sudo kubectl describe service nginx

# Consultar EndpointSlices
sudo kubectl get endpointslices

# Consultar endpoints legados, quando disponíveis
sudo kubectl get endpoints nginx
```

**O que conferir:** portas do Service, `targetPort`, seletores e endereços dos Pods associados.

## 4. Ingress e Traefik

```bash
# Listar recursos Ingress
sudo kubectl get ingress

# Listar Ingress com mais detalhes
sudo kubectl get ingress -o wide

# Descrever o Ingress
sudo kubectl describe ingress nginx-ingress

# Verificar a classe de Ingress
sudo kubectl get ingressclass

# Verificar Pods do Traefik
sudo kubectl get pods -A | grep -i traefik

# Consultar logs do Traefik
sudo kubectl logs -n kube-system deployment/traefik
```

No laboratório, o Ingress `nginx-ingress` usa a classe `traefik` e encaminha o host `nginx.lab` para o Service `nginx` na porta `80`.

## 5. Testes HTTP

### Testar o NGINX pelo Service NodePort

```bash
curl -i http://127.0.0.1:<NODEPORT>
```

Substitua `<NODEPORT>` pela porta atribuída ao NodePort no ambiente. No teste realizado no laboratório, a porta foi `32671`.

### Testar o Ingress localmente no nó K3s

```bash
curl -i -H 'Host: nginx.lab' http://127.0.0.1/
```

O cabeçalho `Host` é importante porque o Ingress usa o nome `nginx.lab` para selecionar a regra de encaminhamento.

### Testar a partir de outro computador

```powershell
curl.exe -i -H "Host: nginx.lab" http://<IP-DO-NO-K3S>/
```

Use o IP alcançável do nó K3s. Se estiver usando o nome `nginx.lab` diretamente, configure a resolução DNS ou uma entrada apropriada no arquivo `hosts` do cliente.

## 6. Troubleshooting: ordem de diagnóstico

Siga a sequência da camada mais básica para a entrada HTTP:

```bash
# 1. Nó
sudo kubectl get nodes -o wide

# 2. Pods e estado
sudo kubectl get pods -A -o wide

# 3. Deployment
sudo kubectl describe deployment nginx

# 4. Service e endpoints
sudo kubectl describe service nginx
sudo kubectl get endpoints nginx

# 5. Ingress
sudo kubectl describe ingress nginx-ingress

# 6. Logs da aplicação
sudo kubectl logs deployment/nginx

# 7. Logs do Traefik
sudo kubectl logs -n kube-system deployment/traefik
```

### Interpretação rápida

- **Pod não está `Running` ou `Ready`:** examine `describe pod`, eventos e logs.
- **Service sem endpoints:** confira se os seletores do Service correspondem aos labels dos Pods e se os Pods estão prontos.
- **Resposta 404 pelo Ingress:** confira o host enviado pelo cliente, a regra do Ingress, a classe `traefik` e os logs do controlador.
- **Resposta 502/503:** verifique o backend, o Service, a porta de destino e a prontidão dos Pods.
- **Timeout ou conexão recusada:** confira IP, porta, firewall, rota de rede e se o controlador está escutando nessa porta.
- **Funciona no nó, mas não em outro computador:** investigue conectividade de rede, firewall e resolução do nome `nginx.lab`.

## 7. Códigos HTTP comuns

| Código | Significado | Próxima verificação |
|---|---|---|
| `200 OK` | Requisição atendida | Aplicação acessível |
| `301` / `302` | Redirecionamento | Cabeçalhos e destino do redirecionamento |
| `404 Not Found` | Recurso ou regra não encontrado | Host, caminho e regras do Ingress |
| `502 Bad Gateway` | Gateway recebeu resposta inválida do backend | Service, porta e backend |
| `503 Service Unavailable` | Serviço indisponível | Endpoints e prontidão dos Pods |
| `504 Gateway Timeout` | Backend demorou a responder | Logs, conectividade e saúde do backend |

## 8. Comandos úteis de manutenção

```bash
# Aplicar um manifesto
sudo kubectl apply -f <arquivo.yaml>

# Validar os recursos
sudo kubectl get all

# Remover recursos definidos em um manifesto
sudo kubectl delete -f <arquivo.yaml>

# Ver os eventos recentes ordenados por horário
sudo kubectl get events --sort-by=.metadata.creationTimestamp
```

> **Atenção:** antes de executar `delete`, confirme o arquivo e os recursos afetados. Em ambientes compartilhados ou de produção, valide o impacto antes de alterar ou remover recursos.

## 9. Referência do laboratório

- **Distribuição:** Debian 12 minimal
- **Kubernetes:** K3s
- **Ingress Controller:** Traefik
- **Aplicação de teste:** NGINX
- **Ingress:** `nginx-ingress`
- **Host de teste:** `nginx.lab`
- **Service backend:** `nginx:80`

### Documentação complementar no repositório `devops-platform-lab`

- `kubernetes/k3s/README.md`
- `kubernetes/k3s/manifests/nginx-ingress.yaml`
- `kubernetes/k3s/troubleshooting.md`
