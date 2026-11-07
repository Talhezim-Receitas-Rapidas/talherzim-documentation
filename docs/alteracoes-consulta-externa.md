# Alterações na consulta de receitas

Ingredientes sem receitas na base local passam a ser consultados na TheMealDB. Os resultados aparecem junto das sugestões locais, com identificação da fonte externa. Ingredientes, quantidades e modo de preparo são traduzidos para português ao abrir a receita.

Quando a consulta não encontra resultados ou o serviço está indisponível, a tela informa o motivo e permite tentar novamente. A configuração e o contrato das rotas estão em [Consulta externa de receitas](consulta-externa.md).

## Backend

| Arquivo | Implementação |
|---|---|
| `src/providers/theMealDBProvider.js` | Busca por ingrediente, mapeamento português/inglês e consulta de detalhes com imagem, medidas, preparo, região e fonte |
| `src/services/recipeSuggestionService.js` | Reúne PostgreSQL e dataset, detecta ingredientes sem cobertura, consulta fonte externa, elimina receitas repetidas e ordena por correspondências |
| `src/services/translationService.js` | Traduz detalhes com MyMemory, divide textos por bytes, trata entidades e formatação e salva apenas traduções completas |
| `src/routes/ingredientes.js` | CRUD autenticado com validação de entrada e filtro por usuário |
| `src/routes/receitas.js` | Rotas autenticadas de sugestões, pesquisa e detalhe |
| `src/app.js`, `src/index.js` | Conecta os serviços ao servidor e configura CORS por origem |
| `db/schema.sql`, `initdb.d/schema.sql` | Acrescenta a tabela idempotente de cache `traducoes_receitas` |
| `.env.example` | Documenta `FRONTEND_ORIGIN`, `THEMEALDB_API_KEY` e valores locais de host e porta do PostgreSQL |
| `src/tests/recipeSuggestion.unit.test.js` | Regressões de consulta complementar, pesquisa, mensagens, deduplicação e entrada pelas rotas autenticadas |
| `src/tests/translation.unit.test.js` | Divisão por bytes, tradução dos campos, cache, limite do provedor e decodificação de entidades |
| `src/tests/receitas.test.js` | Integração de despensa, receitas, importação e cache com banco SQL |
| `src/models/receitaModel.js` | Corrige o conflito de importação para usar o índice único parcial de fonte e identificador |
| `src/tests/auth.test.js` | Corrige os helpers de requisição para preservar a API de asserções do Supertest |
| `package.json`, `package-lock.json` | Atualiza a biblioteca de testes PostgreSQL, inclui as novas integrações no script de testes e corrige dependências vulneráveis |
| `README.md` | Atualiza as rotas, configuração e orientação de migração |
| `src/openapi.yaml` | Inclui os contratos de despensa, sugestões, pesquisa, detalhe e campos de tradução na versão 1.1.0 |

## Frontend

| Arquivo | Implementação |
|---|---|
| `src/features/receitas/receitaService.ts` | Usa a API local por padrão, preserva a função que retorna array e acrescenta consulta com avisos, procedência e tradução |
| `src/features/receitas/RecipesListPage.tsx` | Pesquisa no backend após 500 ms sem digitação, separa o cache por termo e mostra mensagens e estados vazios |
| `src/features/receitas/RecipeCard.tsx` | Identifica receitas encontradas em fonte externa |
| `src/features/receitas/RecipeDetailPage.tsx` | Mostra fonte, região, tradução e ação de nova tentativa. Corrige retorno para a rota de sugestões |
| `.env.example`, `README.md` | Documenta a URL padrão da API e o comportamento integrado |
| `src/app/router.tsx` | Carrega as páginas protegidas sob demanda e reduz o arquivo JavaScript inicial |
| `src/services/httpClient.ts` | Usa a API local também quando a variável de URL está vazia |
| `src/features/despensa/despensaSchema.ts` | Alinha os limites de nome e quantidade ao backend |
| `src/pages/auth/LoginPage.tsx`, `src/pages/auth/RegisterPage.tsx`, `src/features/auth/RegisterForm.tsx` | Alinha a validação e a indicação de senha mínima aos oito caracteres exigidos pela API |
| `package-lock.json` | Atualiza as versões resolvidas das dependências com correções de segurança |

## Correções de comportamento

- IDs inválidos da despensa retornam 404 antes de chegar ao PostgreSQL.
- Quantidades inferiores a 0,01 são rejeitadas para impedir gravação arredondada para zero.
- Corpo ausente, JSON inválido e corpo acima do limite recebem respostas de validação.
- Receitas externas válidas são mantidas quando outro detalhe da mesma consulta falha.
- IDs antigos do dataset continuam abrindo receitas após a importação para PostgreSQL.
- A importação usa transação e o índice único parcial correto. Reexecutar a carga não duplica receitas.
- O cache de sugestões é separado por sessão e termo pesquisado.
- Sessões expiradas são encerradas também nas chamadas do serviço de receitas.
- A URL de API vazia usa o backend local. Foram removidos os dados simulados do serviço de receitas.
- A listagem informa receitas encontradas e mantém a pontuação dos ingredientes disponíveis.

## Testes

| Verificação | Resultado e abrangência |
|---|---|
| `npm test` no backend | 105 testes aprovados em nove suítes, incluindo 40 testes de integração |
| `npm run lint` no backend | Sem erros ou avisos |
| `npm run lint` no frontend | Aprovado |
| `npx tsc -b` no frontend | Aprovado após as alterações de tradução |
| `npm run build` no frontend | Aprovado sem avisos. Arquivo inicial de aproximadamente 407 kB antes da compressão |
| Verificação de formatação | Arquivos novos e alterados da integração aprovados pelo Prettier |
| Contrato OpenAPI e links | YAML e referências internas verificados em oito caminhos e dez operações. Links locais conferidos em sete documentos |
| `npm audit --omit=dev` | Nenhuma vulnerabilidade nas dependências de produção dos dois projetos |
| Consulta externa real | Busca por pepino retornou três receitas da TheMealDB |
| Demonstração no navegador | Login, despensa com ovo e pepino, sugestões e detalhe da Salada de camarão Bang Bang com ingredientes, medidas e preparo traduzidos |

Os testes de integração usaram um banco PGlite isolado em memória, acessado pelo protocolo PostgreSQL com o driver `pg`. Aplicaram o schema completo, incluindo `pgcrypto`, e executaram as rotas com autenticação real, transações e consultas SQL. As respostas dos provedores externos foram simuladas nessa suíte para tornar os testes reproduzíveis. A consulta à TheMealDB e a tradução também foram verificadas no ambiente local. A execução com PostgreSQL em Docker depende de Docker disponível.

## Dependências

As atualizações corrigiram os alertas altos identificados. A auditoria completa ainda aponta 16 alertas no backend, sendo seis baixos e dez moderados, e dez moderados no frontend. Todos estão nas dependências de desenvolvimento e teste. Envolvem ferramentas como ESLint, Jest e Testcontainers e suas dependências transitivas.

As versões de produção não apresentaram alertas na auditoria executada. Os arquivos de lock devem acompanhar os commits para reproduzir as versões verificadas.

## Atualização do banco

Execute `npm run migrate` no backend para criar a tabela `traducoes_receitas`. O servidor também cria a tabela ao iniciar. As variáveis de configuração estão nos arquivos `.env.example`.

O frontend e o backend devem ser atualizados juntos para exibir os avisos das consultas externas.

## Compatibilidade com a develop

A integração reutiliza os controllers, models e serviços de despensa e receitas existentes na equipe. O serviço de consulta externa estende `ReceitaService`, preservando o ranqueamento e o fallback anteriores. `GET /receitas/sugeridas` mantém a resposta em array. O novo frontend usa `detalhes=true` para receber o envelope de avisos e habilitar a pesquisa externa. A configuração de CORS mantém `CORS_ORIGIN` e aceita `FRONTEND_ORIGIN` como alias. Os testes existentes de autenticação, despensa e receitas continuam incluídos na validação.
