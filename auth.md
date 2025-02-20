# Configurando Autenticação com Better Auth e Prisma no Next.js

Este documento detalha como configurar a autenticação em um projeto Next.js usando `better-auth` com o adaptador Prisma, adicionando modelos de usuário, sessões e contas, e configurando rotas protegidas e públicas.

## Pré-requisitos

- VS Code instalado.
- Node.js e npm/bun instalados.
- Projeto Next.js configurado.
- Prisma instalado e configurado com um banco de dados PostgreSQL.
- `bun` instalado no seu projeto.

## Passos de Configuração

1.  **Instalar Dependências:**
    - Abra o terminal na raiz do seu projeto.
    - Execute o seguinte comando para instalar as dependências necessárias:

    ```bash
    bun add better-auth path-to-regexp
    ```

2.  **Atualizar o Schema do Prisma:**
    - Abra o arquivo `prisma/schema.prisma`.
    - Adicione os seguintes modelos:

    ```typescript
    model User {
      id            String    @id
      name          String
      email         String    @unique
      emailVerified Boolean   @default(false) @map("email_verified")
      image         String?
      createdAt     DateTime  @map("created_at")
      updatedAt     DateTime  @map("updated_at")
      sessions      Session[]
      accounts      Account[]
      plan          UserPlan?
      stores        Store[]

      @@map("user")
    }

    model Session {
      id        String   @id
      expiresAt DateTime @map("expires_at")
      token     String   @unique
      createdAt DateTime @map("created_at")
      updatedAt DateTime @map("updated_at")
      ipAddress String?  @map("ip_address")
      userAgent String?  @map("user_agent")
      userId    String
      user      User     @relation(fields: [userId], references: [id], onDelete: Cascade)

      @@map("session")
    }

    model Account {
      id                    String    @id
      accountId             String
      providerId            String
      userId                String
      accessToken           String?
      refreshToken          String?
      idToken               String?
      accessTokenExpiresAt  DateTime? @map("access_token_expires_at")
      refreshTokenExpiresAt DateTime? @map("refresh_token_expires_at")
      scope                 String?
      password              String?
      createdAt             DateTime  @map("created_at")
      updatedAt             DateTime  @map("updated_at")
      user                  User      @relation(fields: [userId], references: [id], onDelete: Cascade)

      @@map("account")
    }

    model Verification {
      id         String    @id
      identifier String
      value      String
      expiresAt  DateTime  @map("expires_at")
      createdAt  DateTime? @map("created_at")
      updatedAt  DateTime? @map("updated_at")

      @@map("verification")
    }
    ```

    - Execute `bunx prisma db push` para aplicar as mudanças ao banco de dados.

3.  **Criar Arquivo de Configuração do Auth (`src/lib/auth.ts`):**
    - Crie um arquivo chamado `auth.ts` dentro do diretório `src/lib`.
    - Adicione o seguinte código:

    ```typescript
    import { betterAuth } from "better-auth";
    import { prismaAdapter } from "better-auth/adapters/prisma";
    import { nextCookies } from "better-auth/next-js";

    import { env } from "@/config/env";
    import { db } from "@/lib/db";

    export const auth = betterAuth({
      database: prismaAdapter(db, { provider: "postgresql" }),
      plugins: [nextCookies()],
      secret: env.AUTH_SECRET,
      baseURL: env.NEXT_PUBLIC_APP_URL,
      emailAndPassword: { enabled: true },
    });
    ```

4.  **Criar Cliente de Autenticação (`src/lib/auth.client.ts`):**
    - Crie um arquivo chamado `auth.client.ts` dentro do diretório `src/lib`.
    - Adicione o seguinte código:

    ```typescript
    import { createAuthClient } from "better-auth/react";

    import { env } from "@/config/env";

    export const authClient = createAuthClient({
      baseURL: env.NEXT_PUBLIC_APP_URL,
    });
    ```

5.  **Criar Função de Autenticação do Lado do Servidor (`src/lib/auth-server.ts`):**
    - Crie um arquivo chamado `auth-server.ts` dentro do diretório `src/lib`.
    - Adicione o seguinte código:

    ```typescript
    import { headers } from "next/headers";
    import { redirect } from "next/navigation";

    import { Paths } from "@/config/constants";

    import { auth } from "./auth";

    export async function authServer() {
      const session = await auth.api.getSession({ headers: await headers() });
      const userId = session?.user.id;

      if (!session) {
        redirect(Paths.SIGN_IN);
      }

      return { session, userId };
    }
    ```

6.  **Adicionar Constantes de Rotas (`src/config/constants.ts`):**
    - Abra o arquivo `src/config/constants.ts`.
    - Adicione o seguinte código:

    ```typescript
    export enum Paths {
      HOME = "/",
      SIGN_IN = "/sign-in",
      SIGN_UP = "/sign-up",
      VERIFY_EMAIL = "/verify-email",
      FORGOT_PASSWORD = "/forgot-password",
    }

    export const PUBLIC_ROUTES = [
      "/sign-in",
      "/sign-up",
      "/forgot-password",
      "/reset-password",
    ];
    ```

7.  **Adicionar Mensagens de Erro (`src/lib/error-messages.ts`):**
    - Crie um arquivo chamado `error-messages.ts` dentro do diretório `src/lib`.
    - Adicione o seguinte código:

    ```typescript
    export const errorMessages: Record<string, string> = {
      USER_ALREADY_EXISTS: "Email já cadastrado",
      INVALID_EMAIL_OR_PASSWORD: "Credenciais inválidas",
      "": "Erro inesperado",
    };
    ```

8.  **Configurar Middleware (`src/middleware.ts`):**
    - Crie ou atualize o arquivo `src/middleware.ts`.
    - Adicione o seguinte código:

    ```typescript
    import type { NextRequest } from "next/server";
    import { NextResponse } from "next/server";

    import { getSessionCookie } from "better-auth";
    import { match } from "path-to-regexp";

    import { PUBLIC_ROUTES, Paths } from "@/config/constants";

    export async function middleware(request: NextRequest): Promise<NextResponse> {
      const { nextUrl, headers } = request;

      const ip = headers.get("x-forwarded-for");
      if (!ip) {
        return new NextResponse(null, { status: 400, statusText: "Invalid IP" });
      }

      const sessionCookie = getSessionCookie(request);
      const isLoggedIn = !!sessionCookie;

      const isPublicRoute = PUBLIC_ROUTES.some((route) => {
        const matchRoute = match(route, { decode: decodeURIComponent });
        return !!matchRoute(nextUrl.pathname);
      });

      if (isPublicRoute && isLoggedIn) {
        return NextResponse.redirect(new URL(Paths.HOME, nextUrl));
      }

      if (!isPublicRoute && !isLoggedIn) {
        return NextResponse.redirect(new URL(Paths.SIGN_IN, nextUrl));
      }

      return NextResponse.next();
    }
    export const config = {
      matcher: [
        "/((?!_next|[^?]*\\.(?:html?|css|js(?!on)|jpe?g|webp|png|gif|svg|ttf|woff2?|ico|csv|docx?|xlsx?|zip|webmanifest)).*)",
        "/(api|trpc)(.*)",
      ],
    };
    ```

9.  **Configurar Rotas da API de Autenticação (`src/api/auth/[...all]/route.ts`):**
    - Crie o arquivo `src/api/auth/[...all]/route.ts`.
    - Adicione o seguinte código:

    ```typescript
    import { toNextJsHandler } from "better-auth/next-js";
    import { auth } from "@/lib/auth";

    export const { POST, GET } = toNextJsHandler(auth);
    ```

10. **Criar Action de Cadastro de Usuário (`src/app/actions/auth/sign-up-action.ts`):**
    - Crie um arquivo chamado sign-up-action.ts dentro do diretório src/app/actions/auth
    - Adicione o seguinte código para a action de signUp de usuário:

    ```typescript
    'use server';

    import { APIError } from "better-auth";
    import { errorMessages } from "@/lib/error-messages";
    import { auth } from "@/lib/auth";
    import { revalidatePath } from "next/cache";
    import { Paths } from "@/config/constants";

    export async function signUpAction(formData: FormData) {
      const name = formData.get("name") as string;
      const email = formData.get("email") as string;
      const password = formData.get("password") as string;

      try {
        await auth.api.signUpEmail({ body: { email, password, name } });
        revalidatePath(Paths.HOME);
        return { message: "Conta criada com sucesso", status: "success" };
      } catch (error) {
        if (error instanceof APIError) {
          return { message: errorMessages[error.body.code || ""], status: "error" };
        }
        return { message: "Erro ao criar conta", status: "error" };
      }
    }
    ```

11. **Criar Action de Login de Usuário (`src/app/actions/auth/sign-in-action.ts`):**
    - Crie um arquivo chamado sign-in-action.ts dentro do diretório src/app/actions/auth.
    - Adicione o seguinte código para a action de singIn de usuário:

    ```TypeScript
    'use server';

    import { APIError } from "better-auth";
    import { errorMessages } from "@/lib/error-messages";
    import { auth } from "@/lib/auth";
    import { revalidatePath } from "next/cache";
    import { Paths } from "@/config/constants";

    export async function signInAction(formData: FormData) {
      const email = formData.get("email") as string;
      const password = formData.get("password") as string;

      try {
        await auth.api.signInEmail({ body: { email, password } });
        revalidatePath(Paths.HOME);
        return { message: "Login realizado com sucesso", status: "success" };
      } catch (error) {
        if (error instanceof APIError) {
          console.log(error.body);
          return { message: errorMessages[error.body.code || ""], status: "error" };
        }
        return { message: "Erro ao realizar login", status: "error" };
      }
    }
    ```
