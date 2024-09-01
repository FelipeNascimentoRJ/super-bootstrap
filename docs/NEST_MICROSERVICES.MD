## Nest Microsserviços

## Descrição

Microsserviços no Nest.js referem-se a uma arquitetura onde a aplicação é dividida em múltiplos serviços menores, cada um focado em uma funcionalidade específica, e que podem ser desenvolvidos, implantados e escalados de forma independente. Essa abordagem é especialmente útil para aplicações grandes e complexas, onde a modularidade e a independência de componentes são cruciais.

Nest.js oferece suporte robusto para construir aplicações baseadas em microsserviços, permitindo a comunicação entre eles de forma simples e eficaz, utilizando diferentes padrões de comunicação como mensagens, eventos e RPC (Remote Procedure Call).

## Principais Usos dos Microsserviços

1. **Escalabilidade Independente**: Cada microsserviço pode ser escalado independentemente dos outros, permitindo uma alocação mais eficiente de recursos.

2. **Desenvolvimento Modular**: Diferentes equipes podem trabalhar em diferentes microsserviços simultaneamente, acelerando o desenvolvimento e a entrega de novas funcionalidades.

3. **Resiliência e Tolerância a Falhas**: A falha em um microsserviço não afeta os outros, tornando a aplicação mais resiliente e menos propensa a falhas catastróficas.

4. **Facilidade de Manutenção**: Cada microsserviço é um componente isolado, facilitando a manutenção, atualização e correção de bugs sem afetar o restante da aplicação.

5. **Flexibilidade Tecnológica**: Cada microsserviço pode ser desenvolvido em uma linguagem ou tecnologia diferente, permitindo a escolha da melhor ferramenta para cada tarefa.

## Comunicação Entre Microsserviços

Nest.js suporta vários mecanismos de comunicação entre microsserviços:

1. **TCP**: Comunicação baseada em sockets TCP, que é adequada para comunicação síncrona e de baixa latência.
2. **gRPC**: Comunicação síncrona e escalável usando o protocolo gRPC.
3. **Mensageria (Ex: RabbitMQ, Kafka)**: Comunicação assíncrona e escalável usando sistemas de mensageria como RabbitMQ ou Apache Kafka.
4. **Redis**: Comunicação assíncrona e de alta performance usando o Redis como mecanismo de mensageria.

## Exemplo de Uso de Microsserviços com `Nest.js` e `RabbitMQ`

Vamos criar um exemplo completo de `microserviços` usando `NestJS` com `RabbitMQ` como meio de transporte. Este exemplo consistirá em duas aplicações NestJS separadas:

1. **Microserviço (Backend)**: Um serviço que realiza operações matemáticas, como soma e subtração.

2. **Cliente (Gateway)**: Uma aplicação que consome o microserviço e expõe endpoints HTTP para o cliente final.

### Visão Geral

- `Microserviço`: Responde a mensagens de operações matemáticas via RabbitMQ.
- `Cliente`: Envia solicitações ao microserviço através de RabbitMQ e expõe endpoints HTTP para interagir com o cliente.

### Pré-requisitos

- `Node.js`instalado (preferencialmente a versão LTS).
- `Docker` e `Docker Compose` instalados.
- `Nest CLI` instalado globalmente. ```npm install -g @nestjs/cli```


1. **Configurar o RabbitMQ**: Vamos utilizar o Docker para facilitar a configuração do RabbitMQ.

   - `Iniciar o RabbitMQ com Docker`: Abra o terminal e execute o seguinte comando:

    <br />

    ```bach
    docker run -d --hostname my-rabbit --name some-rabbit -p 5672:5672 -p 15672:15672 rabbitmq:3-management
    ```
   - `Porta 5672`: Porta padrão para comunicação AMQP.
   - `Porta 15672`: Porta para acessar o painel de gerenciamento do RabbitMQ.
   
   <br />
   
   **Acessar o Painel de Gerenciamento**: Abra o navegador e vá para http://localhost:15672, faça login com as seguintes credenciais:

   - `Usuário`: guest
   - `Senha`: guest

   <br />

2. **Criar o Microserviço**: Vamos criar o microserviço que realiza operações matemáticas.

   <br />

   **Passo 2.1: Criar o Projeto NestJS para o Microserviço**

    No terminal, execute o seguinte comando para criar um novo projeto NestJS chamado `math-microservice`:
  
    ```bash
    nest new math-microservice
    ```

    Siga as instruções e escolha o gerenciador de pacotes de sua preferência (`npm` ou `yarn`).

   <br />

   **Passo 2.2: Instalar Dependências Necessárias**

   Navegue até o diretório do microserviço e instale as dependências do microserviço e do RabbitMQ:

   ```bash
    cd math-microservice
    npm install --save @nestjs/microservices amqplib
   ```
   <br />

   **Passo 2.3: Configurar o Microserviço**

    - `2.3.1`: Substitua o conteúdo de `src/main.ts` pelo seguinte código:

     ```typescript
      import { NestFactory } from '@nestjs/core';
      import { AppModule } from './app.module';
      import { MicroserviceOptions, Transport } from '@nestjs/microservices';

      async function bootstrap() {
        const app = await NestFactory.createMicroservice<MicroserviceOptions>(
          AppModule,
          {
            transport: Transport.RMQ,
            options: {
              urls: ['amqp://guest:guest@localhost:5672'],
              queue: 'math_queue',
              queueOptions: {
                durable: false
              },
            },
          },
        );
        await app.listen();
        console.log('Math Microservice is listening...');
      }
      bootstrap();
     ```

    - `2.3.2`: Criar o Controller para as Operações Matemáticas

      Crie um controller para lidar com as operações matemáticas.

      ```bash
      nest generate controller math
      ```

      Substitua o conteúdo de `src/math/math.controller.ts` pelo seguinte código:

      ```typescript
      import { Controller } from '@nestjs/common';
      import { MessagePattern, Payload } from '@nestjs/microservices';

      @Controller()
      export class MathController {
        @MessagePattern({ cmd: 'sum' })
        sum(@Payload() data: number[]): number {
          return (data || []).reduce((a, b) => a + b, 0);
        }

        @MessagePattern({ cmd: 'subtract' })
        subtract(@Payload() data: number[]): number {
          return data[0] - data[1];
        }
      }
      ```

    - `2.3.3`: Atualizar o Módulo Principal

      Atualize o módulo principal `src/app.module.ts` para importar o `MathController` e configurar o microserviço.

      ```typescript
      import { Module } from '@nestjs/common';
      import { MathController } from './math/math.controller';

      @Module({
        imports: [],
        controllers: [MathController],
        providers: [],
      })
      export class AppModule {}
      ```

   **Passo 2.4: Executar o Microserviço**

     No terminal, dentro do diretório `math-microservice`, execute:

     ```bash
     npm run start
     ```

     Você deve ver a mensagem: `Math Microservice is listening...`

    <br />

3. **Criar o Cliente (Gateway)**: Vamos criar o cliente que consome o microserviço e expõe endpoints HTTP.

   **Passo 3.1: Criar o Projeto NestJS para o Cliente**

     No terminal, execute o seguinte comando para criar um novo projeto NestJS chamado `math-client`:

     ```bash
     nest new math-client
     ```

   **Passo 3.2: Instalar Dependências Necessárias**

      Navegue até o diretório do cliente e instale as dependências do cliente e do RabbitMQ:
  
      ```bash
      cd math-client
      npm install --save @nestjs/microservices amqplib
      ```

   **Passo 3.3: Configurar o Cliente**

    - `3.3.1`: Substitua o conteúdo de `src/main.ts` pelo seguinte código:

      ```typescript
      import { NestFactory } from '@nestjs/core';
      import { AppModule } from './app.module';
      import { Transport, ClientProxyFactory, ClientProxy } from '@nestjs/microservices';

      async function bootstrap() {
        const app = await NestFactory.create(AppModule);
        await app.listen(3000);
        console.log('Math Client is listening on port 3000');
      }
      bootstrap();
      ```

    - `3.3.2`: Criar um Serviço para Comunicar com o Microserviço

      Crie um serviço para se comunicar com o microserviço através de RabbitMQ.

      ```bash
      nest generate service math
      ```

      Substitua o conteúdo de `src/math/math.service.ts` pelo seguinte código:

      ```typescript
      import { Injectable, OnModuleInit } from '@nestjs/common';
      import { ClientProxy, ClientProxyFactory, Transport } from '@nestjs/microservices';

      @Injectable()
      export class MathService implements OnModuleInit {
        private client: ClientProxy;

        onModuleInit() {
          this.client = ClientProxyFactory.create({
            transport: Transport.RMQ,
            options: {
              urls: ['amqp://guest:guest@localhost:5672'],
              queue: 'math_queue',
              queueOptions: {
                durable: false
              },
            },
          });
        }

        sum(numbers: number[]): Promise<number> {
          return this.client.send<number>({ cmd: 'sum' }, numbers).toPromise();
        }

        subtract(numbers: number[]): Promise<number> {
          return this.client.send<number>({ cmd: 'subtract' }, numbers).toPromise();
        }
      }
      ```

    - `3.3.3`: Atualizar o Módulo Principal

      Atualize o módulo principal `src/app.module.ts` para importar o `MathService`.

      ```typescript
      import { Module } from '@nestjs/common';
      import { AppController } from './app.controller';
      import { AppService } from './app.service';
      import { MathService } from './math/math.service';

      @Module({
        imports: [],
        controllers: [AppController],
        providers: [AppService, MathService],
      })
      export class AppModule {}
      ```

    - `3.3.4`: Criar um Controller para Expor Endpoints HTTP

      Crie um controller para expor endpoints HTTP para o cliente.

      ```bash
      nest generate controller app
      ```

      Substitua o conteúdo de `src/app.controller.ts` pelo seguinte código:

      ```typescript
      import { Controller, Get, Query } from '@nestjs/common';
      import { MathService } from './math/math.service';

      @Controller('math')
      export class AppController {
        constructor(private readonly mathService: MathService) {}

        @Get('sum')
        async sum(@Query('a') a: string, @Query('b') b: string): Promise<number> {
          const numA = parseInt(a, 10);
          const numB = parseInt(b, 10);
          return this.mathService.sum([numA, numB]);
        }

        @Get('subtract')
        async subtract(@Query('a') a: string, @Query('b') b: string): Promise<number> {
          const numA = parseInt(a, 10);
          const numB = parseInt(b, 10);
          return this.mathService.subtract([numA, numB]);
        }
      }
      ```

      Este controller expõe dois endpoints HTTP: `/math/sum` e `/math/subtract` que chamam os métodos `sum` e `subtract` do `MathService`.

      - `GET /math/sum?a=2&b=3`: Retorna a soma de 2 e 3.
      - `GET /math/subtract?a=5&b=2`: Retorna a subtração de 5 e 2.

  **Passo 3.4: Executar o Cliente**

    No terminal, dentro do diretório `math-client`, execute:

    ```bash
    npm run start
    ```

    Você deve ver a mensagem: `Math Client is listening on port 3000`

4. **Testar a Comunicação Entre os Microsserviços**

   - **Testar a Operação de Soma**: Abra o navegador e vá para http://localhost:3000/math/sum?a=2&b=3. Você deve ver o resultado `5`.

   - **Testar a Operação de Subtração**: Abra o navegador e vá para http://localhost:3000/math/subtract?a=5&b=2. Você deve ver o resultado `3`.

   - **Verificar o RabbitMQ**: Abra o painel de gerenciamento do RabbitMQ (http://localhost:15672) e verifique as mensagens na fila `math_queue`.

   - **Escalabilidade**: Você pode escalar o microserviço e o cliente separadamente para lidar com mais solicitações.

   - **Resiliência**: Se o microserviço falhar, o cliente ainda pode funcionar normalmente, garantindo a resiliência da aplicação.

   - **Flexibilidade**: Você pode adicionar mais microsserviços ou clientes conforme necessário, mantendo a flexibilidade da arquitetura.

5. **Verificar Logs**

   - `Microserviço`: No terminal do microserviço (`math-microservice`), você pode adicionar logs para verificar se as mensagens estão sendo recebidas e processadas.

    Por exemplo, atualize `math.controller.ts` para adicionar logs:

    ```typescript
    import { Controller, Logger } from '@nestjs/common';
    import { MessagePattern, Payload } from '@nestjs/microservices';

    @Controller()
    export class MathController {
      private readonly logger = new Logger(MathController.name);

      @MessagePattern({ cmd: 'sum' })
      sum(@Payload() data: number[]): number {
        this.logger.log(`Sum called with data: ${data}`);
        return (data || []).reduce((a, b) => a + b, 0);
      }

      @MessagePattern({ cmd: 'subtract' })
      subtract(@Payload() data: number[]): number {
        this.logger.log(`Subtract called with data: ${data}`);
        return data[0] - data[1];
      }
    }
    ```

    Agora, ao fazer requisições, você verá logs como:

    ```bash
    [MathController] Sum called with data: [ 10, 5 ]
    [MathController] Subtract called with data: [ 10, 5 ]
    ```

    - `Cliente`: No terminal do cliente (`math-client`), você pode adicionar logs para verificar se as mensagens estão sendo enviadas e recebidas corretamente.

    Por exemplo, atualize `math.service.ts` para adicionar logs:

    ```typescript
    import { Injectable, OnModuleInit, Logger } from '@nestjs/common';
    import { ClientProxy, ClientProxyFactory, Transport } from '@nestjs/microservices';

    @Injectable()
    export class MathService implements OnModuleInit {
      private client: ClientProxy;
      private readonly logger = new Logger(MathService.name);

      onModuleInit() {
        this.client = ClientProxyFactory.create({
          transport: Transport.RMQ,
          options: {
            urls: ['amqp://guest:guest@localhost:5672'],
            queue: 'math_queue',
            queueOptions: {
              durable: false
            },
          },
        });
      }

      sum(numbers: number[]): Promise<number> {
        this.logger.log(`Sending sum request with data: ${numbers}`);
        return this.client.send<number>({ cmd: 'sum' }, numbers).toPromise();
      }

      subtract(numbers: number[]): Promise<number> {
        this.logger.log(`Sending subtract request with data: ${numbers}`);
        return this.client.send<number>({ cmd: 'subtract' }, numbers).toPromise();
      }
    }
    ```

6. **Considerações Finais**

    - **Segurança**: Certifique-se de adicionar medidas de segurança, como autenticação e autorização, para proteger a comunicação entre os microsserviços.
  
    - **Monitoramento**: Use ferramentas de monitoramento e rastreamento para acompanhar o desempenho e a integridade dos microsserviços.
  
    - **Documentação**: Mantenha a documentação atualizada para facilitar o desenvolvimento e a integração de novos microsserviços.
  
    - **Testes**: Realize testes automatizados para garantir a qualidade e a confiabilidade dos microsserviços.
  
    - **Escalabilidade**: Projete os microsserviços para serem escaláveis horizontalmente, permitindo lidar com um aumento na carga de trabalho.
  
    - **Padrões de Projeto**: Use padrões de projeto como Circuit Breaker, Retry e Timeout para tornar os microsserviços mais robustos e resilientes.
  
    - **Monitoramento de Erros**: Implemente um sistema de monitoramento de erros para identificar e corrigir problemas rapidamente.
  
    - **Gerenciamento de Configuração**: Use ferramentas de gerenciamento de configuração para manter as configurações dos microsserviços consistentes e atualizadas.

    - **Observabilidade**: Adicione recursos de observabilidade, como logs, métricas e rastreamento, para facilitar a depuração e o monitoramento dos microsserviços.
  
    - **Padrões de Comunicação**: Use padrões de comunicação como mensagens, eventos e RPC de forma consistente para garantir uma comunicação eficaz entre os microsserviços.

## Referências Úteis

- [Documentação Oficial do Nest.js](https://docs.nestjs.com/)
- [Documentação do RabbitMQ](https://www.rabbitmq.com/documentation.html)
- [Documentação do Docker](https://docs.docker.com/)
- [Documentação do AMQP](https://www.amqp.org/)

Com essa base, você pode expandir a arquitetura de microserviços conforme as necessidades do seu projeto, aproveitando os recursos poderosos que o NestJS oferece para construir sistemas escaláveis e robustos.