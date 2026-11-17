# 🏗️ Oficina Infra & Orchestration

## Autor
- **Nome:** Lucas Eduardo Peters Scuissiatto
- **RM:** RM365455
- **Curso:** Pós-Graduação em Arquitetura de Software

---

## 📖 Descrição

Este repositório contém as definições de infraestrutura como código (IaC) para o ecossistema de microsserviços da oficina. Aqui estão centralizados os bancos de dados, o broker de mensageria e as ferramentas de suporte.

### 📋 Componentes da Infraestrutura

| Serviço              | Tecnologia                  | Finalidade                                                               |
|:---------------------|:----------------------------|:-------------------------------------------------------------------------|
| **Bancos de Dados**  | PostgreSQL, MySQL, MongoDB  | Persistência de dados dos microsserviços OS, Billing e Execution.        |
| **Mensageria**       | RabbitMQ                    | Broker responsável pela comunicação assíncrona entre serviços.           |
| **Qualidade**        | SonarQube                   | Plataforma para análise estática e métricas de cobertura de código.      |
| **Email Fake**       | MailHog                     | SMTP local para testes de envio de notificações sem disparo real.        |


## 🐳 Execução via Docker Compose (Desenvolvimento)

O arquivo `docker-compose.yml` é a forma mais rápida de subir todas as dependências necessárias para rodar os microsserviços localmente em sua IDE.

**Como subir:**

No diretório raiz deste repositório, execute:

```bash
docker-compose up -d
```

**Portas e Acessos:**
* **RabbitMQ Panel**: http://localhost:15672 (credenciais de desenvolvimento definidas no `docker-compose.yml`)
* **SonarQube**: http://localhost:9000
* **MailHog UI**: http://localhost:8025


## ☸️ Orquestração via Kubernetes (K8s)

O arquivo `services.yaml` contém os manifestos de Deployment e Service para orquestrar o ambiente em um cluster Kubernetes (como Minikube, Kind ou Docker Desktop).

**Como aplicar os manifestos:**

As senhas e o token do Mercado Pago ficam no Secret `oficina-secrets`, que **não é versionado**. Crie o seu a partir do modelo:

```bash
cp secrets.example.yaml secrets.yaml   # secrets.yaml está no .gitignore
# edite os valores em secrets.yaml
kubectl apply -f secrets.yaml
kubectl apply -f services.yaml
```

Os microsserviços leem o mesmo Secret nos seus próprios manifestos (`k8s/deployment.yaml` de cada repositório).

### Estrutura de Deployments:

O manifesto cria pods isolados para cada infraestrutura, garantindo alta disponibilidade e gerenciamento de recursos:

* `rabbitmq`: Porta 5672 (AMQP) e 15672 (Management).
* `postgres`: Instância configurada para o banco `oficina_os`.
* `mysql`: Instância configurada para o banco `oficina_billing`.
* `mongo`: Instância para o banco de dados documental do Execution.
* `sonarqube`: Para manter o histórico de qualidade dentro do cluster.

> As credenciais do `docker-compose.yml` são apenas para desenvolvimento local. Não reutilize esses valores em nenhum ambiente exposto.
