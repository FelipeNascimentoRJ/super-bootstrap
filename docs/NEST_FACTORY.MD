# Nest useFactory

## Descrição

O `useFactory` é um método de configuração de providers no Nest.js que permite a criação de instâncias de classes ou funções de forma personalizada. Ele é útil quando você precisa de mais controle sobre a criação de um provider, como a definição de parâmetros ou a execução de lógica adicional.

## Por Que Usar `useFactory`?

O `useFactory` é útil quando você precisa de uma instância de um provider que não pode ser criada diretamente através de uma classe ou função. Por exemplo, você pode querer inicializar um provider com base em variáveis de ambiente, ou configurar um provider de acordo com o ambiente de execução.

- `Configuração .env`: O `useFactory` permite a leitura de variáveis de ambiente para configurar um provider de forma dinâmica.
- `Configuração Dinâmica`: O `useFactory` permite a configuração dinâmica de providers com base em parâmetros ou lógica personalizada.
- `Reutilização de Lógica`: O `useFactory` permite a reutilização de lógica de inicialização complexa em diferentes partes da aplicação.

## Como Funciona useFactory?

Quando você define um provider com `useFactory`, em vez de fornecer uma classe que será instanciada, você fornece uma função que o Nest.js chamará para criar a dependência. Essa função pode receber outros providers como parâmetros, e seu retorno será o valor injetado em quem solicitar esse provider.

## Exemplo de Uso de `useFactory`

Vamos considerar um exemplo onde precisamos configurar um serviço de email. O serviço de email precisa ser configurado com base em variáveis de ambiente, como as credenciais do servidor SMTP.

## Configuração com `useFactory`

```typescript
import { Module, Injectable } from '@nestjs/common';
import { ConfigService } from '@nestjs/config';

@Injectable()
export class EmailService {
  constructor(private smtpHost: string, private smtpPort: number) {}

  sendEmail(to: string, subject: string, body: string) {
    // Lógica para enviar o email usando o servidor SMTP configurado
  }
}

@Module({
  providers: [
    {
      provide: EmailService,
      useFactory: (configService: ConfigService) => {
        const smtpHost = configService.get<string>('SMTP_HOST');
        const smtpPort = configService.get<number>('SMTP_PORT');
        return new EmailService(smtpHost, smtpPort);
      },
      inject: [ConfigService], // Especifica que ConfigService será injetado na fábrica
    },
  ],
  exports: [EmailService],
})
export class EmailModule {}
```

## Explicação do Exemplo

- `provide: EmailService`: Define que este provider está associado ao `EmailService`. Quando o `EmailService` for solicitado em outro lugar da aplicação, a instância retornada será aquela criada pela função useFactory.

- `useFactory`: A função de fábrica que cria e retorna uma instância do `EmailService`. Ela utiliza o `ConfigService` para buscar as configurações de ambiente necessárias (`SMTP_HOST` e `SMTP_PORT`).

- `inject`: Especifica outros providers que devem ser injetados como dependências na função de fábrica. Neste caso, o `ConfigService` é injetado para que a função possa acessar as configurações.

## Benefícios do `useFactory`

- `Flexibilidade`: Permite lógica complexa na criação de dependências, como configuração condicional ou uso de dados dinâmicos.

- `Reuso`: Pode reutilizar dependências existentes no processo de criação, aproveitando a injeção de dependência.

- `Modularidade`: Facilita a separação da lógica de criação de instâncias, tornando o código mais modular e testável.

## Resumo

- `useFactory` é uma forma de criar providers dinamicamente, utilizando uma função de fábrica.
- É usado quando a criação de uma dependência requer lógica adicional, como leitura de configurações ou uso de outros providers.
- A função de fábrica pode receber outros providers como dependências e deve retornar o valor ou a instância a ser injetada.

Essa abordagem é poderosa para cenários onde a configuração e inicialização de serviços não é trivial e depende de condições variáveis.