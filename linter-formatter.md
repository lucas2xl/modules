# Configurando Biome no VS Code

Este documento detalha como configurar o Biome no VS Code para formatação e linting de código, além de adicionar scripts no `package.json` para facilitar o uso do Biome.

## Pré-requisitos

- VS Code instalado.
- Extensão Biome para VS Code instalada (`biomejs.biome`).
- `bun` instalado no seu projeto.

## Passos de Configuração

1.  **Instalar a Extensão Biome:**
    - Abra o VS Code.
    - Vá para a aba de extensões (Ctrl+Shift+X ou Cmd+Shift+X).
    - Pesquise por "biomejs.biome" e instale a extensão.

2.  **Configurar as Configurações do Usuário (JSON):**
    - Abra as configurações do usuário no formato JSON:
        - Windows/Linux: `Arquivo > Preferências > Configurações > Abrir Configurações do Usuário (JSON)`
        - macOS: `Código > Preferências > Configurações > Abrir Configurações do Usuário (JSON)`
    - Adicione ou modifique o seguinte código JSON:

    ```json
    {
      // ...
      "editor.defaultFormatter": "biomejs.biome",
      "editor.codeActionsOnSave": {
        "source.organizeImports.biome": "explicit",
        "quickfix.biome": "explicit",
        "source.fixAll": "explicit"
      }
      // ...
    }
    ```

    - **Explicação:**
        - `"editor.defaultFormatter": "biomejs.biome"`: Define o Biome como o formatador padrão.
        - `"editor.codeActionsOnSave"`: Configura as ações ao salvar o arquivo.

3.  **Adicionar o Biome como dependência no `package.json`:**

    ```bun add --dev --exact @biomejs/biome```

4.  **Criar o Arquivo de Configuração do Biome (`biome.json`):**
    - Na raiz do seu projeto, crie um arquivo chamado `biome.json`.
    - Cole o seguinte código JSON no arquivo:

    ```json
    {
      "$schema": "[https://biomejs.dev/schemas/1.9.4/schema.json](https://biomejs.dev/schemas/1.9.4/schema.json)",
      "vcs": {
        "enabled": false,
        "clientKind": "git",
        "useIgnoreFile": false
      },
      "files": {
        "ignoreUnknown": false,
        "ignore": []
      },
      "organizeImports": {
        "enabled": true
      },
      "formatter": {
        "enabled": true,
        "formatWithErrors": false,
        "ignore": [],
        "attributePosition": "auto",
        "indentStyle": "space",
        "indentWidth": 2,
        "lineWidth": 80,
        "lineEnding": "lf"
      },
      "javascript": {
        "formatter": {
          "semicolons": "always",
          "quoteStyle": "single",
          "lineEnding": "lf"
        }
      },
      "linter": {
        "enabled": true,
        "rules": {
          "recommended": true,
          "suspicious": {
            "noConsoleLog": "error",
            "noImplicitAnyLet": "off",
            "noExplicitAny": "off",
            "noArrayIndexKey": "off",
            "noAssignInExpressions": "off"
          },
          "performance": {
            "noDelete": "off"
          },
          "complexity": {
            "noExtraBooleanCast": "off",
            "noForEach": "off"
          },
          "correctness": {
            "useExhaustiveDependencies": "off"
          },
          "style": {
            "noNonNullAssertion": "off"
          },
          "a11y": {
            "useFocusableInteractive": "off",
            "useSemanticElements": "off",
            "noLabelWithoutControl": "off",
            "useMediaCaption": "off",
            "useKeyWithClickEvents": "off",
            "useButtonType": "off"
          },
          "security": {
            "noDangerouslySetInnerHtml": "off"
          }
        }
      }
    }
    ```

    - **Explicação:**
        - `$schema`: Define o esquema JSON.
        - `vcs`, `files`, `organizeImports`, `formatter`, `javascript`, `linter`: Configuram o comportamento do Biome.

4.  **Adicionar Scripts ao `package.json`:**
    - Abra o arquivo `package.json` na raiz do seu projeto.
    - Adicione os seguintes scripts dentro da seção `"scripts"`:

    ```json
    {
      // ...
      "scripts": {
        "lint": "bun run biome lint --write --unsafe ./src",
        "check": "bun run biome check --write --unsafe ./src",
        "format": "bun run biome format --write ./src"
      },
      // ...
    }
    ```

    - **Explicação dos Scripts:**
        - `"lint"`: Executa o linter do Biome.
        - `"check"`: Executa a verificação do Biome.
        - `"format"`: Executa o formatador do Biome.
        - `"--write"`: Aplica correções automaticamente.
        - `"--unsafe"`: Permite correções potencialmente perigosas (use com cautela).
        - `"./src"`: Especifica o diretório alvo.

## Uso dos Scripts

Para executar os scripts, use os seguintes comandos no terminal:

- `bun run lint`
- `bun run check`
- `bun run format`

## Substituindo `.prettierrc.json` e `.eslintrc.json`

O arquivo `biome.json` substitui os arquivos de configuração `.prettierrc.json` e `.eslintrc.json`. Remova esses arquivos se eles existirem no seu projeto.
