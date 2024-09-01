# Nest Pipes

## Descrição

Pipes no Nest.js são uma ferramenta usada para transformar e validar os dados que entram em sua aplicação. Eles são aplicados aos parâmetros da rota antes que o controlador os receba. Pipes podem ser usados para:

1. **Transformação**: Alterar os dados de entrada para um formato ou tipo específico antes que sejam processados pelo controlador. Por exemplo, converter strings em números.

2. **Validação**: Garantir que os dados de entrada atendem a certos critérios ou regras. Se os dados não forem válidos, o pipe pode lançar uma exceção, impedindo a execução do controlador.

## Principais Usos dos Pipes

1. **Transformar Tipos de Dados**: Convertendo strings de parâmetros de rota para números ou datas, por exemplo.
2. **Validar Dados de Entrada**: Garantir que o corpo de uma requisição contém os dados necessários e que estão no formato correto.
3. **Sanitização**: Remover ou modificar campos indesejados nos dados de entrada.

## Tipos de Pipes

1. **Built-in Pipes**: O Nest.js vem com alguns pipes embutidos, como:

  - `ValidationPipe`: Valida automaticamente os objetos baseados em DTOs.
  - `ParseIntPipe`: Converte um parâmetro de string para inteiro.
  - `ParseBoolPipe`: Converte um parâmetro de string para booleano.

2. **Custom Pipes**: Você pode criar seus próprios pipes para necessidades específicas de validação ou transformação.

## Exemplo de Uso de Pipes

1. **Usando o `ValidationPipe` para Validar Dados**

   ```typescript
    import { Controller, Post, Body } from '@nestjs/common';
    import { IsString, IsInt, MinLength } from 'class-validator';
    import { ValidationPipe } from '@nestjs/common';

    class CreateUserDto {
      @IsString()
      @MinLength(3)
      name: string;

      @IsInt()
      age: number;
    }

    @Controller('users')
    export class UsersController {
      @Post()
      create(@Body(new ValidationPipe()) createUserDto: CreateUserDto) {
        return `User ${createUserDto.name} with age ${createUserDto.age} created!`;
      }
    }
   ```

   Neste exemplo, o `ValidationPipe` garante que os dados enviados no corpo da requisição atendem às regras definidas no `CreateUserDto`. Se os dados forem inválidos, o pipe lançará uma exceção e retornará uma resposta de erro ao cliente.

2. **Criando um Pipe Customizado**

   ```typescript
    import { Injectable, PipeTransform, ArgumentMetadata, BadRequestException } from '@nestjs/common';

    @Injectable()
    export class ParseIntPipe implements PipeTransform<string, number> {
      transform(value: string, metadata: ArgumentMetadata): number {
        const val = parseInt(value, 10);
        if (isNaN(val)) {
          throw new BadRequestException('Validation failed: Invalid number');
        }
        return val;
      }
    }
   ```

   Neste exemplo, criamos um pipe customizado `ParseIntPipe` que converte um parâmetro de string para um número inteiro. Se a conversão falhar, o pipe lança uma exceção `BadRequestException`.

## Aplicando Pipes

1. **A uma Rota Específica**: Você pode aplicar um pipe a uma rota específica usando o construtor do pipe como argumento do decorador `@Param`, `@Body`, ou `@Query`:

   ```typescript
    import { Controller, Get, Param } from '@nestjs/common';
    import { ParseIntPipe } from './parse-int.pipe';

    @Controller('users')
    export class UsersController {
      @Get(':id')
      findOne(@Param('id', new ParseIntPipe()) id: number) {
        return `This action returns a user with id ${id}`;
      }
    }
   ```

2. **Globalmente**: Você pode aplicar pipes globalmente no main.ts:

   ```typescript
    import { ValidationPipe } from '@nestjs/common';

    async function bootstrap() {
      const app = await NestFactory.create(AppModule);
      app.useGlobalPipes(new ValidationPipe());
      await app.listen(3000);
    }
    bootstrap();
   ```

   Neste exemplo, o `ValidationPipe` é aplicado globalmente a todas as rotas da aplicação.

## Resumo

`Pipes` são essenciais para transformar e validar os dados de entrada em sua aplicação Nest.js. Eles ajudam a garantir que os dados recebidos pelo controlador estão no formato correto e atendem às regras necessárias, aumentando a segurança e a robustez da aplicação. Eles podem ser aplicados tanto a nível de rota quanto globalmente, oferecendo flexibilidade e controle.