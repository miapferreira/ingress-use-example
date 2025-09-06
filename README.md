# Ingress Use Example

Este repositório demonstra como configurar e usar Ingress no Kubernetes com Kind (Kubernetes in Docker), criando um ambiente local para testar roteamento de múltiplas aplicações através de um único ponto de entrada.

## 📋 Pré-requisitos

Antes de começar, certifique-se de ter instalado:

- [Docker](https://docs.docker.com/get-docker/)
- [Kind](https://kind.sigs.k8s.io/docs/user/quick-start/#installation)
- [kubectl](https://kubernetes.io/docs/tasks/tools/)

## 🏗️ Arquitetura

Este lab cria:

- **2 Aplicações Nginx** (`app1-michel` e `app2-michel`)
- **2 Services** para expor as aplicações
- **1 Ingress** para rotear tráfego baseado no hostname
- **NGINX Ingress Controller** para processar as regras de ingress

```
┌─────────────────┐    ┌──────────────────┐    ┌─────────────────┐
│   Navegador     │───▶│  Ingress         │───▶│  App1-michel    │
│                 │    │  Controller      │    │  (nginx)        │
└─────────────────┘    │                  │    └─────────────────┘
                       │                  │
                       │                  │    ┌─────────────────┐
                       └──────────────────┘───▶│  App2-michel    │
                                              │  (nginx)        │
                                              └─────────────────┘
```

## 🚀 Setup Rápido

### 1. Criar o Cluster Kind

```bash
# Criar cluster com mapeamento de portas
kind create cluster --config kind-config.yaml
```

### 2. Instalar NGINX Ingress Controller

```bash
# Instalar ingress controller específico para Kind
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/kind/deploy.yaml

# Aguardar o controller ficar pronto
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```

### 3. Deploy das Aplicações

```bash
# Deploy da App1
kubectl apply -f app1-deployment.yaml
kubectl apply -f app-1-service.yaml

# Deploy da App2
kubectl apply -f app-2-deployment.yaml
kubectl apply -f app-2-service.yaml
```

### 4. Configurar Ingress

```bash
# Aplicar configuração do ingress
kubectl apply -f multi-app-ingress.yaml
```

### 5. Configurar DNS Local

Adicione as seguintes linhas ao seu arquivo `/etc/hosts`:

```bash
sudo nano /etc/hosts
```

Adicione:
```
127.0.0.1 app1-michel
127.0.0.1 app2-michel
```

## 🧪 Testando as Aplicações

### Via Navegador

Abra seu navegador e acesse:

- **App1:** http://app1-michel:8080
- **App2:** http://app2-michel:8080

### Via cURL

```bash
# Testar App1
curl -H "Host: app1-michel" http://localhost:8080

# Testar App2
curl -H "Host: app2-michel" http://localhost:8080
```

## 📁 Estrutura dos Arquivos

```
├── app1-deployment.yaml      # Deployment da primeira aplicação
├── app-1-service.yaml        # Service da primeira aplicação
├── app-2-deployment.yaml     # Deployment da segunda aplicação
├── app-2-service.yaml        # Service da segunda aplicação
├── multi-app-ingress.yaml    # Configuração do Ingress
├── kind-config.yaml          # Configuração do cluster Kind
└── README.md                 # Esta documentação
```

## 🔧 Configurações Detalhadas

### Kind Configuration (`kind-config.yaml`)

```yaml
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: michel-test-cluster
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 80
        hostPort: 8080    # Mapeia porta 80 do container para 8080 do host
        protocol: TCP
      - containerPort: 443
        hostPort: 8443    # Mapeia porta 443 do container para 8443 do host
        protocol: TCP
  - role: worker
  - role: worker
```

### Ingress Configuration (`multi-app-ingress.yaml`)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: multi-app-ingress
spec:
  ingressClassName: nginx    # Especifica o controller a ser usado
  rules:
  - host: app1-michel        # Roteia app1-michel para app1-michel-service
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app1-michel-service
            port:
              number: 80
  - host: app2-michel        # Roteia app2-michel para app2-michel-service
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: app2-michel-service
            port:
              number: 80
```

## 🔍 Verificações e Troubleshooting

### Verificar Status dos Recursos

```bash
# Verificar pods
kubectl get pods

# Verificar services
kubectl get services

# Verificar ingress
kubectl get ingress

# Verificar ingress controller
kubectl get pods -n ingress-nginx
```

### Logs do Ingress Controller

```bash
# Ver logs do ingress controller
kubectl logs -n ingress-nginx -l app.kubernetes.io/component=controller
```

### Teste de Conectividade

```bash
# Testar conectividade interna
kubectl run test-pod --image=busybox --rm -it --restart=Never -- wget -qO- http://app1-michel-service

# Testar via port-forward (alternativa)
kubectl port-forward -n ingress-nginx service/ingress-nginx-controller 8080:80
```

## 🧹 Limpeza

Para remover todos os recursos:

```bash
# Deletar cluster Kind
kind delete cluster --name michel-test-cluster

# Remover entradas do /etc/hosts (opcional)
sudo sed -i '' '/app1-michel/d' /etc/hosts
sudo sed -i '' '/app2-michel/d' /etc/hosts
```

## 🎯 Objetivos de Aprendizado

Este lab demonstra:

- ✅ Como configurar um cluster Kubernetes local com Kind
- ✅ Como instalar e configurar um Ingress Controller
- ✅ Como criar Deployments e Services
- ✅ Como configurar Ingress para roteamento baseado em hostname
- ✅ Como mapear portas no Kind para acesso externo
- ✅ Como configurar DNS local para desenvolvimento

## 📚 Conceitos Aplicados

- **Ingress**: Recurso do Kubernetes para gerenciar acesso externo aos serviços
- **Ingress Controller**: Implementação que processa as regras de ingress
- **Host-based Routing**: Roteamento baseado no cabeçalho Host da requisição
- **Kind**: Ferramenta para executar clusters Kubernetes locais usando Docker
- **Port Mapping**: Mapeamento de portas do container para o host

## ⭐ Se Este Repositório Te Ajudou

Se este lab te ajudou a entender como funciona o **Ingress no Kubernetes** e você conseguiu implementar com sucesso, considere dar uma ⭐ **estrela** para o repositório! 

Isso ajuda:
- 📈 **Visibilidade** do projeto para outras pessoas
- 🎯 **Motivação** para criar mais conteúdo educativo
- 🤝 **Comunidade** a encontrar recursos úteis

## 🤝 Contribuições

Sinta-se à vontade para contribuir com melhorias, correções ou novos exemplos!

- 🐛 **Encontrou um bug?** Abra uma issue
- 💡 **Tem uma ideia?** Faça um fork e abra um PR
- 📚 **Quer melhorar a documentação?** Suas contribuições são bem-vindas!

## 📄 Licença

Este projeto está sob a licença MIT. Veja o arquivo [LICENSE](LICENSE) para mais detalhes.