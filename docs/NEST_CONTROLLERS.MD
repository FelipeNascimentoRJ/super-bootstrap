# Nest Controllers

## Descrição

Controladores (ou controllers) no Nest.js são classes que lidam com as requisições HTTP e determinam o que deve ser feito com essas requisições. Eles são responsáveis por receber a entrada do cliente, processá-la (geralmente delegando essa tarefa para serviços), e devolver uma resposta apropriada.

## Função dos Controladores

Os controladores atuam como a camada intermediária entre o cliente (como um navegador web ou uma aplicação mobile) e a lógica de negócios da aplicação. Eles mapeiam as requisições HTTP (como `GET`, `POST`, `PUT`, `DELETE`) para métodos específicos dentro da classe do controlador.

## Estrutura Básica de um Controlador

Um controlador é definido como uma classe no Nest.js, usando o decorador `@Controller()`. Este decorador pode receber um caminho de URL, que será usado para definir a rota base para todos os métodos dentro daquele controlador.

## Exemplo de um Controlador

Aqui está um exemplo simples de um controlador que lida com operações relacionadas a usuários:

```typescript
import { Controller, Get, Post, Body, Param } from '@nestjs/common';
import { UserService } from './user.service';
import { CreateUserDto } from './dto/create-user.dto';

@Controller('users') // Define a rota base como '/users'
export class UserController {
  constructor(private readonly userService: UserService) {} // Injeção de dependência do serviço

  @Get() // Mapeia requisições GET para este método
  findAll() {
    return this.userService.findAll(); // Chama o serviço para obter todos os usuários
  }

  @Get(':id') // Mapeia requisições GET com um parâmetro para este método
  findOne(@Param('id') id: string) {
    return this.userService.findOne(id); // Chama o serviço para obter um usuário específico
  }

  @Post() // Mapeia requisições POST para este método
  create(@Body() createUserDto: CreateUserDto) {
    return this.userService.create(createUserDto); // Chama o serviço para criar um novo usuário
  }
}
```

## Explicação dos Decoradores

- `@Controller('users')`: Define que todas as rotas dentro deste controlador começam com `/users`. Por exemplo, `/users` e `/users/:id` serão mapeados para métodos dentro desta classe.
- `@Get()`: Mapeia requisições HTTP do tipo `GET` para o método que segue o decorador. O método `findAll()` será chamado quando uma requisição `GET` for feita para `/users`.
- `@Post()`: Mapeia requisições HTTP do tipo `POST` para o método que segue o decorador. O método `create()` será chamado quando uma requisição `POST` for feita para `/users`.
- `@Param('id')`: Extrai um parâmetro da URL (neste caso, o `id` do usuário) e o injeta como argumento no método `findOne()` quando uma requisição `GET` for feita para `/users/:id`.
- `@Body()`: Extrai o corpo da requisição (geralmente um objeto JSON) e o injeta como argumento no método `create()` quando uma requisição `POST` for feita para `/users`.

## Integração com Serviços

Controladores geralmente não contêm a lógica de negócios. Em vez disso, eles delegam o trabalho para serviços, que são injetados no controlador através do construtor. Isso permite que a lógica de negócios seja testada separadamente dos controladores e facilita a manutenção.

## Resumo

- `Controladores` no Nest.js lidam com requisições HTTP e determinam como processar essas requisições.
- `Controladores` mapeiam rotas para métodos específicos, utilizando decoradores como `@Get()`, `@Post()`, entre outros.
- `Controladores` normalmente delegam a lógica de negócios para serviços, mantendo o código organizado e separado em diferentes responsabilidades.

Entender o papel dos controladores é fundamental para construir APIs RESTful de maneira eficaz e organizada no Nest.js.