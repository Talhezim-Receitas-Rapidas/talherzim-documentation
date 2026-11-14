# Roadmap — Talherzim

Plano de execução do MVP (1 mês / 4 semanas), derivado do [documento de requisitos](docs/requisitos.md) e do [resumo](resumo.md). O roadmap está organizado em fases sequenciais; as fases 1–4 são bloqueantes para o aceite, a fase 5 é opcional (*stretch goal*).

## Visão de alto nível

| Fase | Foco | Semana | Status |
|---|---|---|---|
| 0 | Setup e fundação | S1 | ⬜ |
| 1 | Autenticação de usuários | S1 | ⬜ |
| 2 | Despensa de ingredientes | S2 | ⬜ |
| 3 | Receitas, matching e ranqueamento | S2–S3 | ⬜ |
| 4 | Polimento, testes e entrega | S4 | ⬜ |
| 5 | Reconhecimento por foto (opcional) | S4+ | ⬜ |

---

## Fase 0 — Setup e fundação (S1)

- [ ] Confirmar a stack e criar os repositórios (`talherzim-backend`, `talherzim-frontend`).
- [ ] Scaffold do back-end: Node.js + Express, estrutura de pastas, ESLint/Prettier.
- [ ] Scaffold do front-end: React (ex.: Vite), roteamento e camada de chamadas HTTP.
- [ ] Configurar PostgreSQL e aplicar o [schema](docs/schema.sql) (migrations).
- [ ] Definir variáveis de ambiente e arquivo `.env.example` (sem segredos).
- [ ] Configurar CI básico (lint + build) e proteção de branch.

**Entregável:** projeto sobe localmente com banco criado e pipeline verde.

## Fase 1 — Autenticação de usuários (S1)

Requisitos: **RF01, RF02, RNF02, RNF03**.

- [ ] Endpoint de cadastro com validação de e-mail/senha.
- [ ] Hash de senha com bcrypt e login emitindo token (JWT).
- [ ] Middleware de autenticação e autorização por usuário.
- [ ] Telas de cadastro e login com tratamento de erros.
- [ ] Logout e persistência de sessão no front-end.

**Entregável:** usuário cria conta, entra, sai e só acessa os próprios dados.

## Fase 2 — Despensa de ingredientes (S2)

Requisitos: **RF03, RF04, RF05, RF11**.

- [ ] CRUD de ingredientes (nome, quantidade, unidade) restrito ao usuário autenticado.
- [ ] Listagem da despensa com edição e remoção.
- [ ] Validação de campos obrigatórios e feedback de erro.
- [ ] Normalização de nomes (case-insensitive) para o matching.

**Entregável:** usuário gerencia sua lista de ingredientes de ponta a ponta.

## Fase 3 — Receitas, matching e ranqueamento (S2–S3)

Requisitos: **RF06, RF07, RF08, RF09, RF10, RNF01, RNF07**.

- [ ] Definir a interface `RecipeProvider` e o modelo de receita.
- [ ] Importar o dataset estático (JSON) para `receitas`/`receita_ingredientes`.
- [ ] Integrar a API Spoonacular (com chave via env) e sincronização idempotente por `fonte_id`.
- [ ] Fallback automático para o dataset estático quando a API falhar (RNF07).
- [ ] Implementar o matching/ranqueamento (JOIN + contagem) e ordenar os resultados.
- [ ] Tela de listagem de receitas sugeridas com imagem (ou padrão).
- [ ] Tela de detalhe da receita (nome, ingredientes, modo de preparo, imagem).
- [ ] Garantir tempo de resposta ≤ 3 s (RNF01).

**Entregável:** usuário recebe receitas ordenadas pelas que aproveitam mais itens da despensa.

## Fase 4 — Polimento, testes e entrega (S4)

Requisitos: **RNF04, RNF08, RNF09**.

- [ ] Responsividade desktop/mobile (RNF04).
- [ ] Testes automatizados dos fluxos críticos (auth, CRUD, matching) e testes manuais.
- [ ] Tratamento de erros e estados de carregamento/vazio.
- [ ] Documentar API (ex.: OpenAPI) e atualizar READMEs.
- [ ] Revisão do [critério de aceite](resumo.md#7-prazo-e-critério-de-aceite) e demo final.

**Entregável:** MVP validado contra o critério de aceite, pronto para entrega.

## Fase 5 — Reconhecimento por foto (opcional / stretch)

Requisitos: **RF12–RF16, RNF10, RNF11, RNF12, RNF13**.

- [ ] Escolher o serviço de visão (Google Vision, Clarifai, Azure AI, etc.) por custo/limite.
- [ ] Upload/captura de imagem com limite de 5 MB.
- [ ] Mapear rótulos retornados (inglês) para os nomes de ingredientes da base.
- [ ] Tela de confirmação/edição dos itens reconhecidos antes de salvar.
- [ ] Fallback para cadastro manual quando o reconhecimento falhar.
- [ ] Política de descarte da imagem após processamento (privacidade).

**Entregável:** cadastro de ingredientes por foto, sem bloquear o restante do app em caso de falha.

---

## Backlog futuro (pós-MVP)

- Aplicativo mobile nativo (ex.: Flutter).
- Recomendação por machine learning/IA.
- Filtros avançados (restrição alimentar, tipo de dieta, tempo de preparo).
- Compartilhamento social de receitas.
- Avaliação/comentários de receitas.

## Riscos e mitigações

| Risco | Mitigação |
|---|---|
| Indisponibilidade/limite da API Spoonacular | Dataset estático como contingência (RNF07). |
| Atraso nas funcionalidades essenciais | Reconhecimento por imagem é opcional e movível para próxima iteração. |
| Qualidade do matching por nomes divergentes | Normalização case-insensitive e padronização dos nomes no dataset. |
| Escopo maior que o prazo de 1 mês | Priorização do stakeholder (P1 > P2 > P3) e backlog explícito. |
