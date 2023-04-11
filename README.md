# nestjs-supabase-auth

## Installation

### Install peer dependencies

Using npm:
```
npm install passport passport-jwt @nestjs/passport
npm install --save-dev @types/passport-jwt
```

Using yarn:
```
yarn add passport passport-jwt @nestjs/passport
yarn add -D @types/passport-jwt
```

### Install strategy

Using npm:
```
npm install nestjs-supabase-auth
```

Using yarn:
```
yarn add nestjs-supabase-auth
```

## Example

### Extends the strategy to create your own strategy

In this example, I'm passing supabase related options through dotenv and [env-cmd](https://github.com/toddbluhm/env-cmd) package. 

```ts
import { Injectable } from "@nestjs/common";
import { PassportStrategy } from "@nestjs/passport";
import { ExtractJwt } from "passport-jwt";
import { SupabaseAuthStrategy, SupabaseAuthUser } from "@h-sw/nestjs-supabase-auth";

@Injectable()
export class SupabaseStrategy extends PassportStrategy(
  SupabaseAuthStrategy,
  "supabase",
) {
  public constructor () {
    console.log(ExtractJwt.fromAuthHeaderAsBearerToken())
    super({
      supabaseUrl: process.env.SUPABASE_URL,
      supabaseKey: process.env.SUPABASE_KEY,
      supabaseOptions: {},
      supabaseJwtSecret: process.env.SUPABASE_JWT_SECRET,
      extractor: ExtractJwt.fromAuthHeaderAsBearerToken(),
    });
  }

  async validate (user: SupabaseAuthUser) {
    return super.validate(user);
  }

  authenticate (req: never) {
    return super.authenticate(req);
  }
}
```

### Add the strategy to your auth module

```ts
import { Module } from '@nestjs/common';
import { AuthService } from './auth.service';
import { AuthResolver } from './auth.resolver';
import { SupabaseStrategy } from './supabase.strategy';
import { PassportModule } from '@nestjs/passport';
import supabase from '../../supabase';

@Module({
  imports: [PassportModule],
  providers: [
    AuthService,
    AuthResolver,
    SupabaseStrategy,
  ],
  exports: [AuthService, SupabaseStrategy],
})
export class AuthModule {}
```
### Protect your routes

#### Example for Graphql

supabase.guard.ts
```ts
import { Injectable } from '@nestjs/common';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class SupabaseGuard extends AuthGuard('supabase') {}
```

### CurrentUser decorator

```ts
import { createParamDecorator, ExecutionContext } from "@nestjs/common";
import { SupabaseAuthUser } from "@h-sw/nestjs-supabase-auth";

export type SupabaseAuthRequest = Partial<Request> & { user?: SupabaseAuthUser };

export const User = createParamDecorator(
  (_data: unknown, context: ExecutionContext) => {
    const request = context.switchToHttp().getRequest<SupabaseAuthRequest>();

    return request.user;
  },
);

export const UserId = createParamDecorator(
  (_data: unknown, context: ExecutionContext) => {
    const request = context.switchToHttp().getRequest<SupabaseAuthRequest>();

    return parseInt(request.user?.user_metadata.sub as string, 10);
  },
);

```
