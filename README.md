# API REST — Armazém do Seu João

Projeto didático de **API REST** construído em **TypeScript + Node + Express + SQLite**, seguindo uma arquitetura em camadas.

API Feita para solucionar um caso de um pequeno armazém do Seu João que previamente controlava tudo usando um caderno: os produtos em estoque, a lista de clientes fiéis (os "fiados") e as vendas do dia. E como o armazém começou a crescer e a aceitar pedidos por WhatsApp. O caderno, que antes dava conta, agora está virando um problema:

O Seu João demora para achar o que um cliente já comprou no mês.
Já aconteceu de vender um produto que estava em falta sem ele perceber.
A filha dele, que ajuda no caixa, não consegue consultar o caderno enquanto ele está atendendo.
No fim do mês, somar manualmente o total que cada cliente deve dá um trabalho enorme.

## Pré-requisitos

Antes de começar, garanta que você tem instalado:

- **Node.js 20 ou superior** — [download](https://nodejs.org)
- **npm** (vem junto com o Node)
- Um editor de código — recomendamos **VS Code**
- **Extensão REST Client** no VS Code (para testar os endpoints com o arquivo `.http`)

Verifique as versões no terminal:

```bash
node --version   # esperado: v20.x ou superior
npm --version
```

---

## 2. Tecnologias e por que usamos cada uma

| Tecnologia     | Papel no projeto      | Por que esta escolha                                                                                                     |
| -------------- | --------------------- | ------------------------------------------------------------------------------------------------------------------------ |
| **TypeScript** | Linguagem             | Adiciona tipos ao JavaScript — o editor avisa de erros antes de rodar e ajuda a entender o fluxo de dados entre camadas. |
| **Node.js**    | Runtime               | Executa o JavaScript fora do navegador. É o motor do nosso servidor.                                                     |
| **Express**    | Framework HTTP        | Simplifica a criação de rotas, o parse de JSON e o uso de middlewares.                                                   |
| **sqlite3**    | Banco de dados        | Banco relacional em arquivo único, sem servidor. Perfeito para aprendizado.                                              |
| **Zod**        | Validação             | Garante que os dados que chegam nos endpoints estão no formato esperado.                                                 |
| **tsx**        | Executor de TS em dev | Roda arquivos TypeScript direto, com reload automático.                                                                  |

---

## 3. Arquitetura em camadas

A aplicação é organizada em **camadas**, onde cada camada **só conversa com a camada imediatamente abaixo**. Isso mantém cada peça com uma única responsabilidade.

```
        ┌──────────────────────┐
        │       Cliente        │  (navegador, app, Postman, arquivo .http)
        └──────────┬───────────┘
                   │ HTTP
        ┌──────────▼───────────┐
        │       Routes         │  Define as URLs e os métodos HTTP
        └──────────┬───────────┘
                   │
        ┌──────────▼───────────┐
        │     Controller       │  Recebe a requisição, valida entrada,
        │                      │  responde com status + JSON
        └──────────┬───────────┘
                   │
        ┌──────────▼───────────┐
        │       Service        │  Regras de negócio (RN01..RN07)
        └──────────┬───────────┘
                   │
        ┌──────────▼───────────┐
        │      Repository      │  Acesso ao banco (SQL puro)
        └──────────┬───────────┘
                   │
        ┌──────────▼───────────┐
        │       SQLite         │  armazenamento físico (arquivo .db)
        └──────────────────────┘
```

### Responsabilidade de cada camada

- **Model**: descreve o **formato dos dados**. No nosso projeto, os Models são **interfaces TypeScript** (`Product`, `Customer`, `Order`, `OrderItem`).
- **Repository**: a **única camada que fala com o banco**. Contém o SQL. Devolve dados como objetos simples.
- **Service**: contém as **regras de negócio**. Decide _o que_ pode e _o que_ não pode acontecer (ex.: "e-mail duplicado não é permitido"). Não conhece HTTP nem SQL.
- **Controller**: adapta o **mundo HTTP** para o **mundo do Service**. Lê `req.body`, `req.params`, chama o Service, devolve `res.status(...).json(...)`.
- **Routes**: liga **URL + método HTTP** a uma função do Controller.

> **Regra de ouro**: Se você está escrevendo SQL em um Controller, ou status HTTP em um Service, a camada errada está fazendo o trabalho. Pare e mova o código.

---

## 4. Estrutura de pastas

```
exemplo-api-2ads/
├── src/
│   ├── config/
│   │   └── database.ts           # conexão com o SQLite
│   ├── models/                   # interfaces TypeScript das entidades
│   │   ├── product.model.ts
│   │   ├── customer.model.ts
│   │   └── order.model.ts
│   ├── repositories/             # acesso ao banco (SQL puro)
│   │   ├── product.repository.ts
│   │   ├── customer.repository.ts
│   │   └── order.repository.ts
│   ├── services/                 # regras de negócio
│   │   ├── product.service.ts
│   │   ├── customer.service.ts
│   │   └── order.service.ts
│   ├── controllers/              # adaptam HTTP ↔ Service
│   │   ├── product.controller.ts
│   │   ├── customer.controller.ts
│   │   └── order.controller.ts
│   ├── routes/                   # definição das URLs
│   │   ├── product.routes.ts
│   │   ├── customer.routes.ts
│   │   ├── order.routes.ts
│   │   └── index.ts              # agrega todas as rotas
│   ├── middlewares/
│   │   └── error-handler.ts      # trata erros de forma centralizada
│   ├── schemas/                  # validações Zod
│   │   ├── product.schema.ts
│   │   ├── customer.schema.ts
│   │   └── order.schema.ts
│   ├── errors/
│   │   └── app-error.ts          # classe de erro customizada
│   ├── database/
│   │   ├── schema.sql            # estrutura do banco
│   │   └── seed.ts               # dados iniciais
│   ├── app.ts                    # configura o Express
│   └── server.ts                 # sobe o servidor
├── requests.http                 # exemplos de chamadas aos endpoints
├── package.json
├── tsconfig.json
├── .gitignore
├── README.md
└── Analise.md
```

---