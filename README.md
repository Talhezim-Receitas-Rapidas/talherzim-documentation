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
| [docs/requisitos.md](docs/requisitos.md) | Conversa com o stakeholder, escopo do MVP, requisitos funcionais (RF01–RF16) e não funcionais (RNF01–RNF13), cronograma e critério de aceite. |
| [docs/der.md](docs/der.md) | Diagrama Entidade-Relacionamento, decisões de modelagem, índices e query de validação do ranqueamento. |
| [docs/schema.sql](docs/schema.sql) | Script de criação do schema PostgreSQL (tabelas `usuarios`, `ingredientes`, `receitas`, `receita_ingredientes` e índices). |
| [docs/diagrama-classes.md](docs/diagrama-classes.md) | Diagrama de classes com entidades, serviços e provedores de dados. |
| [docs/modelagem-fluxo-app.md](docs/modelagem-fluxo-app.md) | Fluxos de navegação/uso do app e rastreabilidade fluxo × requisito × caminho de erro. |
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
│   └── modelagem-fluxo-app.md
├── resumo.md
├── roadmap.md
├── README.md
└── LICENSE
```

## Escopo do MVP

**Incluso:** autenticação de usuários, CRUD da despensa de ingredientes, base de receitas (API Spoonacular ou dataset estático), matching/ranqueamento por ingredientes correspondentes e listagem/detalhe de receitas.

**Fora do escopo:** app mobile nativo, recomendação por IA/ML, filtros avançados, compartilhamento social e avaliações de receitas.

**Opcional (stretch goal):** reconhecimento de ingredientes por foto (RF12–RF16).

## Licença

Distribuído sob a licença MIT. Consulte [LICENSE](LICENSE).
