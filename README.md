# Desafio Loja - Arquitetura de Microsserviços com Spring Boot e RabbitMQ

**Repositório Principal:** [guntz-desafio-loja-meta](https://github.com/ricardoguntzell/guntz-desafio-loja-meta)

Este projeto implementa uma solução de processamento de pedidos de uma loja virtual, utilizando uma arquitetura baseada em microsserviços com comunicação assíncrona. A solução é dividida em dois serviços principais: `order-receive` e `order-processor`.

## Visão Geral da Arquitetura

O fluxo de um pedido é desacoplado para garantir resiliência e escalabilidade:

1.  **Recebimento do Pedido**: O cliente (ou um sistema de frontend) envia um novo pedido para o microsserviço `order-receive` através de uma API REST.
2.  **Enfileiramento**: O `order-receive` realiza uma validação inicial e, se o pedido for válido, o publica em uma fila no RabbitMQ para processamento assíncrono. Isso permite que a API de recebimento responda rapidamente ao cliente.
3.  **Processamento e Persistência**: O microsserviço `order-processor` consome os pedidos da fila, realiza o processamento (cálculos, validações de negócio) e persiste as informações em um banco de dados.

```
[Cliente] -> [POST /orders @ order-receive:8080] -> [RabbitMQ] -> [order-processor:8081] -> [H2 Database]
```

A solução também conta com uma pilha de **observabilidade** para monitoramento de métricas com Prometheus e Grafana.

---

## Microsserviços

Este repositório é um monorepo que contém os seguintes microsserviços:

### 1. `order-receive`

*   **Responsabilidade**: Servir como um gateway para receber novos pedidos. É uma aplicação leve cuja única função é validar a estrutura da requisição e enviá-la para a fila de processamento.
*   **Tecnologias**: Spring Boot, Spring Web, Spring for RabbitMQ.
*   **Porta**: `8080`
*   **Repositório**: guntz-desafio-loja-order-receive

### 2. `order-processor`

*   **Responsabilidade**: Consumir os pedidos da fila, aplicar regras de negócio, calcular totais, salvar o pedido final no banco de dados e expor uma API para consulta dos dados processados.
*   **Tecnologias**: Spring Boot, Spring Data JPA, Spring for RabbitMQ, H2 Database.
*   **Porta**: `8081`
*   **Repositório**: guntz-desafio-loja-order-processor

### 3. `docker-compose` (Infraestrutura)

*   **Responsabilidade**: Orquestrar os contêineres da infraestrutura necessária para a aplicação.
*   **Serviços Inclusos**:
    *   **RabbitMQ**: Broker de mensageria para comunicação assíncrona.
    *   **Prometheus**: Coleta de métricas dos microsserviços.
    *   **Grafana**: Visualização das métricas coletadas pelo Prometheus em dashboards.
*   **Portas**:
    *   RabbitMQ: `5672` (AMQP), `15672` (Management UI)
    *   Prometheus: `9090`
    *   Grafana: `3000`
---

## Pré-requisitos

*   Java 17 ou superior
*   Maven 3.8+
*   Docker e Docker Compose (para rodar o RabbitMQ)
*   Um cliente de API (Postman, Insomnia, etc.)

## Como Executar o Projeto

### 1. Iniciar o RabbitMQ

Na raiz do projeto, execute o Docker Compose para iniciar um container do RabbitMQ:
Na raiz do projeto, execute o Docker Compose para iniciar toda a infraestrutura (RabbitMQ, Prometheus e Grafana):
```bash
docker-compose up -d
```

Isso iniciará o RabbitMQ na porta `5672` e a interface de gerenciamento na porta `15672`.

### 2. Configurar Variáveis de Ambiente

Os microsserviços esperam as credenciais do RabbitMQ como variáveis de ambiente. Exporte-as no seu terminal:

```bash
export RABBITMQ_DEFAULT_USER=rabbitmq
export RABBITMQ_DEFAULT_PASS=rabbitmq
```

### 3. Executar os Microsserviços

Abra dois terminais separados.

**Terminal 1: `order-receive`**
```bash
cd microservices/order-receive
./mvnw spring-boot:run
```

**Terminal 2: `order-processor`**
```bash
cd microservices/order-processor
./mvnw spring-boot:run
```

Ao final, ambos os serviços estarão rodando e conectados ao RabbitMQ.

## Acessando os Serviços e Ferramentas

Com tudo em execução, você pode acessar as seguintes URLs:

- **API do `order-receive` (Swagger UI)**:
  - http://localhost:8080/swagger-ui.html
- **Console do Banco H2 (`order-processor`)**:
  - http://localhost:8081/h2-console
- **Prometheus UI**:
  - http://localhost:9090
- **Grafana UI**:
  - http://localhost:3000 (Login: `admin` / `admin`)
- **RabbitMQ Management UI**:
  - http://localhost:15672 (Login: `rabbitmq` / `rabbitmq` ou o que foi definido nas variáveis de ambiente)

## Autor

**Ricardo Guntzell**
*   **GitHub**: @ricardoguntzell
*   **LinkedIn**: ricardoguntzelljr