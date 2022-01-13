---
emoji: ''
title: 'Nest.js: Authentication & Authorization'
date: '2021-09-15 23:00:00'
author: seoyul
tags: backend
categories: BackEnd
---

## 초기 flow

1. 이메일을 입력하면 해당 이메일로 인증 코드 전송
2. 인증 코드가 확인되면 access token, refresh token을 cookie에 담아 response

   \*cookie 옵션으로 maxAge, sameSite, secure, httponly를 설정한다.

3. 클라이언트 측에서 저장된 cookie와 함께 request
4. 서버에서 request header cookie 에 담긴 token 을 secret key 와 함께 확인
5. 토큰이 valid 하며 일치할 경우 정상적으로 response, 일치하지 않을 경우 error를 보낸다.
6. 토큰이 만료되었을 경우 refresh token으로 access token을 재발급 받아 사용한다.

\*httpOnly

이 옵션은 자바스크립트 같은 클라이언트 측 스크립트가 쿠키를 사용할 수 없게 한다. `document.cookie`를 통해 쿠키를 볼 수도 없고 조작할 수도 없다.

해커가 악의적인 자바스크립트 코드를 페이지에 삽입하고 사용자가 그 페이지에 접속하기를 기다리는 방식의 공격을 예방할 때 이 옵션을 사용한다.(CSS(Cross Site Scripting))

사용자가 웹 페이지에 방문할 때 `document.cookie`를 볼 수 있고 조작도 할 수 있는 해커의 코드도 함께 실행될 경우 쿠키에 인증 정보가 있어서 해커가 이 정보를 훔치거나 조작할 수 있게 된다.

하지만 `httpOnly` 옵션이 설정된 쿠키는 `document.cookie`로 쿠키 정보를 읽을 수 없기 때문에 쿠키를 보호할 수 있다.

\*secure

Secure 옵션을 적용하면, HTTPS 프로토콜을 사용할 때에만 전송된다.

HTTP Only Cookie를 사용하면 Client에서 Javascript를 통한 쿠키 탈취문제를 예방할 수 있지만 Javascript가 아닌 네트워크를 직접 감청하여 쿠키를 가로챌 수도 있다.

이러한 통신상의 정보유출을 막기 위해, HTTPS 프로토콜을 사용하여 데이터를 암호화하는 방법이 주로 사용되고 있으며 HTTPS를 사용하면 쿠키 또한 암호화되어 전송되기 때문에, 제3자는 내용을 알 수 없게 된다.

\*sameSite

서버가 사이트 간 요청과 함께 쿠키를 보내서는 안 된다고 주장할 수 있으며, 사이트 간 요청 위조 공격에 대한 일부 보호 기능을 제공한다.(CSRF)

`SameSite` 옵션은 쿠키가 `Cross-Site` 의 요청과 함께 전달되지 않음을 요구하는 방식이다.

### XSRF(cross-site request forgery, XSRF)

![Screen Shot 2021-09-15 at 12.13.46 AM.png](https://marble-experience-963.notion.site/image/https%3A%2F%2Fs3-us-west-2.amazonaws.com%2Fsecure.notion-static.com%2F1fc79d96-acf5-44fc-a8d5-5733e5d608a9%2FScreen_Shot_2021-09-15_at_12.13.46_AM.png?table=block&id=164c348f-7512-40b9-b7ab-c2f76f6df1ac&spaceId=959eefb3-3390-4ad1-a9d4-24291ee14327&width=2000&userId=&cache=v2)

현재 `bank.com`에 로그인되어있다고 가정하면 해당 사이트에서 사용되는 인증 쿠키가 브라우저에 저장되고, 브라우저는 `bank.com`에 요청을 보낼 때마다 인증 쿠키를 함께 전송한다. 서버는 전송받은 쿠키를 이용해 사용자를 식별하고, 보안이 필요한 재정 거래를 처리한다.

이제 (로그아웃하지 않고) 다른 창을 띄워서 웹 서핑을 하던 도중에 뜻하지 않게 `evil.com`에 접속했는데 이 사이트엔 해커에게 송금을 요청하는 폼(form) `<form action="https://bank.com/pay">`이 있고, 이 폼은 자동으로 제출되도록 설정되어 있을경우 폼이 `evil.com`에서 은행 사이트로 바로 전송될 때 인증 쿠키도 함께 전송된다.

`bank.com`에 요청을 보낼 때마다 `bank.com`에서 설정한 쿠키가 전송되기 때문에 은행은 전송받은 쿠키를 읽어 (해커가 아닌) 계정 주인이 접속한 것이라 생각하고 해커에게 돈을 송금한다.

## Jwt Strategy, Jwt Guard 생성

```jsx
//jwt.strategy.ts
import { Injectable } from "@nestjs/common";
import { ConfigService } from "@nestjs/config";
import { PassportStrategy } from "@nestjs/passport";
import cookie from "cookie";
import { ExtractJwt, Strategy } from "passport-jwt";

import { CustomError, ErrorCode } from "src/common/utils/error";
import { AdminUserQueryService } from "src/admin-user/services/admin-user.query.service";

export interface Payload {
  email: string;
  exp: number;
  iat: number;
  sub: string;
}

@Injectable()
export class JtwStrategy extends PassportStrategy(Strategy) {
  constructor(
    readonly configService: ConfigService,
    private readonly adminUserQueryService: AdminUserQueryService
  ) {
    super({
      ignoreExpiration: false,
      jwtFromRequest: ExtractJwt.fromExtractors([
        (request) => {
          const cookies = request.headers.cookie;

          if (!cookies) {
            throw new CustomError(
              "쿠키가 존재하지 않습니다.",
              ErrorCode.COOKIE_DOES_NOT_EXIST
            );
          }

          const accessToken = cookie.parse(cookies).AccessToken;
          const refreshToken = cookie.parse(cookies).RefreshToken;

          if (!accessToken && !refreshToken) {
            throw new CustomError(
              "토큰이 존재하지 않습니다.",
              ErrorCode.ACCESS_AND_REFRESH_TOKEN_DOES_NOT_EXIST
            );
          }

          if (!accessToken) {
            throw new CustomError(
              "토큰이 존재하지 않습니다.",
              ErrorCode.ACCESS_TOKEN_DOES_NOT_EXIST
            );
          }

          return accessToken;
        },
      ]),
      secretOrKey: configService.get("JWT_ACCESS_TOKEN_SECRET"),
    });
  }

  async validate(payload: Payload) {
    const user = this.adminUserQueryService.getUserByEmail(
      payload.email
    );

    if (!user) {
      throw new CustomError(
        "사용자가 존재하지 않습니다.",
        ErrorCode.USER_DOES_NOT_EXIST
      );
    }

    return user;
  }
}

//payload
{
  email: 'seoyul.kim@.....com',
  sub: '14766dae-6c9c-41a0-91a6-4422817335d5',
  iat: 1631675933,
  exp: 1631679533
}
```

```jsx
//jwt.guard.ts
import { Injectable, ExecutionContext } from "@nestjs/common";
import { Reflector } from "@nestjs/core";
import { GqlExecutionContext } from "@nestjs/graphql";
import { AuthGuard } from "@nestjs/passport";

@Injectable()
export class JwtAuthGuard extends AuthGuard("jwt") {
  constructor(private reflector: Reflector) {
    super();
  }

  getRequest(context: ExecutionContext) {
    const ctx = GqlExecutionContext.create(context);

    return ctx.getContext().req;
  }

  canActivate(context: ExecutionContext) {
    const isPublic = this.reflector.getAllAndOverride<boolean>(
      process.env["IS_PUBLIC_KEY"],
      [context.getHandler(), context.getClass()]
    );

    if (isPublic) {
      return true;
    }

    return super.canActivate(context);
  }
}
```

```jsx
@Module({
  imports: [
    GraphQLModule.forRoot({
      autoSchemaFile: true,
      context: ({ req, res }) => makeContext({ res, req }),
      installSubscriptionHandlers: true,
      playground: !isProductionMode,
      formatError: (err: GraphQLError) => {
        if (err?.extensions?.exception && !err.extensions.exception.errorCode) {
          err.extensions.exception.errorCode = 'SYSTEM_DEFAULT';
          err.extensions.exception.name = 'SYSTEM_DEFAULT';
        }

        return err;
      },
      plugins,
      cors: {
        credentials: true,
        origin: true,
      },
    }),
    MailerModule.forRoot({
      transport: {
        SES: ses,
      },
      defaults: {
        from: 'noreply@.....com',
      },
      template: {
        dir: 'src/templates',
        adapter: new HandlebarsAdapter(),
        options: {
          strict: true,
        },
      },
    }),
    ConfigModule.forRoot({
      validationSchema: Joi.object({
        JWT_ACCESS_TOKEN_SECRET: Joi.string().required(),
        JWT_REFRESH_TOKEN_SECRET: Joi.string().required(),
        IS_PUBLIC_KEY: Joi.string().required(),
      }),
    }),
    ...Module,
  ],
  providers: [{ provide: APP_GUARD, useClass: JwtAuthGuard }],
})
export class AppModule {}
```

```jsx
//skip-auth.decorator.ts
import { SetMetadata } from '@nestjs/common';

export const Public = () => SetMetadata(process.env['IS_PUBLIC_KEY'], true);
```

## Jwt-Refresh strategy, Jwt-Refresh guard 생성

```jsx
//jwt-refresh.strategy.ts
import { Injectable } from "@nestjs/common";
import { ConfigService } from "@nestjs/config";
import { PassportStrategy } from "@nestjs/passport";
import cookie from "cookie";
import { ExtractJwt, Strategy } from "passport-jwt";

import { Payload } from "src/auth/strategies/jwt.strategy";
import { CustomError, ErrorCode } from "src/common/utils/error";
import { RequestParam } from "src/context";
import { AdminUserMutationService } from "src/admin-user/services/admin-user.mutation.service";

@Injectable()
export class JwtRefreshStrategy extends PassportStrategy(
  Strategy,
  "jwt-refresh-token"
) {
  constructor(
    private readonly adminUserMutationService: AdminUserMutationService,
    readonly configService: ConfigService
  ) {
    super({
      ignoreExpiration: false,
      jwtFromRequest: ExtractJwt.fromExtractors([
        (request) => {
          const cookies = request.headers.cookie;

          if (!cookies) {
            throw new CustomError(
              "쿠키가 존재하지 않습니다.",
              ErrorCode.COOKIE_DOES_NOT_EXIST
            );
          }
          const refreshToken = cookie.parse(cookies).RefreshToken;

          if (!refreshToken) {
            throw new CustomError(
              "토큰이 존재하지 않습니다.",
              ErrorCode.REFRESH_TOKEN_DOES_NOT_EXIST
            );
          }
          return refreshToken;
        },
      ]),
      passReqToCallback: true,
      secretOrKey: configService.get("JWT_REFRESH_TOKEN_SECRET"),
    });
  }
  async validate(req: RequestParam, payload: Payload) {
    const cookies = req.headers.cookie;

    if (!cookies) {
      throw new CustomError(
        "쿠키가 존재하지 않습니다.",
        ErrorCode.COOKIE_DOES_NOT_EXIST
      );
    }

    const refreshToken = cookie.parse(cookies)?.RefreshToken;
    const user = this.adminUserMutationService.validateRefreshToken({
      refreshToken: refreshToken!,
      userID: payload.sub,
    });

    return user;
  }
}
```

```jsx
//jwt-refresh.guard.ts
import { ExecutionContext, Injectable } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';
import { AuthGuard } from '@nestjs/passport';

@Injectable()
export class JwtRefreshGuard extends AuthGuard('jwt-refresh-token') {
  constructor() {
    super();
  }

  getRequest(context: ExecutionContext) {
    const ctx = GqlExecutionContext.create(context);
    return ctx.getContext().req;
  }

  canActivate(context: ExecutionContext) {
    return super.canActivate(context);
  }
}
```

## Resolver 에서 사용

```tsx
@Public()
  @UseGuards(JwtRefreshGuard)
  @Mutation(() => Boolean)
  async regenerateToken(
    @Context() ctx: IContext,
    @User() user: AdminUser
  ): Promise<Boolean> {
    const { accessToken, accessTokenOption } =
      await this.authMutationService.generateToken(user.email);

    ctx.res.cookie("AccessToken", accessToken, accessTokenOption);

    return true;
  }
```

```tsx
//user.decorator.ts
import { createParamDecorator, ExecutionContext } from '@nestjs/common';
import { GqlExecutionContext } from '@nestjs/graphql';

export const User = createParamDecorator(
  (_, ctx: ExecutionContext) => GqlExecutionContext.create(ctx).getContext().req.user,
);
```

\*문제

⇒ 현재 safari는 cross domain cookie set이 되지 않는다.

\*해결방안

⇒ 현재 백엔드에서 사용하고 있는 api gateway에 custom domain을 달아준다. [참고](https://spectrum.chat/apollo/apollo-client/unable-to-set-cookie-from-server-on-safari~e6a621bc-a9bd-469d-b0a9-8b7e93cf04a6)

```toc

```
