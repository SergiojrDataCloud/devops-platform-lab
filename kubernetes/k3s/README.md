# K3s — laboratório Kubernetes com NGINX e Traefik

## Objetivo
Documentar um cluster K3s de nó único com NGINX publicado por Ingress via Traefik.

## Ambiente validado
- VM: `VMk8s`
- Sistema: Debian GNU/Linux 12 (Bookworm)
- Kubernetes: K3s; comandos via `k3s kubectl`
- Runtime: containerd
- Ingress Controller: Traefik, instalado pelo K3s
- Aplicação: NGINX
- Host de Ingress: `nginx.lab`
- Resultado final pelo Windows: `HTTP/1.1 200 OK`

Não publique IPs privados reais, kubeconfig, tokens ou segredos no repositório.

## Arquitetura

```text
Windows / cliente HTTP
        | HTTP :80, Host: nginx.lab
        v
IP da VM K3s :80
        v
Traefik (Ingress Controller)
        v
Service nginx :80
        v
Pod NGINX :80
```

## Inspeção do cluster

Execute na VM como root ou usuário autorizado:

```bash
systemctl status k3s --no-pager
k3s kubectl get nodes -o wide
k3s kubectl get pods -A
k3s kubectl get deployment nginx
k3s kubectl get svc nginx
k3s kubectl get endpoints nginx
k3s kubectl get ingress nginx-ingress -o wide
k3s kubectl get ingressclass
k3s kubectl get svc -n kube-system traefik
```

O aviso de depreciação do recurso `Endpoints` não significa, por si só, falha. Alternativa moderna:

```bash
k3s kubectl get endpointslice -l kubernetes.io/service-name=nginx
```

## Ingress

O manifesto `manifests/nginx-ingress.yaml` define a classe `traefik`, host `nginx.lab`, caminho `/` e backend `nginx:80`.

Aplicar a partir da raiz do repositório:

```bash
k3s kubectl apply -f kubernetes/k3s/manifests/nginx-ingress.yaml
```

Inspecionar:

```bash
k3s kubectl describe ingress nginx-ingress
```

## Testes de validação

### 1. Testar o Pod diretamente na VM

Obtenha o IP atual com `k3s kubectl get pods -o wide`:

```bash
curl -i http://IP_DO_POD:80/
```

Esperado: `HTTP/1.1 200 OK` e `Server: nginx/...`.

### 2. Testar o Ingress dentro da VM

```bash
curl -i -H 'Host: nginx.lab' http://127.0.0.1/
```

Esperado: `HTTP/1.1 200 OK` e HTML do NGINX.

### 3. Testar do Windows PowerShell

Substitua `IP_REAL_DA_VM` pelo IPv4 da interface da VM alcançável pelo Windows:

```powershell
Test-NetConnection IP_REAL_DA_VM -Port 80
curl.exe -v -i --resolve nginx.lab:80:IP_REAL_DA_VM http://nginx.lab/
```

`--resolve` direciona `nginx.lab` ao IP indicado e envia o Host correto sem depender de DNS.

Resultado final observado: `HTTP/1.1 200 OK`.

## Observações
- O NGINX foi criado como Deployment e exposto por Service `NodePort` durante o exercício.
- O acesso por Ingress na porta 80 é diferente do acesso direto ao NodePort: o Traefik seleciona a rota pelo cabeçalho Host.
- O endereço `EXTERNAL-IP` do Service Traefik pode ser diferente do IPv4 da interface da VM.
- `nginx.lab` é um nome de laboratório; `curl.exe --resolve` evita configurar DNS ou o arquivo `hosts`.
- Este exercício valida HTTP; TLS/HTTPS ainda não foi configurado.

## Limpeza opcional
Só execute se quiser remover os recursos deste exercício:

```bash
k3s kubectl delete ingress nginx-ingress
k3s kubectl delete service nginx
k3s kubectl delete deployment nginx
```

Não é necessário remover o Traefik ou o K3s para limpar apenas a aplicação de teste.
