# Atividade Avaliativa - Deploy de Aplicação Fullstack em Kubernetes

## 1. Objetivo
Este projeto realiza o deploy de uma aplicação fullstack (Frontend React, Backend Flask e Banco PostgreSQL) em um cluster Kubernetes (Kubectl)[cite: 30, 32].

## 2. Arquitetura

A aplicação é dividida em dois Namespaces para isolamento: `app-namespace` (Aplicação) e `db-namespace` (Banco de Dados)[cite: 65].

* **Frontend (React/Nginx):** Deployment com 2 réplicas[cite: 63, 104, 119]. Acesso externo via Ingress na rota `/`[cite: 74, 104].
* **Backend (Flask API):** Deployment com 2 réplicas[cite: 63, 104, 119]. Acesso externo via Ingress na rota `/api/`[cite: 76, 104].
* **Banco de Dados (PostgreSQL):** StatefulSet com Volume Persistente (PVC)[cite: 64, 79, 104, 118]. Configurações via Secrets[cite: 49, 95].
* **Configuração:** Variáveis de ambiente não sensíveis via ConfigMap[cite: 47, 82].

## 3. Passos para Aplicação (kubectl apply)

Certifique-se de que um IngressController NGINX (como o fornecido pelo Kind ou instalado separadamente) esteja rodando no cluster.

1.  **Configurações e Banco de Dados (Database First):**
    ```bash
    # 1. Aplica Namespaces
    kubectl apply -f namespace.yaml
    # 2. Aplica Configurações
    kubectl apply -f configmap.yaml
    kubectl apply -f secret.yaml
    # 3. Aplica o Banco (PVC e StatefulSet)
    kubectl apply -f database/pvc.yaml
    kubectl apply -f database/service.yaml
    kubectl apply -f database/statefulset.yaml
    ```

2.  **Deploy da Aplicação (Frontend/Backend):**
    ```bash
    # 1. Aplica Backend
    kubectl apply -f backend/service.yaml
    kubectl apply -f backend/deployment.yaml
    # 2. Aplica Frontend
    kubectl apply -f frontend/service.yaml
    kubectl apply -f frontend/deployment.yaml
    ```

3.  **Acesso Externo (Ingress):**
    ```bash
    # 1. Aplica as regras de Ingress
    kubectl apply -f ingress/ingress.yaml
    ```

## 4. Endereço de Acesso Esperado

Após aplicar todos os arquivos e o Ingress estar pronto, o acesso esperado será (assumindo que o IP do seu IngressController seja `localhost` ou `127.0.0.1`):

* **Aplicação Frontend:** `http://<IP-do-IngressController>/`
* **API Backend:** `http://<IP-do-IngressController>/api/`
