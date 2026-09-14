# Diagrama de Classes

Este documento apresenta o diagrama de classes do sistema, incluindo as entidades principais, serviços, provedores de dados e relacionamentos entre os componentes.


##  Estrutura do Sistema

```mermaid
classDiagram
    direction LR

    %% ENTIDADES

    class Usuario {
        -uuid id
        -string email
        -string senhaHash
        -Date criadoEm
    }

    class Ingrediente {
        -uuid id
        -uuid usuarioId
        -string nome
        -decimal quantidade
        -string unidade
        -Date criadoEm
        -Date atualizadoEm
    }

    class Receita {
        -uuid id
        -string nome
        -string modoPreparo
        -string imagemUrl
        -string fonte
        -string fonteId

        +ingredientes() ReceitaIngrediente[]
    }

    class ReceitaIngrediente {
        -uuid id
        -uuid receitaId
        -string nomeIngrediente
    }


    %% SERVIÇOS

    class AuthService {
        +registrar(email, senha) Usuario
        +login(email, senha) Token
        +hashSenha(senha) string
    }

    class DespensaService {
        +adicionarIngrediente(usuarioId, ingrediente) Ingrediente
        +editarIngrediente(usuarioId, ingredienteId, dados) Ingrediente
        +removerIngrediente(usuarioId, ingredienteId) void
        +listarIngredientes(usuarioId) Ingrediente[]
    }

    class RecipeSuggestionService {
        -RecipeProvider provider

        +buscarReceitasCompativeis(usuarioId) Receita[]
        +ranquearPorCorrespondencia(receitas, despensa) Receita[]
    }


    %% PROVIDERS

    class RecipeProvider {
        <<interface>>

        +buscarReceitas(ingredientes) Receita[]
    }

    class SpoonacularProvider {
        +buscarReceitas(ingredientes) Receita[]
    }

    class StaticDatasetProvider {
        +buscarReceitas(ingredientes) Receita[]
    }


    %% FUNCIONALIDADE FUTURA

    class ImageRecognitionService {
        <<Future>>

        -ImageRecognitionProvider provider

        +reconhecerIngredientes(imagem) string[]
    }

    class ImageRecognitionProvider {
        <<interface>>

        +detectar(imagem) string[]
    }


    %% RELACIONAMENTOS

    Usuario "1" --> "*" Ingrediente : possui

    Receita "1" *--> "*" ReceitaIngrediente : possui

    Usuario ..> AuthService : autentica-se via

    DespensaService ..> Usuario : associado a
    DespensaService ..> Ingrediente : gerencia

    RecipeSuggestionService o--> RecipeProvider : utiliza
    RecipeSuggestionService ..> Receita : gera ranking
    RecipeSuggestionService ..> ReceitaIngrediente : compara

    SpoonacularProvider ..|> RecipeProvider
    StaticDatasetProvider ..|> RecipeProvider

    StaticDatasetProvider ..> Receita : contingencia

    ImageRecognitionService o--> ImageRecognitionProvider : utiliza
    ImageRecognitionService ..> Ingrediente : sugere
```

## Relacionamentos Principais

| Relacionamento                 | Descrição                                                              |
| ------------------------------ | ---------------------------------------------------------------------- |
| `Usuario → Ingrediente`        | Um usuário possui vários ingredientes em sua despensa.                 |
| `Receita → ReceitaIngrediente` | Uma receita é composta por vários ingredientes.                        |
| `AuthService`                  | Responsável pelo registro e autenticação dos usuários.                 |
| `DespensaService`              | Gerencia os ingredientes da despensa do usuário.                       |
| `RecipeSuggestionService`      | Busca e ranqueia receitas compatíveis com os ingredientes disponíveis. |
| `RecipeProvider`               | Define um contrato para diferentes fontes de receitas.                 |
| `SpoonacularProvider`          | Implementa a integração com a API Spoonacular.                         |
| `StaticDatasetProvider`        | Fornece uma fonte alternativa de receitas para contingência.           |
| `ImageRecognitionService`      | Funcionalidade futura para identificação de ingredientes por imagem.   |
