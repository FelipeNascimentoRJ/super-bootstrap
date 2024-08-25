# Nest Middlewares

## Descrição

No Nest.js, `middlewares` são funções que podem ser executadas durante o ciclo de vida de uma requisição HTTP, antes que ela alcance o controlador. Eles são usados para realizar tarefas comuns, como autenticação, logging, manipulação de requisições, validações ou qualquer outra lógica que você deseja aplicar a todas (ou a uma seleção) de requisições.

## Características dos Middlewares

- `Interceptação de Requisições`: Middlewares têm acesso à requisição antes que ela chegue ao controlador e à resposta antes de ser enviada ao cliente.
- `Execução Sequencial`: Eles podem ser configurados para serem executados em uma ordem específica, de acordo com a lógica da aplicação.
- `Reutilizáveis`: Middlewares podem ser aplicados globalmente, em módulos específicos ou em rotas específicas.

## Exemplo de Uso de Middleware

Vamos criar um `middleware` simples que registra o tempo que uma requisição leva para ser processada.

1. **Criando o Middleware**: Primeiro, você define a função do middleware em uma classe que implementa a interface `NestMiddleware`:

   ```typescript
    import { Injectable, NestMiddleware } from '@nestjs/common';
    import { Request, Response, NextFunction } from 'express';

    @Injectable()
    export class LoggerMiddleware implements NestMiddleware {
      use(req: Request, res: Response, next: NextFunction) {
        console.log(`Request...`);
        const start = Date.now();

        res.on('finish', () => {
          const duration = Date.now() - start;
          console.log(`Request to ${req.method} ${req.url} took ${duration}ms`);
        });

        next(); // Passa o controle para o próximo middleware ou para o controlador
      }
    }
   ```

   Neste exemplo:

   - `use(req, res, next)`: Este é o método onde você implementa a lógica do middleware. Ele recebe os objetos `Request`, `Response`, e a função next que você deve chamar para continuar o processamento da requisição.

   - `res.on('finish', ...)`: Este código escuta o evento `finish` do objeto de resposta para calcular o tempo de processamento da requisição.

   - `next()`: Chama o próximo middleware na fila, ou o controlador se não houver mais middlewares.

2. **Aplicando o Middleware**: Depois de criar o middleware, você precisa aplicá-lo em um módulo ou em rotas específicas. Isso pode ser feito dentro do módulo onde deseja que ele seja executado.

## Aplicação Global

Para aplicar o middleware em todas as rotas da aplicação:

```typescript
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './logger.middleware';
import { UserModule } from './user/user.module';

@Module({
  imports: [UserModule],
})
export class AppModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware) // Aplica o LoggerMiddleware
      .forRoutes('*'); // Em todas as rotas
  }
}
```

## Aplicação em Rotas Específicas

Você pode aplicar o middleware apenas em rotas ou controladores específicos:

```typescript
import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
import { LoggerMiddleware } from './logger.middleware';
import { UserController } from './user/user.controller';

@Module({
  controllers: [UserController],
})
export class UserModule implements NestModule {
  configure(consumer: MiddlewareConsumer) {
    consumer
      .apply(LoggerMiddleware) // Aplica o LoggerMiddleware
      .forRoutes(UserController); // Apenas no UserController
  }
}
```

## Usos Comuns de Middlewares

- `Autenticação`: Verificar se o usuário está autenticado antes de acessar rotas protegidas.
- `Logging`: Registrar detalhes de cada requisição, como método, URL, status e tempo de resposta.
- `Manipulação de Requisições`: Modificar ou adicionar propriedades à requisição antes de ela ser processada pelo controlador.
- `Validações Globais`: Aplicar validações que devem ser feitas em todas as requisições, como checar se um cabeçalho específico está presente.

## Resumo

- `Middlewares`: interceptam requisições antes de chegarem aos controladores, permitindo executar lógica adicional.
- `Reutilizáveis e Configuráveis`: Podem ser aplicados globalmente ou em rotas específicas.
- `Diversos Usos`: São usados para tarefas como autenticação, logging, validação, entre outras.

Middlewares são uma parte fundamental do fluxo de requisições no Nest.js, permitindo que você implemente funcionalidades transversais de forma eficiente e organizada.

## Aplicar Middlewares com Nest.js e Fastify

Quando você usa o Nest.js com o `Fastify` como servidor HTTP em vez do `Express`, a maneira de aplicar middlewares é um pouco diferente devido à arquitetura do `Fastify`. O Nest.js abstrai a maior parte dessa diferença, mas existem alguns detalhes específicos que precisam ser observados.

## Passos para Aplicar Middlewares com Nest.js e Fastify

1. **Configurando o Nest.js para Usar Fastify**: Primeiro, é importante garantir que sua aplicação Nest.js está configurada para usar o Fastify como servidor HTTP.

   Se você ainda não fez isso, pode instalar o pacote necessário e configurar o bootstrap da sua aplicação:
 
   ```bash
   npm install @nestjs/platform-fastify fastify
   ```

    No arquivo main.ts, altere o bootstrap para usar o Fastify:

    ```typescript
    import { NestFactory } from '@nestjs/core';
    import { AppModule } from './app.module';
    import { FastifyAdapter } from '@nestjs/platform-fastify';

    async function bootstrap() {
      const app = await NestFactory.create(AppModule, new FastifyAdapter());
      await app.listen(3000);
    }
    bootstrap();
    ```

2. **Criando um Middleware**: Um middleware no Nest.js com `Fastify` é criado da mesma forma que com `Express`. O processo de criação de um middleware é o mesmo:

   ```typescript
    import { Injectable, NestMiddleware } from '@nestjs/common';
    import { FastifyRequest, FastifyReply } from 'fastify';

    @Injectable()
    export class LoggerMiddleware implements NestMiddleware {
      use(req: FastifyRequest, res: FastifyReply, next: () => void) {
        console.log(`Request...`);
        res.raw.on('finish', () => {
          console.log(`Request to ${req.method} ${req.url} has finished.`);
        });
        next();
      }
    }
   ```

   Neste exemplo, o middleware é adaptado para usar os tipos do Fastify (`FastifyRequest` e `FastifyReply`), mas a lógica de implementação permanece praticamente igual.

3. **Aplicando o Middleware**: A aplicação de middlewares com `Fastify` no Nest.js é feita da mesma forma que com o `Express`. Você usa o método configure do `NestModule` para aplicar o `middleware`:

   ```typescript
    import { Module, NestModule, MiddlewareConsumer } from '@nestjs/common';
    import { LoggerMiddleware } from './logger.middleware';
    import { UserController } from './user/user.controller';

    @Module({
      controllers: [UserController],
    })
    export class UserModule implements NestModule {
      configure(consumer: MiddlewareConsumer) {
        consumer
          .apply(LoggerMiddleware)
          .forRoutes(UserController); // Aplica ao UserController
      }
    }
   ```

   Neste exemplo, o middleware LoggerMiddleware é aplicado ao UserController da mesma forma que faria com o Express.

4. **Middlewares Globais**: Se você quiser aplicar um middleware globalmente (para todas as rotas), pode fazer isso diretamente no bootstrap da aplicação:

   ```typescript
    import { NestFactory } from '@nestjs/core';
    import { AppModule } from './app.module';
    import { FastifyAdapter } from '@nestjs/platform-fastify';
    import { LoggerMiddleware } from './logger.middleware';

    async function bootstrap() {
      const app = await NestFactory.create(AppModule, new FastifyAdapter());
      
      app.use(LoggerMiddleware); // Aplicando globalmente

      await app.listen(3000);
    }
    bootstrap();
   ```

   Neste exemplo, o middleware LoggerMiddleware é aplicado globalmente a todas as rotas da aplicação.

## Considerações Especiais para Fastify

- `Objetos de Requisição e Resposta`: Em Fastify, os objetos de requisição e resposta (`req` e `res`) são diferentes dos do Express. Você pode acessar o objeto de resposta bruto com res.raw e o objeto de requisição com req.raw se precisar de compatibilidade direta com o Node.js.

- `Método next()`: Diferente do Express, o Fastify não usa o conceito de `"next"` da mesma forma, mas o Nest.js ainda abstrai isso para garantir compatibilidade. No entanto, é importante estar ciente disso ao escrever middlewares.

## Resumo

- `Criação de Middleware`: O processo de criação de middlewares em Nest.js com `Fastify` é semelhante ao `Express`, com algumas adaptações para os tipos do Fastify.
Aplicação: Middlewares podem ser aplicados em módulos específicos ou globalmente, como no Express.
- `Considerações Especiais`: Ao usar Fastify, preste atenção ao uso dos objetos `req` e `res`, que são específicos do Fastify.

Essas práticas permitem que você use middlewares de forma eficaz em uma aplicação Nest.js que usa Fastify como servidor HTTP, mantendo uma arquitetura modular e organizada.