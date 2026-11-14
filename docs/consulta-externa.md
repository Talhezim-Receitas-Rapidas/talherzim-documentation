# Consulta externa de receitas

O Talherzim procura receitas no PostgreSQL e no dataset local. Para cada ingrediente da despensa que não aparece em nenhuma receita local, consulta a TheMealDB. Os resultados são reunidos na mesma lista e ordenados pela quantidade de ingredientes correspondentes. Receitas externas mantêm identificação própria e podem ser abertas novamente pelo identificador do provedor, inclusive nos favoritos.

O campo de busca também consulta a base local por nome de receita ou ingrediente. Quando não encontra resultado local, consulta o ingrediente na fonte externa, mesmo que ele não esteja cadastrado na despensa. Por exemplo, pesquisar `pepino` utiliza o termo `cucumber` no provedor. O frontend aguarda meio segundo após a digitação para iniciar a consulta. O parâmetro `busca` é usado junto de `detalhes=true` em `GET /receitas/sugeridas`.

A consulta usa a chave educacional `1` por padrão. A documentação do serviço está em https://www.themealdb.com/api.php. Configure `THEMEALDB_API_KEY` no backend para utilizar outra chave. Ingredientes comuns possuem um mapeamento de português para inglês. Termos desconhecidos são enviados como digitados, normalizados. Ao abrir os detalhes, o backend traduz nome, região, ingredientes, medidas e etapas para português usando MyMemory. A região culinária é a classificação informada pelo provedor.

As traduções completas ficam na tabela `traducoes_receitas` e são reutilizadas enquanto o conteúdo original permanecer igual. A tabela é criada na inicialização do backend. A tradução é automática e pode precisar de revisão. O serviço MyMemory tem limite gratuito anônimo de 5.000 caracteres por dia e recebe apenas os textos públicos da receita. Se houver falha ou limite atingido, a tela informa a indisponibilidade, mantém o original e oferece uma nova tentativa. Referências em https://mymemory.translated.net/doc/spec.php e https://mymemory.translated.net/doc/usagelimits.php.

São consultados até cinco ingredientes sem cobertura por requisição, com até três pratos por ingrediente. Cada chamada externa tem limite de oito segundos. Ausência de resultados e falhas externas geram mensagens próprias e preservam as sugestões locais. As receitas externas são consultadas no provedor, sem gravação no PostgreSQL.

Cada trecho enviado ao MyMemory tem até 480 bytes, abaixo do limite de 500 bytes do serviço. A tradução usa até quatro chamadas simultâneas por receita, com limite de dez segundos por chamada. A primeira abertura pode demorar mais porque consulta e traduz o conteúdo. A listagem mantém os nomes originais e a tradução acontece no detalhe.

## Contrato das rotas

Todas as rotas abaixo exigem `Authorization: Bearer <jwt>`. A autenticação usa o contrato de [contrato-api.md](contrato-api.md). JSON malformado retorna 400 com `json_invalido`. Corpos acima do limite do parser retornam 413 com `corpo_muito_grande`. Falhas inesperadas retornam HTTP 500 com `{ "erro": "erro_interno", "campos": [] }`.

### Despensa

| Método | Rota | Entrada | Sucesso |
|---|---|---|---|
| GET | `/ingredientes` | Sem corpo | 200 com array de ingredientes do usuário |
| POST | `/ingredientes` | `{ "nome": "pepino", "quantidade": 1, "unidade": "un" }` | 201 com ingrediente criado |
| PUT | `/ingredientes/:id` | Mesmo corpo do cadastro | 200 com ingrediente atualizado |
| DELETE | `/ingredientes/:id` | Sem corpo | 204 sem conteúdo |

Cada ingrediente contém `id`, `usuario_id`, `nome`, `quantidade`, `unidade` e datas de criação e atualização. A API aceita nome de 1 a 150 caracteres, quantidade opcional e unidade opcional de até 20 caracteres. Mantém a compatibilidade com quantidade zero ou ausente. Quantidades positivas devem estar entre 0.01 e 99999999.99, com armazenamento em duas casas decimais. O formulário exige quantidade positiva e oferece `un`, `g`, `kg`, `ml` e `l`. Entradas inválidas retornam 422 com `dados_invalidos` e mensagens em `campos`. UUID inválido ou item ausente retorna 404. Operações sobre itens de outro usuário retornam 403 com `acesso_negado`, preservando o contrato da equipe.

### Sugestões e pesquisa

`GET /receitas/sugeridas` mantém a resposta em array para os clientes existentes. Com `detalhes=true`, consulta os ingredientes da despensa autenticada e retorna o envelope com avisos e receitas externas. Nesse modo, o parâmetro opcional `busca` pesquisa nome de prato ou ingrediente na base local. Sem resultado local, o termo é usado como ingrediente na TheMealDB. Por exemplo, `GET /receitas/sugeridas?detalhes=true&busca=pepino`.

O parâmetro deve ser uma string de até 150 caracteres. Entrada inválida retorna 422. Uma consulta concluída retorna 200, inclusive quando a lista está vazia ou quando o provedor externo falha. Essas situações ficam em `avisos`.

```json
{
  "receitas": [],
  "avisos": [
    "Não encontramos receitas para ingrediente_exemplo na fonte externa. Confira o nome ou tente outro ingrediente."
  ],
  "despensaVazia": false
}
```

`despensaVazia` indica o estado real da despensa, mesmo durante uma busca digitada. Uma pesquisa pode encontrar receitas sem que o ingrediente pesquisado esteja na despensa. A pontuação continua contando os ingredientes disponíveis ao usuário. IDs externos seguem `externa-<idMeal>`. IDs locais são UUIDs do PostgreSQL ou identificadores do dataset como `estatico-002`.

| Situação | Mensagem de exemplo |
|---|---|
| Resultado externo | `Consulta externa realizada para pepino.` |
| Sem resultado externo | `Não encontramos receitas para ingrediente_exemplo na fonte externa. Confira o nome ou tente outro ingrediente.` |
| Falha ou demora externa | `A consulta externa para pepino está indisponível. Tente novamente.` |
| Mais de cinco ingredientes sem cobertura | `Consultamos até 5 ingredientes sem receitas por vez. Reduza a lista para consultar os demais.` |

### Detalhe

`GET /receitas/:id` retorna 200 com a receita encontrada ou 404 com o envelope `{ "erro": "recurso_nao_encontrado", "campos": [] }`. Indisponibilidade do provedor no detalhe retorna 503 com `A fonte externa está indisponível. Tente novamente.`. Os IDs do dataset continuam aceitos após a importação para PostgreSQL, preservando favoritos existentes.

| Campo | Conteúdo |
|---|---|
| `id`, `nome` | Identificação e nome do prato |
| `ingredientes` | Array de nomes nas receitas locais ou objetos `{ nome, quantidade }` nas externas |
| `modo_preparo` / `modoPreparo` | Texto local ou array de etapas externas. O frontend normaliza as duas formas |
| `imagem_url` / `imagem` | URL da imagem. O frontend aplica uma imagem padrão quando necessário |
| `fonte`, `externa` | Provedor e indicação de receita externa |
| `origem` | Região culinária declarada pelo provedor ou `null` |
| `fonteUrl` | Link HTTP/HTTPS original, quando fornecido pela fonte |
| `idioma` | `pt-BR` quando traduzida ou `en` no original externo |
| `traducaoAutomatica` | `true` após tradução concluída |
| `avisoTraducao` | Mensagem em português quando a tradução falha |
| `correspondencias`, `totalIngredientes` | Pontuação e tamanho da lista nas sugestões. O detalhe não recalcula a despensa |

A tradução concluída preserva o número e a ordem dos ingredientes e o conteúdo das instruções. Entidades HTML são decodificadas e marcadores de formatação removidos. Linhas que contêm apenas títulos de etapas são retiradas para evitar numeração duplicada. Traduções incompletas não são salvas.

Se a tradução falhar, o detalhe continua com HTTP 200 e o conteúdo original, acompanhado de `A tradução está indisponível no momento. Exibindo o conteúdo original. Tente novamente em instantes.`. O botão de nova tentativa repete a consulta.

## Persistência e atualização

`traducoes_receitas` contém `receita_id TEXT PRIMARY KEY`, `original TEXT NOT NULL` e `conteudo JSONB NOT NULL`. É um cache de textos públicos compartilhado. A chave do original considera nome, região, ingredientes e preparo. Alterações nesse conteúdo exigem nova tradução. A receita externa continua sendo consultada na TheMealDB.

Em bancos existentes, execute `npm run migrate` no backend ou reinicie o servidor para criar a tabela. O schema usa `CREATE TABLE IF NOT EXISTS`. A aplicação usa as mesmas tabelas de usuários, despensa e receitas. As integrações externas usam APIs nativas do Node.js. Os arquivos de lock incluem atualizações de segurança e a biblioteca de testes com PostgreSQL foi atualizada. Execute `npm ci` após atualizar o projeto. A pasta auxiliar `.demo` usada para a execução local fica fora dos três repositórios.

## Limitações atuais

- O catálogo externo pode não possuir receitas vinculadas a um ingrediente.
- A tradução de ingredientes da busca para inglês usa um mapa finito. Termos regionais, grafias incorretas e ingredientes ausentes do mapa podem não retornar resultados.
- A pesquisa externa usa filtro por ingrediente. Um nome de prato sem resultado local também é enviado a esse filtro e pode não ser reconhecido.
- O matching compara nomes normalizados e traduções conhecidas. Variações como `egg` e `eggs` podem reduzir a pontuação. Quantidades e unidades da despensa não entram no ranqueamento.
- As sugestões indicam correspondência de ingredientes e podem exigir outros itens para preparar o prato.
- A tradução automática pode conter termos culinários inadequados. O link original permanece disponível para consulta.
- A integração depende da disponibilidade dos provedores e da cota diária de tradução. As chamadas são feitas pelo backend.
- O contrato formal de autenticação, despensa e receitas está em `src/openapi.yaml` no backend. Este documento detalha o fluxo da consulta externa.

## Executar

1. No backend, copie `.env.example` para `.env` e configure um PostgreSQL acessível e `JWT_SECRET`. Com Docker disponível, `docker compose up -d` inicia o banco com as credenciais do `.env`.
2. Execute `npm ci` e `npm start` no backend. O servidor prepara as tabelas ao iniciar. Use Node.js 22 ou superior.
3. No frontend, copie `.env.example` para `.env`, execute `npm ci` e `npm run dev`. A URL padrão do backend é `http://localhost:3000`. O backend permite a origem `http://localhost:5173`. `CORS_ORIGIN` aceita uma lista de origens separadas por vírgula e tem prioridade sobre o alias `FRONTEND_ORIGIN`.
4. Crie uma conta e cadastre `arroz` e `lentilha` na despensa. Abra as sugestões. O arroz possui receitas locais e a lentilha dispara a consulta externa. Batata também já possui receitas no dataset local atual.
5. Abra um prato externo e confira imagem, ingredientes com medidas, preparo, região e link original quando disponível.

## Critérios de verificação

- A presença de uma receita local para arroz não impede a consulta externa de outro ingrediente sem receita.
- Resultados repetidos do provedor aparecem uma única vez.
- Despensa vazia, busca sem resultados e serviço indisponível produzem mensagens distintas.
- Cadastro, edição, remoção e listagem de ingredientes usam o usuário autenticado.
- `GET /receitas/sugeridas?detalhes=true` retorna `{ receitas, avisos, despensaVazia }`. Sem essa opção, mantém o array do contrato anterior. `GET /receitas/:id` retorna detalhes locais ou externos. As duas rotas exigem autenticação.
