# Contrato da API — Talherzim

Especificação legível dos endpoints de **autenticação** (RF01, RF02) da Semana 1.
O contrato formal, em OpenAPI 3.0.3, está em [`../openapi.yaml`](../openapi.yaml)
e pode ser aberto no [Swagger Editor](https://editor.swagger.io/) ou no Postman.

- **Base URL (local):** `http://localhost:3000`
- **Content-Type:** `application/json`
- **Versão do contrato:** `1.0.0`
- **Escopo atual:** autenticação. Despensa, sugestão e receitas entram nas fases
  seguintes do [roadmap](../roadmap.md).

---

## 1. Header de autenticação

As rotas protegidas exigem um **JWT** obtido em `POST /auth/login`, enviado no
header `Authorization` no formato `Bearer`:

```http
Authorization: Bearer <jwt>
```

| Situação | Resposta |
|---|---|
| Header ausente | `401 nao_autenticado` |
| Token malformado, expirado ou com assinatura inválida | `401 token_invalido` |

- O token é *stateless*: no logout o cliente deve **descartar o token**.
- A senha nunca trafega de volta; é armazenada com hash bcrypt (RNF02).
- Cada usuário só acessa os próprios dados (RNF03).

---

## 2. Formato de erro

Todos os erros usam o mesmo envelope JSON:

```json
{
  "erro": "codigo_estavel",
  "campos": [
    { "campo": "email", "mensagem": "E-mail inválido." }
  ]
}
```

- `erro` é um **código estável em `snake_case`** — o cliente deve comparar por
  ele, não pelo texto da mensagem.
- `campos` traz os erros de validação por campo e vem **vazio** (`[]`) quando o
  erro não é de validação.

### Códigos estáveis

| Código | HTTP típico | Quando ocorre |
|---|---|---|
| `dados_invalidos` | `422` | Campos ausentes, e-mail malformado ou senha curta. |
| `email_duplicado` | `409` | Já existe conta com o e-mail informado. |
| `credenciais_invalidas` | `401` | E-mail ou senha incorretos no login. |
| `nao_autenticado` | `401` | Rota protegida sem header `Authorization`. |
| `token_invalido` | `401` | Token malformado, expirado ou inválido. |

---

## 3. Endpoints

| Método | Rota | Auth | Sucesso | Erros |
|---|---|---|---|---|
| POST | `/auth/register` | não | `201 { id, email }` | `409`, `422` |
| POST | `/auth/login` | não | `200 { token, usuario }` | `401` |
| POST | `/auth/logout` | sim | `204` | `401` |
| GET | `/auth/me` | sim | `200 { id, email }` | `401` |

### 3.1 `POST /auth/register`

Cria uma conta com e-mail e senha (RF01). A senha é validada e armazenada com
hash bcrypt (RNF02). O e-mail deve ser único.

**Request body** — `RegistroRequest`

| Campo | Tipo | Obrigatório | Regras |
|---|---|---|---|
| `email` | string (`email`) | sim | Formato de e-mail válido e único. |
| `senha` | string (`password`) | sim | Mínimo de 8 caracteres. |

```json
{
  "email": "usuario@exemplo.com",
  "senha": "senhaSegura123"
}
```

**Respostas**

- `201` — `Usuario`

```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "email": "usuario@exemplo.com"
}
```

- `409` — `email_duplicado`

```json
{ "erro": "email_duplicado", "campos": [] }
```

- `422` — `dados_invalidos`

```json
{
  "erro": "dados_invalidos",
  "campos": [
    { "campo": "email", "mensagem": "E-mail inválido." },
    { "campo": "senha", "mensagem": "A senha deve ter ao menos 8 caracteres." }
  ]
}
```

### 3.2 `POST /auth/login`

Valida e-mail e senha e devolve um JWT + os dados do usuário (RF02).

**Request body** — `LoginRequest`

| Campo | Tipo | Obrigatório |
|---|---|---|
| `email` | string (`email`) | sim |
| `senha` | string (`password`) | sim |

```json
{
  "email": "usuario@exemplo.com",
  "senha": "senhaSegura123"
}
```

**Respostas**

- `200` — `LoginResponse`

```json
{
  "token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "usuario": {
    "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
    "email": "usuario@exemplo.com"
  }
}
```

- `401` — `credenciais_invalidas`

```json
{ "erro": "credenciais_invalidas", "campos": [] }
```

### 3.3 `POST /auth/logout`

Encerra a sessão autenticada (RF02). Sem corpo de resposta. Como o JWT é
*stateless*, o cliente deve descartar o token; o servidor invalida a sessão
quando houver lista de revogação.

**Header:** `Authorization: Bearer <jwt>`

**Respostas**

- `204` — sem corpo.
- `401` — `nao_autenticado` ou `token_invalido`.

### 3.4 `GET /auth/me`

Devolve os dados do usuário dono do token (RF02). Usado pelo front-end para
restaurar a sessão e proteger rotas.

**Header:** `Authorization: Bearer <jwt>`

**Respostas**

- `200` — `Usuario`

```json
{
  "id": "3fa85f64-5717-4562-b3fc-2c963f66afa6",
  "email": "usuario@exemplo.com"
}
```

- `401` — `nao_autenticado` ou `token_invalido`.

---

## 4. Schemas

| Schema | Campos | Uso |
|---|---|---|
| `Usuario` | `id` (uuid), `email` (email) | Representação pública; nunca inclui senha. |
| `RegistroRequest` | `email`, `senha` (min. 8) | Corpo de `/auth/register`. |
| `LoginRequest` | `email`, `senha` | Corpo de `/auth/login`. |
| `LoginResponse` | `token`, `usuario` | Resposta de `/auth/login`. |
| `Erro` | `erro`, `campos[]` | Envelope de todos os erros. |
| `CampoErro` | `campo`, `mensagem` | Item de `Erro.campos`. |

Os schemas seguem o modelo de dados de [`der.md`](der.md) e
[`schema.sql`](schema.sql): `usuarios(id UUID, email, senha_hash)` — o
`senha_hash` nunca é exposto pela API.

---

## 5. Exemplos com `curl`

```bash
# Cadastro
curl -X POST http://localhost:3000/auth/register \
  -H "Content-Type: application/json" \
  -d '{"email":"usuario@exemplo.com","senha":"senhaSegura123"}'

# Login (guarde o token retornado)
curl -X POST http://localhost:3000/auth/login \
  -H "Content-Type: application/json" \
  -d '{"email":"usuario@exemplo.com","senha":"senhaSegura123"}'

# Usuário autenticado
curl http://localhost:3000/auth/me \
  -H "Authorization: Bearer $TOKEN"

# Logout
curl -X POST http://localhost:3000/auth/logout \
  -H "Authorization: Bearer $TOKEN"
```

---

## 6. Convenções de mudança no contrato

O contrato é **congelado na Semana 1** (ver [semana-1.md](../semana-1.md)).
Alterações em rota, payload ou código de erro exigem alinhamento com o Lead e o
dev do front-end, além de atualizar **este documento e o [`../openapi.yaml`](../openapi.yaml)**
no mesmo PR, seguindo as [instruções de commit](commits.md).
