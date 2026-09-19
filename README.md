# Talherzim — Documentação

Repositório dedicado à documentação do **Talherzim**, um organizador de receitas com sugestão por ingredientes disponíveis.

> O usuário cadastra os ingredientes que tem em casa e o sistema sugere receitas que podem ser preparadas com eles, priorizando as que aproveitam o maior número de itens da despensa — reduzindo desperdício e facilitando a decisão do que cozinhar.

## Visão geral

| Item | Descrição |
|---|---|
| Produto | Organizador de Receitas com Sugestão por Ingredientes Disponíveis |
| Tipo | Aplicação Web Responsiva (MVP) |
| Prazo | 1 mês (4 semanas) |
| Stack | React (front-end) · Node.js/Express (back-end) · PostgreSQL (banco) |
| Organização | [Talhezim-Receitas-Rapidas](https://github.com/Talhezim-Receitas-Rapidas) |

## Documentação

| Documento | Conteúdo |
|---|---|
| [openapi.yaml](openapi.yaml) | Contrato OpenAPI 3.0.3 dos endpoints de autenticação, com schemas, formato de erro e segurança `Bearer`. |
| [docs/contrato-api.md](docs/contrato-api.md) | Versão legível do contrato da API: header de autenticação, formato de erro, endpoints e exemplos com `curl`. |
| [docs/commits.md](docs/commits.md) | Guia de mensagens de commit no padrão [Conventional Commits](https://www.conventionalcommits.org/en/v1.0.0/). |
| [docs/requisitos.md](docs/requisitos.md) | Conversa com o stakeholder, escopo do MVP, requisitos funcionais (RF01–RF16) e não funcionais (RNF01–RNF13), cronograma e critério de aceite. |
| [docs/der.md](docs/der.md) | Diagrama Entidade-Relacionamento, decisões de modelagem, índices e query de validação do ranqueamento. |
| [docs/schema.sql](docs/schema.sql) | Script de criação do schema PostgreSQL (tabelas `usuarios`, `ingredientes`, `receitas`, `receita_ingredientes` e índices). |
| [docs/diagrama-classes.md](docs/diagrama-classes.md) | Diagrama de classes com entidades, serviços e provedores de dados. |
| [docs/modelagem-fluxo-app.md](docs/modelagem-fluxo-app.md) | Fluxos de navegação/uso do app e rastreabilidade fluxo × requisito × caminho de erro. |
| [semana-1.md](semana-1.md) | Plano da semana 1: contrato da API, divisão do time, issues e Definition of Done. |
| [resumo.md](resumo.md) | Resumo consolidado de toda a documentação. |
| [roadmap.md](roadmap.md) | Roadmap de execução do projeto, por fases e semanas. |

> Os diagramas usam sintaxe [Mermaid](https://mermaid.js.org/) e são renderizados nativamente pelo GitHub ao abrir os arquivos `.md`.

## Estrutura do repositório

```
talherzim-documentation/
├── docs/
│   ├── requisitos.md
│   ├── der.md
│   ├── schema.sql
│   ├── diagrama-classes.md
│   ├── modelagem-fluxo-app.md
│   ├── contrato-api.md
│   └── commits.md
├── openapi.yaml
├── resumo.md
├── roadmap.md
├── semana-1.md
├── README.md
└── LICENSE
```

## Versionamento e commits

Adotamos um fluxo de Git simples e o padrão **Conventional Commits** (RNF08).
O guia completo de mensagens está em [docs/commits.md](docs/commits.md).

### Branches

| Branch | Papel |
|---|---|
| `main` | Código estável e entregável. Protegida; só recebe merge de `develop`. |
| `develop` | Integração contínua das features. Protegida. |
| `feature/<assunto>` | Trabalho em uma funcionalidade (ex.: `feature/auth-backend`). Sai de `develop`. |
| `fix/<assunto>` | Correção de bug fora do fluxo de feature. |
| `docs/<assunto>` | Mudanças apenas de documentação. |

- Uma branch por issue, com nome curto e em `kebab-case`.
- Rebase/merge de `develop` na sua branch antes de abrir o PR.

### Commits

Formato: `<tipo>(<escopo>): <descrição no imperativo>`. Exemplos:

```text
feat(auth): adiciona endpoint de login com JWT
docs(api): documenta formato de erro do contrato
fix(despensa): corrige isolamento por usuário
```

Tipos e escopos permitidos: veja [docs/commits.md](docs/commits.md#2-tipos-permitidos).

### Pull Requests

- PR sempre de `feature/*` → `develop` (nunca direto na `main`).
- Título do PR no mesmo padrão do commit.
- Descrição com o que mudou, como testar e a issue relacionada (`Closes #NN`).
- Pelo menos **1 review** aprovado e CI (lint + build) verde antes do merge.
- Sem segredos no código; variáveis de ambiente via `.env` (ver `.env.example`).

### Contrato da API

O contrato fica em [openapi.yaml](openapi.yaml) e na versão legível em
[docs/contrato-api.md](docs/contrato-api.md). Alterações de rota, payload ou
código de erro devem atualizar os dois arquivos no mesmo PR.

## Escopo do MVP

**Incluso:** autenticação de usuários, CRUD da despensa de ingredientes, base de receitas (API Spoonacular ou dataset estático), matching/ranqueamento por ingredientes correspondentes e listagem/detalhe de receitas.

**Fora do escopo:** app mobile nativo, recomendação por IA/ML, filtros avançados, compartilhamento social e avaliações de receitas.

**Opcional (stretch goal):** reconhecimento de ingredientes por foto (RF12–RF16).

## Licença

Distribuído sob a licença MIT. Consulte [LICENSE](LICENSE).
