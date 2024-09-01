# Nest Decoradores Personalizados

## Descrição

Decoradores personalizados no Nest.js permitem criar anotações próprias que podem ser usadas para adicionar metadados ou lógica personalizada a classes, métodos, parâmetros ou propriedades. Eles são baseados em decorators do TypeScript, mas com uma integração específica ao ecossistema do Nest.js, o que permite estender a funcionalidade de controladores, serviços e outros componentes.

## Principais Usos dos Decoradores Personalizados

1. **Adição de Metadados**: Adicionar informações adicionais a métodos, parâmetros ou classes que podem ser usadas por outras partes da aplicação, como guards, pipes, interceptors, etc.

2. **Validação de Dados**: Validar dados de entrada com base em regras específicas definidas por decoradores personalizados.

3. **Configuração de Comportamento**:  Alterar o comportamento padrão de métodos ou parâmetros com base em metadados adicionados por decoradores personalizados.

4. **Simplificação de Lógica**: Encapsular e reutilizar lógica comum em múltiplos locais da aplicação, reduzindo a repetição de código.

## Exemplo de Uso de Decoradores Personalizados

1. **Criando um Decorador para Capturar Dados do Usuário Autenticado**: Imagine que você quer criar um decorador que permita acessar diretamente os dados do usuário autenticado em um controlador:

   ```typescript
    import { createParamDecorator, ExecutionContext } from '@nestjs/common';

    export const User = createParamDecorator(
      (data: unknown, ctx: ExecutionContext) => {
        const request = ctx.switchToHttp().getRequest();
        return request.user;
      },
    );
   ```

   Este decorador personalizado, `@User()`, pode agora ser usado em qualquer controlador para acessar o usuário autenticado:

   ```typescript
    import { Controller, Get } from '@nestjs/common';

    @Controller('profile')
    export class ProfileController {
      @Get()
      getProfile(@User() user: any) {
        return user;
      }
    }
   ```

2. **Criando um Decorador para Customizar Rotas**: Suponha que você queira criar um decorador que simplifique a aplicação de um prefixo comum a certas rotas:

   ```typescript
    import { SetMetadata } from '@nestjs/common';

    export const RoutePrefix = (prefix: string) => SetMetadata('prefix', prefix);
   ```

   Você pode aplicar esse decorador em métodos de controlador:

   ```typescript
    import { Controller, Get } from '@nestjs/common';

    @Controller('items')
    export class ItemsController {
      @Get()
      @RoutePrefix('admin')
      findAll() {
        return "This is the admin route for items";
      }
    }
   ```

   Para fazer com que o decorador realmente modifique a rota, você precisaria de um interceptor ou outro middleware que processasse esse metadado e ajustasse o comportamento.

## Usando Decoradores Personalizados

Decoradores personalizados são poderosos porque permitem que você introduza e reutilize lógica complexa de maneira muito concisa. Eles são especialmente úteis em aplicações grandes, onde você precisa manter a lógica de autorização, validação ou manipulação de dados organizada e modular.

## Resumo

Decoradores Personalizados no Nest.js oferecem uma forma poderosa de estender e modularizar a funcionalidade da sua aplicação. Eles permitem encapsular lógica reutilizável e adicionar metadados que podem ser usados por outras partes da aplicação, promovendo um código mais limpo, organizado e fácil de manter. Com decoradores personalizados, você pode criar APIs mais expressivas e intuitivas, adaptadas às necessidades específicas do seu projeto.