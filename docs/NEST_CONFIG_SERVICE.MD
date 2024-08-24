# Nest ConfigService

## Descrição

O `ConfigService` no Nest.js é uma ferramenta poderosa para gerenciar configurações de forma centralizada e organizada. Ele permite que você acesse variáveis de ambiente e outras configurações em diferentes partes da sua aplicação de maneira segura e consistente.

## Por Que Usar o ConfigService?

O `ConfigService` é útil quando você precisa acessar configurações específicas da aplicação, como credenciais de banco de dados, chaves de API, URLs de serviços externos, ou qualquer outra informação sensível ou variável. Ele fornece uma maneira segura e padronizada de acessar essas configurações em toda a sua aplicação.

## Passos para Usar o ConfigService

Para usar o `ConfigService` no Nest.js, siga estes passos:

1. **Instalação do Pacote**: Certifique-se de que o pacote `@nestjs/config` está instalado na sua aplicação. Você pode instalá-lo via npm ou yarn:

   ```bash
   npm install --save @nestjs/config
   ```

   ou

   ```bash
   yarn add @nestjs/config
   ```

2. **Configurar o Módulo de Configuração**: 

   Depois de instalar o pacote, você precisa configurar o módulo de configuração no `AppModule` ou em qualquer outro módulo central da sua aplicação:

   ```typescript
   import { Module } from '@nestjs/common';
   import { ConfigModule } from '@nestjs/config';

   @Module({
     imports: [
       ConfigModule.forRoot({
         isGlobal: true, // Torna o ConfigService disponível globalmente, sem necessidade de importação em cada módulo
       }),
       // Outros módulos aqui
     ],
   })
   export class AppModule {}
   ```

3. **Criar um Arquivo .env**: O ConfigService geralmente trabalha com variáveis de ambiente, que podem ser definidas em um arquivo `.env` na raiz do seu projeto. Por exemplo:

    ```plaintext
    DATABASE_HOST=localhost
    DATABASE_PORT=5432
    SMTP_HOST=smtp.example.com
    SMTP_PORT=587
    ```

4. **Acessar Configurações Usando o ConfigService**: Agora, você pode injetar o ConfigService em qualquer provedor ou controlador onde precise acessar essas configurações. Vamos usar um exemplo com um serviço que acessa configurações do banco de dados:

    ```typescript
    import { Injectable } from '@nestjs/common';
    import { ConfigService } from '@nestjs/config';

    @Injectable()
    export class DatabaseService {
      private host: string;
      private port: number;

      constructor(private configService: ConfigService) {
        this.host = this.configService.get<string>('DATABASE_HOST');
        this.port = this.configService.get<number>('DATABASE_PORT');
      }

      connect() {
        console.log(`Connecting to database at ${this.host}:${this.port}`);
        // Lógica para conectar ao banco de dados
      }
    }
    ```

5. **Injetar o ConfigService em Outros Providers**: Você também pode usar o ConfigService em uma função de fábrica para configurar outros serviços. Aqui está um exemplo utilizando o useFactory que vimos anteriormente:

   ```typescript
   import { Module } from '@nestjs/common';
   import { ConfigService } from '@nestjs/config';
   import { EmailService } from './email.service';

   @Module({
     providers: [
       {
         provide: EmailService,
         useFactory: (configService: ConfigService) => {
           const smtpHost = configService.get<string>('SMTP_HOST');
           const smtpPort = configService.get<number>('SMTP_PORT');
           return new EmailService(smtpHost, smtpPort);
         },
         inject: [ConfigService],
       },
     ],
     exports: [EmailService],
   })
   export class EmailModule {}
   ```

## Resumo do Uso do ConfigService

- `Configurações Centralizadas`: O ConfigService permite acessar variáveis de ambiente e outras configurações de forma centralizada.
- `Configuração do Módulo`: O ConfigModule.forRoot() inicializa o ConfigService, permitindo o acesso às configurações em toda a aplicação.
- `Injeção de Dependência`: O ConfigService pode ser injetado em qualquer provedor ou controlador para acessar configurações específicas.

Usar o `ConfigService` é uma prática recomendada no Nest.js para gerenciar variáveis de ambiente e configurações, garantindo que sua aplicação seja fácil de configurar e manter em diferentes ambientes (desenvolvimento, produção, etc.).