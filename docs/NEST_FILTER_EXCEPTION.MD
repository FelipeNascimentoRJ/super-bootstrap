# Nest Filtros de Exceção

## Description

Filtros de exceção no Nest.js são mecanismos que permitem capturar e manipular exceções lançadas em sua aplicação. Eles oferecem uma maneira centralizada de lidar com erros, permitindo que você personalize as respostas enviadas ao cliente quando ocorre uma exceção.

## Principais Usos dos Filtros de Exceção

1. **Manipulação Global de Erros**: Tratar de maneira centralizada todos os erros que acontecem na aplicação.

2. **Personalização de Respostas de Erro**: Modificar o formato das respostas de erro para atender a padrões específicos, como JSON ou XML.

3. **Logging de Erros**: Registrar detalhes das exceções para fins de auditoria ou análise.

## Tipos de Filtros de Exceção

1. **Filtros de Exceção de Escopo Global**: Aplicados a toda a aplicação, eles capturam todas as exceções não tratadas.

2. **Filtros de Exceção de Escopo de Controlador**: Aplicados a um controlador específico, eles capturam exceções lançadas por métodos desse controlador.

## Exemplo de Uso de Filtros de Exceção

1. **Criando um Filtro de Exceção Customizado**: Aqui está um exemplo de um filtro de exceção que captura todas as exceções do tipo `HttpException` e personaliza a resposta:

   ```typescript
    import { ExceptionFilter, Catch, ArgumentsHost, HttpException } from '@nestjs/common';
    import { Request, Response } from 'express';

    @Catch(HttpException)
    export class HttpExceptionFilter implements ExceptionFilter {
      catch(exception: HttpException, host: ArgumentsHost) {
        const ctx = host.switchToHttp();
        const response = ctx.getResponse<Response>();
        const request = ctx.getRequest<Request>();
        const status = exception.getStatus();

        response
          .status(status)
          .json({
            statusCode: status,
            timestamp: new Date().toISOString(),
            path: request.url,
            message: exception.message,
          });
      }
    }
   ```

   Neste exemplo, o filtro captura exceções do tipo `HttpException` e retorna uma resposta JSON padronizada contendo o código de status, um timestamp, o caminho da URL que causou a exceção, e a mensagem de erro.

2. **Aplicando o Filtro de Exceção**

  - `Em um Controlador Específico`: Você pode aplicar o filtro de exceção a um controlador específico usando o decorador `@UseFilters`:

  ```typescript
    import { Controller, Get, UseFilters } from '@nestjs/common';
    import { HttpExceptionFilter } from './http-exception.filter';

    @Controller('cats')
    @UseFilters(HttpExceptionFilter)
    export class CatsController {
      @Get()
      findAll() {
        throw new HttpException('Not Found', 404);
      }
    }
  ```

  - `Globalmente`: Para aplicar o filtro globalmente, você pode registrá-lo no escopo do módulo raiz da aplicação no arquivo `main.ts`:

  ```typescript
    import { NestFactory } from '@nestjs/core';
    import { AppModule } from './app.module';
    import { HttpExceptionFilter } from './http-exception.filter';

    async function bootstrap() {
      const app = await NestFactory.create(AppModule);
      app.useGlobalFilters(new HttpExceptionFilter());
      await app.listen(3000);
    }
    bootstrap();
  ```

## Personalizando Respostas de Erro

Filtros de exceção permitem total controle sobre o formato das respostas de erro. Você pode adicionar informações adicionais, modificar mensagens ou até mesmo redirecionar para outras páginas, dependendo da lógica da aplicação.

## Resumo

Filtros de Exceção são uma ferramenta poderosa no Nest.js para gerenciar e personalizar a maneira como sua aplicação lida com erros. Eles fornecem uma abordagem centralizada para o tratamento de exceções, garantindo que as respostas de erro sejam consistentes e informativas. Seja aplicando filtros globalmente ou a controladores específicos, eles são essenciais para uma gestão robusta de erros na aplicação.