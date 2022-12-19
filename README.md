# CRUD-nodejs
Desafio: Essa aplicação realiza o CRUD (Create, Read, Update, Delete) de repositórios de projetos. Além disso, é possível dar likes em repositórios cadastrados, aumentando a quantidade de likes em 1 a cada vez que a rota é chamada.

Neste projeto é necessário identificar correções e melhorias para que os testes passem.

Abrir projeto com VSCode Online:

https://github1s.com/MyIgnite/CRUD-nodejs

Clone o projeto, navegue até a raiz do projeto e execute:</br>

`yarn` </br>
`yarn dev` </br>

Execute o comando para testar a aplicação </br>
`yarn test` </br> </br>

- [x] Should be able to create a new repository
- [x] Should be able to list the projects
- [x] Should be able to update repository
- [x] Should not be able to update a non existing repository
- [x] Should not be able to update repository likes manually
- [x] Should be able to delete the repository
- [x] Should be able to give a like to the repository
- [x] Should not be able to give a like to a non existing repository


# Código
Essa é uma parte de código de um aplicativo Node.js que utiliza o framework Express.

Na primeira linha, a biblioteca Express é importada com a função "require".

Na segunda linha, a função "uuid" é importada da biblioteca "uuid". A função "uuid" gera identificadores únicos de 128 bits como sequências de caracteres. O método "v4" é utilizado para gerar uma UUID aleatória.

Na quarta linha, o aplicativo Express é inicializado através da função "express()".

Na quinta linha, o middleware "express.json" é configurado para que o aplicativo possa processar requisições com corpos em formato JSON.

Na última linha, uma lista vazia de repositórios é criada (armazenada na variável "repositórios"). Essa lista será utilizada para armazenar os repositórios criados pelo aplicativo.

```js
const express = require("express");

const { v4: uuid } = require("uuid");

const app = express();

app.use(express.json());

const repositories = [];
```

## Middlewarer

Middlewarer que busca um repositório pelo seu "id", que é passado como parâmetro na requisição (através do objeto "request.params").

Se o repositório for encontrado, a função armazena o índice do repositório em "request.repositoryIndex" e chama a próxima função da cadeia de middlewares (através da chamada de "next()").

Se o repositório não for encontrado, a função retorna uma resposta de erro (código 404) com a mensagem "Repository not found".


```js
function findRepositoryByIndex(request, response, next) {
  const { id } = request.params

  const repositoryIndex = repositories.findIndex(
    (repository) => repository.id === id
  )

  if (repositoryIndex < 0) {
    return response.status(404).json({
      error: 'Repository not found.'
    })
  }

  request.repositoryIndex = repositoryIndex

  return next()
}
```

## GET

A função route handler com requisição do tipo GET é tratar a requisição e devolver uma resposta apropriada para o cliente. No caso, a resposta será a lista de repositórios em formato JSON recuperada a partir da variável "repositórios".

```js
app.get("/repositories", (request, response) => {
  return response.json(repositories);
});
```

## POST

A função route handler com requisição do tipo POST quando invocada obtém os dados enviados no corpo da requisição (através de "request.body") e cria um novo objeto "repositório" com esses dados. Esse objeto também tem um atributo "likes" inicializado com 0.

Em seguida, o objeto "repositório" é adicionado à lista de repositórios (armazenada na variável "repositórios") e uma resposta de sucesso é enviada para o cliente, contendo o objeto "repositório" criado como um JSON. A resposta também tem o código de status HTTP 201 (Created), indicando que um novo recurso foi criado com sucesso.

```js
app.post("/repositories", (request, response) => {
  const { title, url, techs } = request.body

  const repository = {
    id: uuid(),
    title,
    url,
    techs,
    likes: 0
  };

  repositories.push(repository);

  return response.status(201).json(repository);
});
```

## PUT

A função route handler com requisição do tipo PUT é invocado passando um ID de repositório como parâmetro na URL da requisição.

Antes de processar a requisição, essa função chama a função "findRepositoryByIndex", que é utilizada para localizar o repositório a ser atualizado na lista de repositórios (armazenada na variável "repositórios"). Se o repositório for encontrado, o índice dele na lista é armazenado em "request.repositoryIndex".

Depois disso, a função obtém os dados enviados no corpo da requisição (através de "request.body") e os utiliza para atualizar o repositório localizado. Se o objeto "request.body" tiver um atributo "likes", ele é removido para evitar que o número de likes do repositório seja alterado diretamente.

Em seguida, o repositório atualizado é armazenado na lista de repositórios e uma resposta é enviada para o cliente, contendo o repositório atualizado como um JSON.

```js
app.put("/repositories/:id", findRepositoryByIndex, (request, response) => {
  const updatedRepository = request.body;
  const { repositoryIndex } = request;

  if (updatedRepository.likes) {
    delete updatedRepository.likes
  }

  const repository = {
    ...repositories[repositoryIndex],
    ...updatedRepository
  };

  repositories[repositoryIndex] = repository;

  return response.json(repository);
});
```

## Delete

A função route handler com requisição do tipo DELETE é invocada passando um ID de repositório como parâmetro na URL da requisição.

Antes de processar a requisição, essa função chama a função "findRepositoryByIndex", que é utilizada para localizar o repositório a ser excluído na lista de repositórios (armazenada na variável "repositórios"). Se o repositório for encontrado, o índice dele na lista é armazenado em "request.repositoryIndex".

Depois disso, a função remove o repositório da lista de repositórios utilizando o método "splice". Esse método remove um ou mais elementos de uma lista e retorna os elementos removidos.

Por fim, a função envia uma resposta para o cliente com o código de status HTTP 204 (No Content), indicando que a operação de exclusão foi realizada com sucesso e não há conteúdo para ser retornado.

```js
app.delete("/repositories/:id", findRepositoryByIndex, (request, response) => {
  const { repositoryIndex } = request;

  repositories.splice(repositoryIndex, 1);

  return response.status(204).send();
});
```

## POST LIKES

A função route handler com requisição do tipo POST é invocada passando um ID de repositório é como parâmetro na URL da requisição.

Antes de processar a requisição, essa função chama a função "findRepositoryByIndex", que é utilizada para localizar o repositório cujo número de likes deve ser incrementado na lista de repositórios (armazenada na variável "repositórios"). Se o repositório for encontrado, o índice dele na lista é armazenado em "request.repositoryIndex".

Depois disso, a função incrementa o número de likes do repositório em 1 e armazena o novo valor em "likes". Em seguida, a função envia uma resposta para o cliente com o novo número de likes como um objeto JSON.

```js
app.post("/repositories/:id/like", findRepositoryByIndex, (request, response) => {
  const { repositoryIndex } = request;

  const likes = ++repositories[repositoryIndex].likes;

  return response.json({ likes });
});
```