# Configurando Variáveis de Ambiente com `@t3-oss/env-nextjs` e `zod` no Next.js

Este documento detalha como adicionar e configurar variáveis de ambiente em um projeto Next.js utilizando a biblioteca `@t3-oss/env-nextjs` junto com `zod` para validação de tipos segura. Isso garante que todas as variáveis necessárias estejam presentes e corretamente tipadas durante a execução da aplicação.

## Pré-requisitos

- Projeto Next.js já configurado.
- Node.js e npm/bun instalados.
- Terminal disponível para instalar dependências.

## Passos de Configuração

1. **Instalar Dependências:**
   - Abra o terminal na raiz do seu projeto.
   - Adicione as dependências `@t3-oss/env-nextjs` e `zod` ao `package.json` executando o seguinte comando:

    ```bash
    bun add @t3-oss/env-nextjs zod
    ```

    **Isso atualizará seu package.json com as seguintes entradas:**

    ```json
    {
      "dependencies": {
        "@t3-oss/env-nextjs": "^0.0.0",
        "zod": "^0.0.0"
      }
    }
    ```

2.  **Criar Arquivo de Configuração de Variáveis de Ambiente (`src/config/env.ts`)**
    - Crie um arquivo chamado `env.ts` dentro do diretório `src/config`
    - Adicione o seguinte código para definir e validar as variáveis de ambiente:

    ```typescript
    import { createEnv } from "@t3-oss/env-nextjs";
    import { z } from "zod";

    export const env = createEnv({
      server: {
        NODE_ENV: z.enum(["development", "production"]).default("development"),
        DATABASE_URL: z.string().min(1),
        AUTH_SECRET: z.string().min(1),
      },
      client: {
        NEXT_PUBLIC_APP_URL: z.string().min(1),
      },
      runtimeEnv: {
        NODE_ENV: process.env.NODE_ENV,
        DATABASE_URL: process.env.DATABASE_URL,
        AUTH_SECRET: process.env.AUTH_SECRET,
        NEXT_PUBLIC_APP_URL: process.env.NEXT_PUBLIC_APP_URL,
      },
    });
```
