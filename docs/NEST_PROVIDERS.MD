# Nest Providers

## Descrição

No Nest.js, providers são componentes que encapsulam a lógica de negócios e outras funcionalidades, como acesso a bancos de dados, manipulação de dados, e integração com APIs externas. Eles são a espinha dorsal da injeção de dependência no Nest.js, permitindo que serviços, repositórios, e outros componentes sejam injetados em controladores ou outros providers.

## O Que São Providers?

Um provider pode ser qualquer classe que o Nest.js possa injetar como dependência em outras partes da aplicação. Isso inclui:

- `Serviços (Services)`: Contêm a lógica de negócios da aplicação.
- `Repositórios (Repositories)`: Lidam com a comunicação com o banco de dados.
- `Fábricas (Factories)`: Criam instâncias de objetos que podem ser complexos de inicializar.
- `Gateways`: Usados para comunicação em tempo real, como WebSockets.

## Definindo um Provider

Para criar um provider, você define uma classe e a marca com o decorador `@Injectable()`. Isso informa ao Nest.js que esta classe pode ser injetada em outras classes que a dependam.

## Exemplo de um Serviço como Provider

Vamos considerar um exemplo de serviço (`UserService`) que será utilizado para manipular dados de usuários:

```typescript
import { Injectable } from '@nestjs/common';
import { UserRepository } from './user.repository';

@Injectable()
export class UserService {
  constructor(private readonly userRepository: UserRepository) {}

  findAll() {
    return this.userRepository.findAll();
  }

  findOne(id: string) {
    return this.userRepository.findOne(id);
  }

  create(data: any) {
    return this.userRepository.create(data);
  }
}
```

## Como Funciona a Injeção de Dependência?

A injeção de dependência permite que o Nest.js crie e gerencie instâncias dos providers, injetando-as automaticamente onde necessário. Quando você define um provider como `UserService` no exemplo acima, você pode injetá-lo em um controlador, em outro serviço, ou em qualquer lugar que precise dele.

## Exemplo de Injeção de Dependência em um Controlador

Aqui está um exemplo de um controlador que injeta o serviço `UserService` para lidar com operações relacionadas a usuários:

```typescript
import { Controller, Get, Post, Param, Body } from '@nestjs/common';
import { UserService } from './user.service';

@Controller('users')
export class UserController {
  constructor(private readonly userService: UserService) {}

  @Get()
  findAll() {
    return this.userService.findAll();
  }

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.userService.findOne(id);
  }

  @Post()
  create(@Body() data: any) {
    return this.userService.create(data);
  }
}
```

## Módulos e Providers

Os providers são geralmente registrados em módulos, que atuam como contêineres organizacionais. Para que um provider esteja disponível em um módulo, ele precisa ser incluído na lista de `providers` do módulo.

## Exemplo de Registro de Providers em um Módulo

Aqui está um exemplo de como o `UserService` pode ser registrado em um módulo de usuários:

```typescript
import { Module } from '@nestjs/common';
import { UserService } from './user.service';
import { UserRepository } from './user.repository';
import { UserController } from './user.controller';

@Module({
  providers: [UserService, UserRepository], // Registrando os providers
  controllers: [UserController], // Registrando os controladores
})
export class UserModule {}
```
## Resumo

- `Providers` são classes que encapsulam a lógica de negócios ou outras funcionalidades que podem ser injetadas em outras partes da aplicação.
- `Providers` são definidos com o decorador `@Injectable()` e podem ser injetados em controladores, serviços, ou outros providers.
- `Módulos` organizam os providers e os disponibilizam para outras partes da aplicação.

Os providers são essenciais para manter o código da aplicação modular, testável e fácil de manter, permitindo que a lógica de negócios seja isolada e reutilizada em diferentes partes da aplicação.