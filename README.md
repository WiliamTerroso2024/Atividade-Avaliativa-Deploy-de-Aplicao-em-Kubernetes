Com certeza\! Vou inserir o diagrama na seção de Arquitetura do seu `README.md`.

Aqui está a versão final do seu **`README.md`** completo, com o diagrama incluído na seção **Arquitetura da Solução**.

-----

## 🚀 Deploy de Aplicação Fullstack em Kubernetes

### 👥 Integrantes da Equipe

Wiliam Terroso de Sousa Melo

-----

### 🎯 Objetivo do Projeto

Este projeto visa realizar o deploy completo de uma aplicação fullstack (**React + Flask + PostgreSQL**) em um cluster Kubernetes, implementando os principais componentes e as melhores práticas de DevOps exigidas na avaliação, incluindo:

  * **Alta Disponibilidade** com mín. 2 réplicas (Frontend/Backend).
  * Gerenciamento de configurações via **ConfigMap**.
  * Gerenciamento de credenciais via **Secrets**.
  * **Persistência de dados** com PersistentVolumeClaim (PVC).
  * Roteamento externo via **Ingress NGINX** (`/` para Frontend e `/api` para Backend).
  * **Separação de Namespaces** para aplicação e banco de dados.

-----

### 🏗️ Arquitetura da Solução

A solução utiliza **dois Namespaces** para garantir o isolamento lógico dos recursos e comunicação interna via **ClusterIP Services**.

```
┌─────────────────────────────────────────────────────────┐
│                     Ingress NGINX                       │
│  localhost/ → Frontend | localhost/api → Backend        │
└──────────────┬──────────────────────┬───────────────────┘
               │                      │
               │                      │
    ┌──────────▼──────────┐  ┌────────▼─────────┐
    │   Frontend Service  │  │  Backend Service  │
    │    (ClusterIP)      │  │   (ClusterIP)     │
    └──────────┬──────────┘  └────────┬──────────┘
               │                      │
    ┌──────────▼──────────┐  ┌────────▼─────────┐
    │ Frontend Deployment │  │ Backend Deployment│
    │    (2 réplicas)     │  │   (2 réplicas)    │
    └─────────────────────┘  └────────┬──────────┘
                                      │
                         ┌────────────▼──────────────┐
                         │  PostgreSQL StatefulSet   │
                         │  (1 réplica + PVC 5Gi)    │
                         └───────────────────────────┘
```

| Recurso | Recurso Kubernetes | Namespace | Detalhe |
| :--- | :--- | :--- | :--- |
| **Frontend (React/Nginx)** | Deployment (**2+ Réplicas**) | `app-namespace` (aplicação) | Consome a API através de variável de ambiente (`VITE_API_URL` do ConfigMap). |
| **Backend (Flask API)** | Deployment (**2+ Réplicas**) | `app-namespace` (aplicação) | Expõe endpoints REST para mensagens. |
| **Banco de Dados (PostgreSQL)** | StatefulSet (1 Réplica) | `db-namespace` (banco de dados) | Usa PVC para persistência. |
| **Configuração** | ConfigMap | Ambos | Variáveis não sensíveis (`DB_HOST`, `DB_PORT`, etc.). |
| **Segurança** | Secret | Ambos | Credenciais do banco (`DB_USER`, `POSTGRES_PASSWORD`, etc.). |
| **Acesso Externo** | Ingress | `app-namespace` | Roteamento HTTP usando NGINX Ingress Controller. |

-----

### 📦 Preparação das Imagens Docker

Verifique se os seus arquivos de deploy (`deployment.yaml`) apontam corretamente para as imagens no seu repositório do Docker Hub:

  * **Backend Image:** `wiliamterroso2025/flask-backend:latest`
  * **Frontend Image:** `wiliamterroso2025/react-frontend:latest`

**Importante:** Os Secrets gerados devem conter as credenciais codificadas em **Base64**.

-----

### 🚀 Passo a Passo para Deploy Manual

Execute estes passos no seu terminal **após iniciar seu cluster Kind** e instalar o **NGINX Ingress Controller**.

#### 1\. Configurações e Banco de Dados (Database First)

Aplique as configurações de isolamento e o deploy do banco de dados.

```bash
# 1. Aplica Namespaces
kubectl apply -f namespace.yaml

# 2. Aplica Configurações e Credenciais
kubectl apply -f configmap.yaml
kubectl apply -f secret.yaml

# 3. Deploy do Banco (PVC, Service e StatefulSet)
kubectl apply -f database/pvc.yaml # Para persistência de dados
kubectl apply -f database/service.yaml # Service ClusterIP para acesso interno
kubectl apply -f database/statefulset.yaml # Deploy do PostgreSQL

# Aguardar PostgreSQL ficar pronto
kubectl wait --namespace db-namespace \
  --for=condition=ready pod \
  --selector=app=postgres \
  --timeout=120s
```

#### 2\. Deploy da Aplicação (Frontend/Backend)

Aplique os Deployments e Services da aplicação, garantindo **alta disponibilidade** (2+ réplicas).

```bash
# 1. Aplica Backend
kubectl apply -f backend/service.yaml
kubectl apply -f backend/deployment.yaml

# 2. Aplica Frontend
kubectl apply -f frontend/service.yaml
kubectl apply -f frontend/deployment.yaml
```

#### 3\. Configurar o Acesso Externo (Ingress)

```bash
# Aplica as regras de Ingress (roteamento / e /api)
kubectl apply -f ingress/ingress.yaml
```

-----

### 🔍 Testando a Aplicação

#### Endereço de Acesso Esperado

O acesso deve ser feito através do IP do seu Ingress Controller (geralmente `localhost` na máquina local):

  * **Aplicação Frontend:** `http://localhost/`
  * **Backend API:** `http://localhost/api/messages`

#### Comandos Úteis para Verificação

  * **Verificar Pods:** `kubectl get pods -A`
  * **Verificar Deployments (Réplicas):** `kubectl get deployments -n app-namespace`
  * **Verificar Ingress:** `kubectl get ingress -n app-namespace`
  * **Verificar PVC:** `kubectl get pvc -n db-namespace`

-----

### 📧 Contato

  * **GitHub:** `@WiliamTerroso2024`
  * **Repositório:** `https://github.com/WiliamTerroso2024/Atividade-Avaliativa-Deploy-de-Aplicao-em-Kubernetes`
