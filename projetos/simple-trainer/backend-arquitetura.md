# 📘 Documento de Arquitetura Backend: Workout BFF

**Versão:** 1.0 (Draft)
**Autor:** Elio Neto (via Assistant)
**Status:** Planejamento
**Data:** 04/02/2026

---

## 1. Visão Geral e Responsabilidades

O Backend atuará como um **Backend For Frontend (BFF)**. Ele não possui estado próprio (Stateless) e não gerencia arquivos em disco local. Sua função é atuar como uma camada de segurança, validação e transformação de dados.

### Responsabilidades Chave
1.  **Adapter:** Converter as requisições de negócio (JSON Objects) em primitivas de armazenamento (Key-Value Pairs) para o banco Rust.
2.  **Gatekeeper:** Gerenciar Autenticação (Quem é você?) e Autorização (Você pode ver isso?).
3.  **Sync Orchestrator:** Processar lotes (batches) de dados vindos do modo offline e entregar deltas de atualização.
4.  **Media Proxy:** Gerenciar assinaturas de segurança para upload direto na CDN (Cloudflare R2), sem trafegar binários pelo servidor.

---

## 2. Stack Tecnológica

| Componente | Tecnologia | Justificativa |
| :--- | :--- | :--- |
| **Runtime** | **Node.js (v20 LTS)** | Alta performance em I/O, ideal para ser "proxy" do banco Rust. |
| **Framework** | **NestJS** | Arquitetura modular, injeção de dependência e forte suporte a TypeScript. |
| **Linguagem** | **TypeScript 5+** | Compartilhamento de DTOs/Interfaces com o Frontend. |
| **Comunicação DB** | **Axios / HttpModule** | Cliente HTTP para comunicar com a API REST do banco Rust. |
| **Storage SDK** | **AWS SDK v3 (S3 Client)** | Compatível com Cloudflare R2 para geração de Presigned URLs. |
| **Auth** | **Passport + JWT** | Padrão da indústria para autenticação stateless. |
| **Validação** | **class-validator** | Validação de DTOs automática nos Controllers. |

---

## 3. Arquitetura de Software

Seguiremos a arquitetura em camadas padrão do NestJS, mas com um módulo especial de integração.

```text
src/
├── app.module.ts
├── main.ts                  # Entry point
│
├── common/                  # Decorators, Filters, Guards globais
│   ├── guards/jwt-auth.guard.ts
│   └── filters/http-exception.filter.ts
│
├── modules/
│   ├── auth/                # Login, Register, JWT Strategy
│   │
│   ├── lsm-bridge/          # CAMADA DE ADAPTAÇÃO (Core)
│   │   ├── lsm.service.ts   # Comunica com Rust DB (HTTP)
│   │   └── lsm.utils.ts     # Transformadores (JSON <-> KV)
│   │
│   ├── sync/                # Lógica de Sincronização
│   │   ├── sync.controller.ts
│   │   └── sync.service.ts
│   │
│   ├── upload/              # Integração R2
│   │   └── upload.service.ts
│   │
│   └── workouts/            # (Opcional) Endpoints REST padrão se necessário
│       └── workouts.controller.ts
```

---

## 4. Integração com Banco de Dados (The LSM Bridge)

Esta é a parte mais crítica do Backend. O NestJS não se conecta via TCP/Socket, ele se conecta via HTTP ao seu binário Rust.

### 4.1. Design das Chaves (Key Design)
O Backend é responsável por montar a chave correta antes de enviar ao Rust.

*   **Padrão:** `ENTITY:USER_ID:UUID:TIMESTAMP`
*   **Exemplo:** `workout:u550e8400:w999123:1707072000`

### 4.2. Fluxo de Escrita (Write)
1.  Nest recebe objeto JSON do Angular.
2.  `LsmBridgeService` serializa o objeto para string.
3.  Monta a chave usando o ID do usuário (do token JWT) e o ID do objeto.
4.  Envia para o Rust: `POST http://rust-db:8080/keys` (ou `/batch`).

### 4.3. Fluxo de Leitura (Read/Search)
1.  Nest recebe pedido de busca (ex: "Todos os treinos").
2.  Chama Rust: `GET http://rust-db:8080/keys/search/prefix?q=workout:u550e8400`
3.  **Transformação:**
    *   O Rust retorna: `[{ key: "...", value: "bytes..." }]`.
    *   O Nest decodifica os bytes para JSON.
    *   O Nest agrupa versões (se o Rust retornar histórico) e entrega apenas a última versão de cada ID.

---

## 5. Módulo de Sincronização (SyncModule)

Este endpoint substitui o CRUD tradicional para a maioria das operações.

### 5.1. Endpoint: PUSH (Client -> Server)
*   **Rota:** `POST /api/sync/push`
*   **Payload (Batch):**
    ```json
    [
      { "entity": "WORKOUT", "id": "uuid-1", "payload": { ... }, "timestamp": 123456 },
      { "entity": "EXERCISE", "id": "uuid-2", "payload": { ... }, "timestamp": 123457 }
    ]
    ```
*   **Lógica:**
    1.  Recebe o array.
    2.  Para cada item, converte para o formato KV:
        *   Key: `workout:userId:uuid-1:123456`
        *   Value: `JSON.stringify(payload)`
    3.  Envia um único Request Batch para o Rust DB.

### 5.2. Endpoint: PULL (Server -> Client)
*   **Rota:** `GET /api/sync/pull?lastSync=1700000000`
*   **Lógica:**
    1.  O Backend consulta o Rust DB buscando por prefixo do usuário (`user:ID`).
    2.  Filtra em memória (Node.js) os registros onde `timestamp_chave > lastSync`.
    3.  Retorna apenas os objetos novos ou modificados.

---

## 6. Módulo de Upload (Cloudflare R2)

Para economizar banda e CPU do servidor Node, usamos Presigned URLs.

### Fluxo
1.  **Frontend:** `POST /api/upload/sign { filename: "foto.jpg", contentType: "image/jpeg" }`
2.  **Backend:**
    *   Valida extensão e tamanho (via metadata).
    *   Gera chave única: `uploads/userId/uuid.jpg`.
    *   Usa `PutObjectCommand` (AWS SDK) para gerar URL assinada (válida por 5 min).
3.  **Backend:** Retorna `{ uploadUrl: "https://r2.cloudfl...", publicUrl: "https://cdn.meuapp..." }`.
4.  **Frontend:** Faz upload direto para `uploadUrl`.
5.  **Frontend:** Salva a `publicUrl` dentro do objeto do exercício e sincroniza via JSON.

---

## 7. Modelo de Autenticação

Como o banco Rust não tem tabela de "Users" relacional, salvaremos o usuário como um KV também.

*   **Registro:**
    1.  Recebe email/senha.
    2.  Hash da senha (bcrypt).
    3.  Gera UUID para o usuário.
    4.  Salva KV: `auth:email:user@email.com` -> `{ userId: "uuid", passwordHash: "..." }`.
    5.  Salva KV: `user:profile:uuid` -> `{ name: "Elio", email: "..." }`.

*   **Login:**
    1.  Busca `auth:email:user@email.com`.
    2.  Compara senha com hash.
    3.  Se OK, gera JWT contendo `{ sub: "uuid", email: "..." }`.

---

## 8. Infraestrutura (Docker Compose)

O ambiente de desenvolvimento deve subir tudo junto.

```yaml
version: '3.8'
services:
  # Seu Banco Rust
  lsm-db:
    image: lsm-kv-store:latest # Sua imagem
    ports:
      - "8080:8080"
    volumes:
      - ./data:/data # Persistência do WAL/SST

  # Backend NestJS
  backend:
    build: .
    ports:
      - "3000:3000"
    environment:
      - DB_URL=http://lsm-db:8080
      - JWT_SECRET=segredo_dev
      - R2_ENDPOINT=...
    depends_on:
      - lsm-db
```

---

## 9. Próximos Passos (Implementação)

1.  **Scaffolding:** `nest new backend` e instalação de dependências (`@nestjs/axios`, `class-validator`, `passport`, `@aws-sdk/client-s3`).
2.  **LSM Service:** Criar o serviço que fala com o Rust e testar a conexão com um "Hello World" de banco de dados.
3.  **Auth:** Implementar o fluxo de registro/login salvando no Rust.
4.  **Sync Controller:** Criar o endpoint que aceita o JSON do Front e converte para KV.

Este documento cobre toda a estrutura necessária para que o Backend seja robusto, seguro e cumpra seu papel de conectar o mundo Offline (Angular) com o mundo de Alta Performance (Rust).
