# Nest Interceptors

## Descrição

Interceptors no Nest.js são uma ferramenta poderosa que permite transformar os dados de requisições e respostas, gerenciar exceções e até mesmo modificar o comportamento da lógica de execução. Eles funcionam de maneira semelhante a middlewares, mas com um foco mais específico em manipular as respostas e os fluxos de dados.

## Principais Usos dos Interceptors

- `Transformação de Respostas`: Modificar o formato da resposta enviada ao cliente. Por exemplo, padronizar o formato JSON das respostas.

- `Manipulação de Exceções`: Capturar e modificar exceções antes que elas sejam enviadas ao cliente, permitindo uma personalização do tratamento de erros.

- `Adição de Lógica antes/depois da Execução de Métodos`: Inserir lógica adicional que deve ser executada antes ou depois de um método do controlador ser chamado.

- `Criação de Lógicas Condicionais`: Alterar a execução com base em certas condições, como medir o tempo de execução de uma rota ou adicionar cabeçalhos personalizados às respostas.

## Exemplo de Uso

Vamos criar um interceptor que mede o tempo de execução de uma rota:

```typescript
import {
  Injectable,
  NestInterceptor,
  ExecutionContext,
  CallHandler,
} from '@nestjs/common';
import { Observable } from 'rxjs';
import { tap } from 'rxjs/operators';

@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler): Observable<any> {
    console.log('Before...');
    const now = Date.now();

    return next
      .handle()
      .pipe(
        tap(() => console.log(`After... ${Date.now() - now}ms`)),
      );
  }
}
```

## Aplicando o Interceptor

Você pode aplicar interceptors em controladores inteiros, rotas específicas, ou globalmente.

- Em uma rota específica:

```typescript
import { Controller, Get, UseInterceptors } from '@nestjs/common';
import { LoggingInterceptor } from './logging.interceptor';

@Controller('users')
export class UsersController {
  @Get()
  @UseInterceptors(LoggingInterceptor)
  findAll() {
    return "This action returns all users";
  }
}
```

- Globalmente:

No `main.ts`, ao inicializar a aplicação:

```typescript
import { NestFactory } from '@nestjs/core';
import { AppModule } from './app.module';
import { LoggingInterceptor } from './logging.interceptor';

async function bootstrap() {
  const app = await NestFactory.create(AppModule);
  app.useGlobalInterceptors(new LoggingInterceptor());
  await app.listen(3000);
}
bootstrap();
```

## Resumo

Interceptors são uma ferramenta poderosa para manipular e transformar as respostas e fluxos de dados em sua aplicação Nest.js. Eles oferecem flexibilidade para realizar operações antes e depois da execução de métodos, além de permitir a customização de como as exceções são tratadas e as respostas são enviadas.