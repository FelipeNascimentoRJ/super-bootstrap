# Nest Guards

## Descrição

No Nest.js, `guards` (ou guardiões) são uma funcionalidade que permite controlar o acesso às rotas da sua aplicação. Eles funcionam como uma barreira que decide se uma requisição pode continuar para o controlador ou se deve ser bloqueada. Geralmente, guards são utilizados para implementar `autenticação e autorização`, garantindo que apenas usuários autorizados possam acessar determinados recursos.

## Características dos Guards

- `Controle de Acesso`: Guards determinam se uma requisição pode ou não prosseguir para o próximo estágio (como um controlador ou rota específica), com base em uma lógica definida.
- `Execução Antecipada`: Eles são executados antes que os manipuladores de rota (handlers) sejam chamados, oferecendo uma camada adicional de segurança.
- `Lógica Customizada`: Você pode definir sua própria lógica nos guards para checar qualquer condição que precise ser verificada antes de permitir o acesso.

## Quando Usar Guards?

Guards são usados em várias situações, como:

- `Autenticação`: Verificar se o usuário está autenticado antes de acessar uma rota protegida.
- `Autorização`: Verificar se o usuário tem as permissões necessárias para executar uma determinada ação.
- `Validação de Condições Específicas`: Garantir que certas condições sejam atendidas antes de permitir que a requisição continue.

## Exemplo de um Guard Simples

Vamos criar um guard simples que verifica se o usuário está autenticado.

1. **Criando o Guard**: Primeiro, você cria uma classe que implementa a interface `CanActivate`:

   ```typescript
    import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
    import { Observable } from 'rxjs';

    @Injectable()
    export class AuthGuard implements CanActivate {
      canActivate(
        context: ExecutionContext,
      ): boolean | Promise<boolean> | Observable<boolean> {
        const request = context.switchToHttp().getRequest();
        return this.validateRequest(request);
      }

      validateRequest(request: any): boolean {
        // Aqui, você pode implementar a lógica de autenticação
        // Por exemplo, verificar se o usuário tem um token válido
        return request.headers.authorization ? true : false;
      }
    }
   ```

2. **Aplicando o Guard**: Você pode aplicar o guard a um controlador ou a rotas específicas usando o decorador `@UseGuards()`.

No Controlador Inteiro

 ```typescript
  import { Controller, Get, UseGuards } from '@nestjs/common';
  import { AuthGuard } from './auth.guard';

  @Controller('protected')
  @UseGuards(AuthGuard) // Aplica o guard em todas as rotas deste controlador
  export class ProtectedController {
    @Get()
    findAll() {
      return "This route is protected and requires authentication";
    }
  }
 ```

Em uma Rota Específica

 ```typescript
  import { Controller, Get, UseGuards } from '@nestjs/common';
  import { AuthGuard } from './auth.guard';

  @Controller('users')
  export class UsersController {
    @Get('profile')
    @UseGuards(AuthGuard) // Aplica o guard apenas a esta rota
    getProfile() {
      return "This route requires authentication to access";
    }
  }
 ```

## Tipos de Guards

No Nest.js, todos os guards são baseados na mesma interface fundamental, `CanActivate`. No entanto, a diferença entre os tipos de guards geralmente está no contexto em que são usados e na lógica que implementam. Embora o Nest.js não defina `"tipos"` específicos de guards, podemos classificá-los com base em seus usos comuns e como eles são aplicados:

- `Guards de Autenticação`: Verificam se o usuário está autenticado.
- `Guards de Autorização`: Verificam se o usuário tem as permissões necessárias.
- `Guards Baseados em Condições`: Implementam qualquer lógica condicional personalizada.
- `Guards Globais`: Aplicam verificação a todas as rotas da aplicação.

## Exemplos de Guards

1. **Guards de Autenticação**
   - `Propósito`: Verificar se o usuário está autenticado antes de permitir o acesso a uma rota ou recurso específico.
   - `Exemplo de Uso`: Um guard que verifica se o token de autenticação (JWT) está presente e válido na requisição.
   - `Implementação`: Lógica que checa a presença e validade de um token, cookie de sessão, ou outra forma de credencial.

   ```typescript
    import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';

    @Injectable()
    export class AuthGuard implements CanActivate {
      canActivate(context: ExecutionContext): boolean {
        const request = context.switchToHttp().getRequest();
        return request.headers.authorization ? true : false;
      }
    }
   ```

2. **Guards de Autorização**

   - `Propósito`: Verificar se o usuário tem as permissões ou roles necessárias para acessar um recurso ou realizar uma ação.
   - `Exemplo de Uso`: Um guard que checa se o usuário pertence a um determinado grupo (e.g., admin) antes de acessar uma rota protegida.
   - `Implementação`: Lógica que compara os roles ou permissões do usuário com os requisitos da rota.

   ```typescript
    import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';
    import { Reflector } from '@nestjs/core';

    @Injectable()
    export class RolesGuard implements CanActivate {
      constructor(private reflector: Reflector) {}

      canActivate(context: ExecutionContext): boolean {
        const roles = this.reflector.get<string[]>('roles', context.getHandler());
        if (!roles) {
          return true;
        }
        const request = context.switchToHttp().getRequest();
        const user = request.user;
        return roles.some(role => user.roles?.includes(role));
      }
    }
   ```

   Usando com Decorador

   ```typescript
    import { SetMetadata } from '@nestjs/common';

    export const Roles = (...roles: string[]) => SetMetadata('roles', roles);
   ```

   E depois aplicando no controlador

   ```typescript
    @Roles('admin')
    @UseGuards(RolesGuard)
    @Get('admin')
    getAdminData() {
      return "This is admin data";
    }
   ```

1. **Guards Baseados em Condições**

   - `Propósito`: Implementar qualquer lógica condicional personalizada que determine se uma requisição deve continuar.
   - `Exemplo de Uso`: Verificar se uma rota só pode ser acessada durante um horário específico ou se uma função está habilitada no sistema.
   - `Implementação`: Lógica que checa uma condição específica, como horários de operação ou recursos habilitados/desabilitados.

   ```typescript
    import { Injectable, CanActivate, ExecutionContext } from '@nestjs/common';

    @Injectable()
    export class TimeGuard implements CanActivate {
      canActivate(context: ExecutionContext): boolean {
        const currentHour = new Date().getHours();
        return currentHour >= 9 && currentHour <= 17; // Apenas entre 9h e 17h
      }
    }
   ```

2. **Guards Globais**
   
   - `Propósito`: Aplicar uma verificação a todas as requisições em toda a aplicação.
   - `Exemplo de Uso`: Um guard global que aplica uma lógica de autenticação básica a todas as rotas.
   - `Implementação`: Configurado no main.ts para ser aplicado a nível global.

   ```typescript
    import { NestFactory } from '@nestjs/core';
    import { AppModule } from './app.module';
    import { AuthGuard } from './auth.guard';

    async function bootstrap() {
      const app = await NestFactory.create(AppModule);
      app.useGlobalGuards(new AuthGuard());
      await app.listen(3000);
    }
    bootstrap();
   ```

## Resumo

- `Guards`: são usados para controlar o fluxo de uma requisição, determinando se ela pode continuar ou deve ser bloqueada.
- `Autenticação e Autorização`: são os casos de uso mais comuns, mas guards podem ser usados para qualquer lógica que precise ser verificada antes de acessar uma rota.
- `Aplicação Flexível`: Guards podem ser aplicados a rotas específicas ou a controladores inteiros, oferecendo uma maneira modular de proteger sua aplicação.

Guards são uma parte essencial do Nest.js, especialmente em aplicações onde a segurança e o controle de acesso são críticos. Eles ajudam a garantir que apenas usuários autenticados e autorizados possam acessar determinados recursos, tornando sua aplicação mais segura e robusta.

Cada guard é uma ferramenta poderosa para garantir que apenas usuários autorizados e em condições específicas possam acessar os recursos da sua aplicação, proporcionando uma camada adicional de segurança e controle.