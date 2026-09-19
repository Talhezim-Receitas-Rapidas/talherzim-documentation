# Guia de Commits — Conventional Commits

Padrão de mensagens de commit do Talherzim, baseado na especificação
[Conventional Commits v1.0.0](https://www.conventionalcommits.org/en/v1.0.0/).
Atende ao RNF08 (histórico de commits organizado por funcionalidade).

## 1. Formato

```
<tipo>[escopo opcional][!]: <descrição>

[corpo opcional]

[rodapé(s) opcional(is)]
```

- **tipo** — categoria da mudança (obrigatório).
- **escopo** — área afetada (opcional, entre parênteses).
- **`!`** — indica *breaking change* (opcional, antes dos dois-pontos).
- **descrição** — resumo curto no imperativo, sem ponto final.
- **corpo** — detalhes, motivação e o que mudou (opcional).
- **rodapé** — referências de issue e breaking changes (opcional).

### Regras de escrita

- Use o **imperativo** ("adiciona", não "adicionado" / "adicionando").
- Primeira letra **minúscula**, sem ponto final.
- Limite a linha da descrição a **~72 caracteres**.
- Uma mudança lógica por commit; evite commits gigantes e mistos.
- Corpo e rodapé separados do título por uma **linha em branco**.

## 2. Tipos permitidos

| Tipo | Quando usar |
|---|---|
| `feat` | Nova funcionalidade. |
| `fix` | Correção de bug. |
| `docs` | Apenas documentação. |
| `style` | Formatação, ponto e vírgula, sem mudança de comportamento. |
| `refactor` | Refatoração sem alterar comportamento. |
| `perf` | Melhoria de desempenho. |
| `test` | Adição/correção de testes. |
| `build` | Build, dependências, Docker, migrations. |
| `ci` | Configuração de CI/CD (GitHub Actions). |
| `chore` | Tarefas de manutenção que não se encaixam acima. |
| `revert` | Reversão de um commit anterior. |

## 3. Escopos do projeto

Escopos sugeridos (em minúsculas, um por commit quando fizer sentido):

| Escopo | Área |
|---|---|
| `auth` | Cadastro, login, logout, sessão (RF01, RF02). |
| `despensa` | CRUD de ingredientes (RF03–RF05, RF11). |
| `receitas` | Listagem/detalhe e dataset (RF06, RF08–RF10). |
| `matching` | Algoritmo de sugestão/ranqueamento (RF07). |
| `api` | Contrato OpenAPI e camada HTTP. |
| `db` | Schema, migrations e índices. |
| `ui` | Componentes e telas do front-end. |
| `infra` | Docker, ambientes, variáveis. |
| `ci` | Pipelines e automações. |
| `docs` | Documentação do repositório. |

## 4. Exemplos

```text
feat(auth): adiciona endpoint de login com JWT
```

```text
fix(auth): corrige expiração do token no middleware
```

```text
docs(api): documenta formato de erro do contrato
```

```text
refactor(despensa): extrai validação de ingrediente para service
```

```text
test(auth): cobre login com credenciais inválidas
```

```text
ci: adiciona lint e build no GitHub Actions
```

Commit com corpo e referência de issue:

```text
feat(despensa): adiciona CRUD de ingredientes

Cada ingrediente pertence ao usuário autenticado (RNF03).
A listagem retorna apenas os itens do próprio usuário.

Refs: #12
```

## 5. Breaking changes

Uma mudança incompatível deve ser sinalizada com `!` após o tipo/escopo **e**
descrita no rodapé com `BREAKING CHANGE:`.

```text
feat(api)!: altera resposta do login para incluir objeto usuario

BREAKING CHANGE: `POST /auth/login` deixou de retornar `user_id` e agora
retorna o objeto `usuario` com `id` e `email`. O front-end deve ser ajustado.
```

## 6. Commits que não seguem o padrão

- **Merge commits** (`Merge branch '...'`) são gerados pelo Git e não seguem o
  formato.
- Revert de um commit usa o tipo `revert`:

```text
revert: remove endpoint de logout experimental

Refs: 1a2b3c4
```

## 7. Checklist antes de commitar

- [ ] O tipo reflete a mudança (e não um "misto" de tudo)?
- [ ] A descrição está no imperativo e sem ponto final?
- [ ] O escopo existe na tabela da Seção 3?
- [ ] Breaking change sinalizada com `!` e `BREAKING CHANGE:`?
- [ ] Referências de issue no rodapé (`Refs: #NN`)?
- [ ] Um assunto lógico por commit?
