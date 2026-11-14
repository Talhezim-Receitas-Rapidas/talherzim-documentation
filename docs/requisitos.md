# Documento de Levantamento de Requisitos

**Projeto:** Organizador de Receitas com Sugestão por Ingredientes Disponíveis (Talherzim)

---

## Parte 1 — Registro da Conversa com o Stakeholder

**Analista:** Bom dia! Vamos conversar sobre a ideia do aplicativo de receitas para entender melhor o que o senhor precisa. Pode me explicar, com suas palavras, qual problema esse app resolve?

**Stakeholder:** Bom dia. A ideia é simples: muita gente tem ingredientes em casa e não sabe o que cozinhar com eles, ou acaba jogando comida fora porque não lembrou que tinha aquele item na geladeira. Quero um app onde a pessoa cadastra o que tem disponível e o sistema sugere receitas possíveis, priorizando as que aproveitam mais itens que ela já possui.

**Analista:** Entendi. Quem seria o usuário principal desse app?

**Stakeholder:** Qualquer pessoa que cozinha em casa no dia a dia — desde quem está começando a cozinhar até quem já tem mais prática, mas quer economizar tempo e evitar desperdício de comida.

**Analista:** E como o usuário vai informar os ingredientes que tem disponíveis?

**Stakeholder:** Ele deve conseguir cadastrar manualmente, digitando o nome do ingrediente e, se possível, a quantidade. Não precisa ser nada muito sofisticado nessa primeira versão — o importante é ele conseguir montar uma "lista da despensa" e editar essa lista quando quiser.

**Analista:** Certo. E sobre as receitas — de onde elas vêm? O sistema vai ter uma base própria ou vai buscar em algum lugar?

**Stakeholder:** Pensei em usar uma API pública de receitas, tipo a Spoonacular, para não precisarmos cadastrar tudo manualmente. Mas se não der tempo de integrar isso, um dataset estático (um arquivo JSON com receitas pré-cadastradas) já resolve para essa primeira entrega.

**Analista:** Sobre a sugestão em si: como o senhor imagina que o "matching" entre ingredientes e receitas deve funcionar?

**Stakeholder:** O sistema deve comparar a lista de ingredientes do usuário com os ingredientes de cada receita e mostrar primeiro as receitas que usam o maior número de itens que ele já tem em casa. Não precisa ser um algoritmo de inteligência artificial complexo — uma lógica de comparação e pontuação (quantos ingredientes batem) já atende.

**Analista:** O senhor mencionou fotos das receitas. Isso é obrigatório nessa primeira versão?

**Stakeholder:** É desejável, deixa a interface mais agradável, mas não é bloqueante. Se a API de receitas já trouxer imagem, ótimo, usamos ela. Se não tivermos tempo de tratar isso direito, colocamos uma imagem padrão no lugar.

**Analista:** Falando em prazo: temos um mês para entregar esse projeto. Como o senhor prioriza as funcionalidades dentro desse tempo?

**Stakeholder:** Prioridade 1: cadastro de ingredientes e o mecanismo de sugestão de receitas funcionando de ponta a ponta, mesmo que simples. Prioridade 2: cadastro/login de usuário, para cada um ter sua própria lista. Prioridade 3: visual bonito, fotos, filtros extras (tipo de prato, tempo de preparo). Se sobrar tempo, é o que refinamos por último.

**Analista:** E em relação à plataforma — o senhor imagina isso como app mobile, web, ou os dois?

**Stakeholder:** Para o prazo que temos, prefiro fechar em uma aplicação web responsiva primeiro. Se depois quisermos, dá pra pensar num app mobile de fato (Flutter, por exemplo), mas não em um mês.

**Analista:** Sobre tecnologia, o senhor tem alguma preferência ou restrição?

**Stakeholder:** Não é uma exigência rígida, mas gostaria de usar React no front-end e Node.js/Express no back-end, já que a equipe tem mais familiaridade. Para o banco, pode ser PostgreSQL ou MongoDB — o que for mais rápido de implementar dentro do prazo.

**Analista:** Existe alguma preocupação com desempenho ou volume de dados?

**Stakeholder:** Não esperamos um volume gigante de usuários nessa primeira versão, é mais uma prova de conceito. Mas a busca de receitas por ingrediente precisa ser rápida — ninguém quer esperar muito tempo depois de cadastrar os itens.

**Analista:** E quanto à segurança dos dados dos usuários?

**Stakeholder:** O básico: senha protegida (com hash, não texto puro), e cada usuário só pode ver e editar a própria lista de ingredientes e não a de outras pessoas.

**Analista:** Depois da nossa primeira conversa, pensamos em uma funcionalidade extra: o usuário enviar uma foto da geladeira ou da despensa para o sistema reconhecer os ingredientes automaticamente. O que o senhor acha disso?

**Stakeholder:** Gostei da ideia, facilita bastante a vida do usuário. Mas entendo que é uma funcionalidade mais complexa, envolvendo reconhecimento de imagem. Podemos incluir como um adicional, desde que não coloque em risco a entrega do que já combinamos para o primeiro mês. Se der tempo depois das funcionalidades principais, ótimo; senão, fica para uma próxima etapa.

**Analista:** Por último, como o senhor vai considerar esse projeto "pronto" ao final do mês?

**Stakeholder:** Quando um usuário conseguir: criar conta, cadastrar os ingredientes que tem, e receber uma lista de receitas sugeridas, ordenada pelas que aproveitam mais itens da despensa dele. Isso é o mínimo que precisa funcionar sem erros. A análise de imagem é um bônus, não um bloqueio.

---

## Parte 2 — Documento Técnico de Requisitos

### 1. Identificação do Projeto

| Campo | Descrição |
|---|---|
| Nome do Projeto | Organizador de Receitas com Sugestão por Ingredientes Disponíveis |
| Tipo | Aplicação Web Responsiva (MVP) |
| Duração do Ciclo | 1 mês (4 semanas) |
| Stakeholder | Cliente/idealizador do produto |

### 2. Objetivo

Desenvolver uma aplicação web onde o usuário cadastra os ingredientes disponíveis em sua residência e recebe sugestões de receitas que podem ser preparadas com o que já possui, priorizando as receitas que aproveitam o maior número de itens disponíveis, reduzindo desperdício de alimentos e facilitando a decisão do que cozinhar.

### 3. Escopo do MVP (adequado ao prazo de 1 mês)

**Incluso no escopo:**

- Cadastro e autenticação de usuários
- Cadastro, edição e remoção de ingredientes na "despensa" do usuário
- Base de receitas via API externa (Spoonacular) ou dataset estático (JSON) como alternativa
- Algoritmo de matching e ranqueamento por quantidade de ingredientes correspondentes
- Listagem de receitas sugeridas com detalhes básicos (nome, ingredientes, modo de preparo, imagem quando disponível)
- Análise de imagem com reconhecimento de ingredientes (funcionalidade adicional/opcional — ver Seção 6)

**Fora do escopo desta entrega (backlog futuro):**

- Aplicativo mobile nativo
- Recomendação por machine learning/IA na sugestão de receitas
- Filtros avançados (restrição alimentar, tipo de dieta, tempo de preparo)
- Compartilhamento social de receitas
- Avaliação/comentários de usuários sobre receitas

### 4. Requisitos Funcionais (RF)

Os itens marcados como **[Adicional]** correspondem à funcionalidade de análise de imagem com reconhecimento de ingredientes, detalhada na Seção 6, e devem ser tratados como incremento opcional frente ao prazo do projeto.

| ID | Requisito | Prioridade |
|---|---|---|
| RF01 | O sistema deve permitir que o usuário crie uma conta com e-mail e senha. | Alta |
| RF02 | O sistema deve permitir que o usuário faça login e logout. | Alta |
| RF03 | O sistema deve permitir que o usuário cadastre um ingrediente informando nome e quantidade. | Alta |
| RF04 | O sistema deve permitir que o usuário edite ou remova um ingrediente já cadastrado. | Alta |
| RF05 | O sistema deve exibir a lista completa de ingredientes cadastrados pelo usuário. | Alta |
| RF06 | O sistema deve buscar receitas compatíveis com os ingredientes cadastrados pelo usuário. | Alta |
| RF07 | O sistema deve ranquear as receitas sugeridas pela quantidade de ingredientes correspondentes entre a receita e a despensa do usuário. | Alta |
| RF08 | O sistema deve exibir, para cada receita sugerida, nome, lista de ingredientes, modo de preparo e imagem (quando disponível). | Média |
| RF09 | O sistema deve exibir uma imagem padrão quando a receita não possuir imagem disponível na fonte de dados. | Baixa |
| RF10 | O sistema deve permitir que o usuário visualize os detalhes completos de uma receita selecionada. | Média |
| RF11 | O sistema deve associar cada lista de ingredientes exclusivamente à conta do usuário autenticado. | Alta |
| RF12 | O sistema deve permitir que o usuário envie uma foto de ingredientes através da interface web (upload de arquivo ou captura pela câmera do dispositivo). **[Adicional]** | Média |
| RF13 | O sistema deve processar a imagem enviada e retornar uma lista de ingredientes identificados, utilizando um serviço de reconhecimento de imagem. **[Adicional]** | Média |
| RF14 | O sistema deve exibir ao usuário os ingredientes reconhecidos na imagem antes de adicioná-los à despensa, permitindo confirmação, edição ou remoção de itens sugeridos. **[Adicional]** | Média |
| RF15 | O sistema deve adicionar à lista de ingredientes do usuário apenas os itens confirmados por ele após o reconhecimento. **[Adicional]** | Média |
| RF16 | O sistema deve informar ao usuário quando a imagem não permitir identificar nenhum ingrediente com confiança suficiente, oferecendo a opção de cadastro manual. **[Adicional]** | Baixa |

### 5. Requisitos Não Funcionais (RNF)

Os itens marcados como **[Adicional]** correspondem à funcionalidade de análise de imagem com reconhecimento de ingredientes, detalhada na Seção 6.

| ID | Requisito | Categoria |
|---|---|---|
| RNF01 | O tempo de resposta da busca por receitas sugeridas não deve ultrapassar 3 segundos em condições normais de uso. | Desempenho |
| RNF02 | As senhas dos usuários devem ser armazenadas utilizando hash (ex.: bcrypt), nunca em texto puro. | Segurança |
| RNF03 | Um usuário não deve conseguir acessar ou modificar dados de ingredientes de outro usuário. | Segurança |
| RNF04 | A interface deve ser responsiva, funcionando adequadamente em navegadores desktop e mobile. | Usabilidade |
| RNF05 | O sistema deve ser desenvolvido utilizando React no front-end e Node.js/Express no back-end. | Tecnologia |
| RNF06 | O sistema deve persistir dados em um banco de dados relacional (PostgreSQL) ou não relacional (MongoDB). | Tecnologia |
| RNF07 | O sistema deve tratar indisponibilidade da API externa de receitas, utilizando o dataset estático como contingência. | Confiabilidade |
| RNF08 | O código-fonte deve ser versionado em repositório Git com histórico de commits organizado por funcionalidade. | Manutenibilidade |
| RNF09 | O sistema deve suportar, no mínimo, uso simultâneo por um pequeno grupo de usuários de teste sem degradação perceptível. | Escalabilidade |
| RNF10 | O processamento de reconhecimento de imagem deve retornar um resultado em até 8 segundos em condições normais de uso. **[Adicional]** | Desempenho |
| RNF11 | Imagens enviadas pelo usuário devem ser armazenadas ou descartadas conforme política de privacidade definida (recomenda-se não reter a imagem após o processamento, salvo consentimento explícito). **[Adicional]** | Privacidade |
| RNF12 | O tamanho máximo de imagem aceito para upload deve ser limitado (ex.: 5 MB) para evitar sobrecarga do serviço. **[Adicional]** | Desempenho |
| RNF13 | O sistema deve tratar falhas ou indisponibilidade do serviço de reconhecimento de imagem sem interromper o restante da aplicação, retornando mensagem clara ao usuário. **[Adicional]** | Confiabilidade |

### 6. Funcionalidade Adicional: Análise de Imagem com Reconhecimento de Ingredientes

#### 6.1 Descrição

Além do cadastro manual de ingredientes, o sistema pode oferecer a possibilidade de o usuário enviar uma foto (por exemplo, da geladeira, da despensa ou dos itens sobre a bancada) para que o sistema identifique automaticamente os ingredientes presentes na imagem e os adicione à lista de ingredientes disponíveis do usuário, mediante confirmação.

O objetivo é reduzir o esforço de cadastro manual, tornando o processo de "informar o que tenho em casa" mais rápido e mais próximo da experiência real do usuário.

#### 6.2 Impacto no Escopo e no Prazo

Esta funcionalidade envolve reconhecimento de imagem (visão computacional), o que representa uma complexidade técnica adicional relevante frente ao ciclo de desenvolvimento de 1 mês já definido para o MVP. Recomenda-se tratá-la como um incremento opcional (*stretch goal*), a ser desenvolvido somente após a estabilização das funcionalidades essenciais (cadastro de ingredientes, autenticação e sugestão de receitas).

Alternativa de menor risco para caber no prazo: integrar uma API de visão computacional já pronta (reconhecimento de objetos/alimentos), em vez de treinar um modelo próprio, e limitar o reconhecimento a uma lista de ingredientes comuns pré-definida.

#### 6.3 Considerações Técnicas

Opções de serviço de reconhecimento de imagem a avaliar quanto a custo, limite de uso gratuito e facilidade de integração:

- Google Cloud Vision API (detecção de objetos/rótulos)
- Clarifai (modelos pré-treinados para alimentos)
- Azure AI Vision (Computer Vision)
- Serviços especializados em reconhecimento de alimentos (ex.: LogMeal, Foodvisor), quando disponíveis para o volume de testes do projeto

Em qualquer caso, recomenda-se mapear o rótulo retornado pela API (geralmente em inglês, ex. "tomato", "onion") para os nomes de ingredientes já usados na base de receitas do sistema, para garantir compatibilidade com o algoritmo de sugestão já especificado (RF06 e RF07).

### 7. Restrições do Projeto

- Prazo total de desenvolvimento: 1 mês.
- Equipe com familiaridade prévia em React e Node.js — priorizar essas tecnologias para não gerar curva de aprendizado adicional.
- Escopo de plataforma limitado a aplicação web nesta entrega.
- Dependência de API externa (Spoonacular ou similar); necessidade de plano de contingência com dados estáticos.
- A funcionalidade de reconhecimento de imagem é condicionada à disponibilidade de tempo após a entrega das funcionalidades essenciais.

### 8. Cronograma Sugerido (4 semanas)

| Semana | Entregas |
|---|---|
| Semana 1 | Levantamento de requisitos (este documento), modelagem do banco de dados, setup do projeto (front-end e back-end), autenticação de usuários. |
| Semana 2 | CRUD de ingredientes (cadastro/edição/remoção), integração inicial com a API/dataset de receitas. |
| Semana 3 | Implementação do algoritmo de matching e ranqueamento, tela de listagem e detalhe de receitas. |
| Semana 4 | Ajustes de interface, tratamento de erros, testes gerais, imagem padrão para receitas sem foto, entrega final. Se houver tempo disponível: integração da análise de imagem com reconhecimento de ingredientes (RF12-RF16). Caso contrário, mover para próxima iteração. |

### 9. Critério de Aceite do MVP

O projeto será considerado entregue quando um usuário conseguir, sem erros: criar uma conta, cadastrar seus ingredientes disponíveis (manualmente e/ou por reconhecimento de imagem, se essa etapa for concluída dentro do prazo), e visualizar uma lista de receitas sugeridas ordenada pela quantidade de ingredientes correspondentes à sua despensa.
