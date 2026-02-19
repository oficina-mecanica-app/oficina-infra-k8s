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

O arquivo `docker-compose.yml é a forma mais rápida de subir todas as dependências necessárias para rodar os microsserviços localmente em sua IDE.

**Como subir:**

No diretório raiz deste repositório, execute:

```bash
docker-compose up -d
```

**Portas e Acessos:**
* **RabbitMQ Panel**: http://localhost:15672 (admin / admin_password)
* **SonarQube**: http://localhost:9000
* **MailHog UI**: http://localhost:8025


## ☸️ Orquestração via Kubernetes (K8s)

O arquivo `services.yaml contém os manifestos de Deployment e Service para orquestrar o ambiente em um cluster Kubernetes (como Minikube, Kind ou Docker Desktop).

**Como aplicar os manifestos:**

```bash
kubectl apply -f services.yaml
```

### Estrutura de Deployments:

O manifesto cria pods isolados para cada infraestrutura, garantindo alta disponibilidade e gerenciamento de recursos:

* `rabbitmq`: Porta 5672 (AMQP) e 15672 (Management).
* `postgres`: Instância configurada para o banco `oficina_os`.
* `mysql`: Instância configurada para o banco `oficina_billing`.
* `mongo`: Instância para o banco de dados documental do Execution.
* `sonarqube: Para manter o histórico de qualidade dentro do cluster.





1. **Abertura e Orçamento**: O usuário cadastra o cliente, o veículo e os itens da OS.
2. **Publicação de Cobrança**: Ao finalizar o orçamento, o serviço publica um evento na fila `os-created-queue` para que o `Billing Service` processe o pagamento.
3. **Atualização por Pagamento**: O serviço consome a fila `os-payment-approved-queue`. Ao receber a confirmação, o status da OS é alterado para `PAID` ou `IN_PROGRESS`.
4. **Finalização de Serviço**: O serviço consome a fila `execution-finished-queue`. Quando o mecânico termina o trabalho no `Execution Service`, a OS é marcada automaticamente como `COMPLETED` aqui.

## 📥 Filas de Entrada (Consumer)

* `os-payment-approved-queue`: Recebe a confirmação de pagamento do Billing Service.
* `execution-finished-queue`: Recebe a notificação de serviço concluído do Execution Service.

## 📤 Fila de Saída (Producer)

* `os-created-queue`: Dispara os dados da OS (ID e Valor Total) para que o sistema de cobrança inicie o checkout.


## 🧪 Qualidade, BDD e Testes
O **OS Service** possui uma estratégia de testes em camadas para garantir a estabilidade do fluxo principal:

### Testes Unitários e SonarQube

Focado na cobertura de Controllers, Services e Repositories.


```bash
# Executar testes e gerar relatório JaCoCo
mvn clean verify

# Enviar para o SonarQube local
mvn sonar:sonar \
  -Dsonar.projectKey=oficina-os-service \
  -Dsonar.host.url=http://localhost:9000 \
  -Dsonar.login=admin \
  -Dsonar.password=admin123
```

## Testes BDD (Behavior Driven Development)
Utilizamos **Cucumber** para validar cenários de negócio em linguagem natural (Gherkin). Isso garante que o software atenda exatamente aos requisitos especificados.
* **Cenário Validado**: Criação de Ordem de Serviço com sucesso, garantindo a integridade dos dados de Cliente e Veículo.
* **Localização**: `src/test/resources/features/service_order.feature

## 🐳 Como rodar a aplicação

1. Certifique-se de que o **RabbitMQ** e o **Banco de Dados** (MySQL/PostgreSQL) estejam rodando (via Docker Compose na raiz do projeto).
2. Execute o comando:

```bash
mvn spring-boot:run
```