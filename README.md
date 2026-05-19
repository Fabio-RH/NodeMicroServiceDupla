# Node Microservices

Arquitetura de microserviços com Node.js, TypeScript e Fastify. Três serviços independentes orquestrados por um API Gateway.

## Arquitetura

```
Cliente (curl / frontend)
        │
        ▼ :3000
   [ API Gateway ]
   ├── /products/* ──► :3001 [ Product Service ]
   └── /orders/*   ──► :3002 [ Order Service ]
                                     │
                                     └──► :3001 [ Product Service ]
```

| Serviço | Porta | Responsabilidade |
|---------|-------|-----------------|
| api-gateway | 3000 | Ponto de entrada único, roteia requisições |
| product-service | 3001 | Catálogo de produtos |
| order-service | 3002 | Pedidos (consulta o product-service internamente) |

## Stack

- **Node.js 20+** com TypeScript
- **Fastify** — framework HTTP
- **npm Workspaces** — monorepo
- **Docker + docker-compose** — containerização

---

## Como rodar

### Opção 1 — Docker (recomendado)

```bash
docker-compose up --build
```

Para encerrar:

```bash
docker-compose down
```

### Opção 2 — Local (3 terminais)

```bash
# Instala dependências (uma vez só)
npm install

# Terminal 1 — Product Service
npm run product

# Terminal 2 — Order Service
npm run order

# Terminal 3 — API Gateway
npm run gateway
```

---

## Testando a API

Todas as requisições passam pelo **API Gateway na porta 3000**.

### Health check

```bash
curl http://localhost:3000/health
```

```json
{
  "status": "ok",
  "timestamp": "2024-01-01T00:00:00.000Z"
}
```

---

### Produtos

#### Listar todos os produtos

```bash
curl http://localhost:3000/products
```

```json
[
  { "id": 1, "name": "Notebook Pro", "price": 3500, "stock": 10 },
  { "id": 2, "name": "Mouse Gamer", "price": 250, "stock": 50 },
  { "id": 3, "name": "Teclado Mecânico", "price": 450, "stock": 30 }
]
```

#### Buscar produto por ID

```bash
curl http://localhost:3000/products/1
```

```json
{ "id": 1, "name": "Notebook Pro", "price": 3500, "stock": 10 }
```

#### Produto inexistente (404)

```bash
curl http://localhost:3000/products/999
```

```json
{ "error": "Produto não encontrado" }
```

---

### Pedidos

#### Criar pedido

```bash
curl -X POST http://localhost:3000/orders \
  -H "Content-Type: application/json" \
  -d '{"productId": 1, "quantity": 3}'
```

```json
{
  "id": 1,
  "productId": 1,
  "productName": "Notebook Pro",
  "quantity": 3,
  "total": 10500,
  "createdAt": "2024-01-01T00:00:00.000Z"
}
```

#### Listar todos os pedidos

```bash
curl http://localhost:3000/orders
```

#### Pedido com produto inexistente (404)

```bash
curl -X POST http://localhost:3000/orders \
  -H "Content-Type: application/json" \
  -d '{"productId": 999, "quantity": 1}'
```

```json
{ "error": "Produto não encontrado no Product Service" }
```

---

## Estrutura do projeto

```
/
├── docker-compose.yml
├── package.json          ← raiz do monorepo (workspaces)
├── tsconfig.json         ← configuração TypeScript base
└── apps/
    ├── product-service/
    │   ├── src/server.ts
    │   ├── Dockerfile
    │   └── package.json
    ├── order-service/
    │   ├── src/server.ts
    │   ├── Dockerfile
    │   └── package.json
    └── api-gateway/
        ├── src/server.ts
        ├── Dockerfile
        └── package.json
```

## Branches

Cada branch representa um passo incremental do desenvolvimento:

| Branch | O que foi adicionado |
|--------|---------------------|
| `step/01-setup` | Monorepo com npm workspaces e TypeScript |
| `step/02-product-service` | Product Service com Fastify |
| `step/03-order-service` | Order Service independente |
| `step/04-http-communication` | Order Service consultando Product Service via HTTP |
| `step/05-api-gateway` | API Gateway como ponto de entrada único |
| `step/06-docker` / `main` | Containerização com Docker e docker-compose |
