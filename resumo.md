# Resumo da Documentação — Talherzim

Consolidação dos documentos de [requisitos](docs/requisitos.md), [modelagem de dados](docs/der.md), [schema](docs/schema.sql), [diagrama de classes](docs/diagrama-classes.md) e [fluxos do app](docs/modelagem-fluxo-app.md).

## 1. O projeto

**Talherzim** é uma aplicação web responsiva (MVP, 1 mês) em que o usuário cadastra os ingredientes que tem em casa e recebe sugestões de receitas preparáveis com eles, ordenadas pelas que aproveitam mais itens da despensa. O objetivo é reduzir o desperdício de alimentos e facilitar a decisão do que cozinhar.

- **Público-alvo:** qualquer pessoa que cozinha em casa.
- **Stack:** React (front-end), Node.js/Express (back-end), PostgreSQL (banco).
- **Prioridades do stakeholder:** (1) cadastro de ingredientes + sugestão ponta a ponta; (2) cadastro/login de usuário; (3) refinamentos visuais, fotos e filtros extras.

## 2. Escopo

**Incluso no MVP:**

- Cadastro e autenticação de usuários.
- Cadastro, edição e remoção de ingredientes na despensa.
- Base de receitas via API Spoonacular ou dataset estático (JSON).
- Algoritmo de matching/ranqueamento por ingredientes correspondentes.
- Listagem de receitas sugeridas e tela de detalhe.

**Fora do escopo:** app mobile nativo, recomendação por IA/ML, filtros avançados, compartilhamento social e avaliações.

**Opcional (stretch goal):** reconhecimento de ingredientes por foto (RF12–RF16), condicionado ao prazo.

## 3. Requisitos

### Funcionais (RF)

| Grupo | IDs | Resumo |
|---|---|---|
| Autenticação | RF01, RF02 | Criar conta com e-mail/senha; login e logout. |
| Despensa | RF03–RF05, RF11 | Cadastrar, editar, remover e listar ingredientes; lista vinculada exclusivamente ao usuário autenticado. |
| Sugestão | RF06, RF07 | Buscar receitas compatíveis e ranquear pela quantidade de ingredientes correspondentes. |
| Receitas | RF08–RF10 | Exibir nome, ingredientes, modo de preparo e imagem; imagem padrão quando ausente; detalhe completo. |
| Foto (Adicional) | RF12–RF16 | Upload/captura de foto, reconhecimento de ingredientes, confirmação/edição antes de salvar e tratamento de falha de reconhecimento. |

### Não funcionais (RNF)

| Categoria | IDs | Resumo |
|---|---|---|
| Desempenho | RNF01, RNF10, RNF12 | Busca de receitas ≤ 3 s; reconhecimento de imagem ≤ 8 s; upload limitado (ex.: 5 MB). |
| Segurança | RNF02, RNF03 | Senhas com hash (bcrypt); isolamento de dados entre usuários. |
| Tecnologia | RNF05, RNF06 | React + Node.js/Express; PostgreSQL ou MongoDB. |
| Confiabilidade | RNF07, RNF13 | Contingência com dataset estático; falha de visão computacional não derruba o app. |
| Usabilidade / Escalabilidade / Manutenibilidade | RNF04, RNF08, RNF09 | Interface responsiva; versionamento Git organizado; suportar pequeno grupo de testes. |
| Privacidade | RNF11 | Não reter imagem após processamento, salvo consentimento. |

## 4. Modelo de dados

Banco escolhido: **PostgreSQL** — o domínio é relacional (usuário 1:N ingredientes, receita 1:N ingredientes) e o ranqueamento é um JOIN + agregação.

| Tabela | Papel |
|---|---|
| `usuarios` | Conta do usuário (`email` único, `senha_hash`). |
| `ingredientes` | Despensa; cada item pertence a um usuário (`usuario_id`), com nome, quantidade e unidade. |
| `receitas` | Receitas persistidas como contingência da API; `fonte` (`spoonacular`/`estatico`) e `fonte_id` evitam duplicidade. |
| `receita_ingredientes` | Ingredientes de cada receita, guardados como texto (a receita não pertence a um usuário). |

O matching (RF07) compara `lower(ingredientes.nome)` com `lower(receita_ingredientes.nome_ingrediente)` por usuário, agrupando por receita e ordenando pela contagem de correspondências. Índices apoiam a listagem da despensa e as comparações case-insensitive.

## 5. Arquitetura (visão de classes)

- **Entidades:** `Usuario`, `Ingrediente`, `Receita`, `ReceitaIngrediente`.
- **Serviços:** `AuthService` (registro/login/hash), `DespensaService` (CRUD de ingredientes), `RecipeSuggestionService` (busca e ranqueamento).
- **Provedores de receita:** interface `RecipeProvider`, implementada por `SpoonacularProvider` e `StaticDatasetProvider` (contingência).
- **Futuro:** `ImageRecognitionService` + `ImageRecognitionProvider` (funcionalidade opcional).

O padrão *Provider* isola a fonte de receitas e permite trocar a API pelo dataset estático sem alterar a lógica de sugestão.

## 6. Fluxos do app

| Fluxo | Requisitos | Caminho de erro coberto |
|---|---|---|
| 1. Cadastro e login | RF01, RF02, RNF02 | Dados inválidos; senha incorreta. |
| 2. Gestão da despensa | RF03, RF04, RF05 | Nome do ingrediente vazio. |
| 3. Sugestão de receitas | RF06, RF07, RNF07 | API indisponível → dataset estático. |
| 4. Detalhe da receita | RF08, RF09, RF10 | Receita sem imagem → imagem padrão. |
| 5. Reconhecimento por foto (opcional) | RF12–RF16, RNF13 | Nenhum ingrediente reconhecido → cadastro manual. |

Os fluxos 1–4 são bloqueantes para o aceite do MVP; o fluxo 5 é incremento opcional.

## 7. Prazo e critério de aceite

Cronograma de 4 semanas: **S1** requisitos, modelagem e setup + autenticação; **S2** CRUD de ingredientes e integração de receitas; **S3** matching/ranqueamento e telas de receitas; **S4** ajustes, testes, imagem padrão e entrega (com reconhecimento de imagem se houver tempo).

**Aceite do MVP:** um usuário consegue, sem erros, criar conta, cadastrar seus ingredientes e visualizar uma lista de receitas sugeridas ordenada pela quantidade de ingredientes correspondentes à sua despensa.

## 8. Documentos de referência

- [docs/requisitos.md](docs/requisitos.md) — requisitos completos e conversa com o stakeholder.
- [docs/der.md](docs/der.md) — diagrama ER e decisões de modelagem.
- [docs/schema.sql](docs/schema.sql) — DDL PostgreSQL.
- [docs/diagrama-classes.md](docs/diagrama-classes.md) — diagrama de classes.
- [docs/modelagem-fluxo-app.md](docs/modelagem-fluxo-app.md) — fluxos e rastreabilidade.
