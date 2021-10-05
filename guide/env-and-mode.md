# Variáveis e modos de ambiente

## Env Variables

Vite expõe variáveis env no objeto especial **`import.meta.env`**.  Algumas variáveis ​​integradas estão disponíveis em todos os casos:

- **`import.meta.env.MODE`**: {string} o [mode](#modes) que o aplicativo está sendo executado.

- **`import.meta.env.BASE_URL`**: {string} o url base a partir do qual o aplicativo está sendo servido. Isso é determinado pela [`base` config option](/config/#base).

- **`import.meta.env.PROD`**: {boolean} se o aplicativo está sendo executado em produção.

- **`import.meta.env.DEV`**: {boolean} se o aplicativo está sendo executado em desenvolvimento (sempre o oposto de `import.meta.env.PROD`)

### Substituição de Produção

Durante a produção, essas variáveis env são **substituídas estaticamente**. Portanto, é necessário sempre referenciá-los usando a string estática completa. Por exemplo, o acesso de chave dinâmica como `import.meta.env[key]` não funcionará.

Ele também substituirá essas strings que aparecem em strings JavaScript e modelos Vue. Isso deve ser um caso raro, mas pode não ser intencional. Existem maneiras de contornar esse comportamento:

- Para strings JavaScript, você pode quebrar a string com um espaço unicode de largura zero, por exemplo `'import.meta\u200b.env.MODE'`.

- Para modelos Vue ou outro HTML compilado em strings JavaScript, você pode usar a [`<wbr>` tag](https://developer.mozilla.org/en-US/docs/Web/HTML/Element/wbr), por exemplo `import.meta.<wbr>env.MODE`.

## Arquivos `.env` 

Vite usa o [dotenv](https://github.com/motdotla/dotenv) para carregar variáveis de ambiente adicionais dos seguintes arquivos em seu [environment directory](/config/#envdir):

```
.env                # loaded in all cases
.env.local          # loaded in all cases, ignored by git
.env.[mode]         # only loaded in specified mode
.env.[mode].local   # only loaded in specified mode, ignored by git
```

As variáveis env carregadas também são expostas ao código-fonte do cliente por meio de `import.meta.env`.

Para evitar o vazamento acidental de variáveis env para o cliente, apenas as variáveis prefixadas com `VITE_` são expostas ao seu código processado pelo Vite. por exemplo, o seguinte arquivo:

```
DB_PASSWORD=foobar
VITE_SOME_KEY=123
```

Apenas `VITE_SOME_KEY` será exposto quanto `import.meta.env.VITE_SOME_KEY` ao código-fonte do seu cliente, mas `DB_PASSWORD` não será.

:::warning SECURITY NOTES

- `.env.*.local` os arquivos são apenas locais e podem conter variáveis confidenciais. Você deve adicionar `.local` ao seu `.gitignore` para evitar que sejam verificados no git.

- Uma vez que quaisquer variáveis expostas ao seu código-fonte Vite acabarão em seu pacote de cliente, as `VITE_*` variáveis não devem conter nenhuma informação sensível.
  :::

### IntelliSense

Por padrão, o Vite fornece definição de tipo para `import.meta.env`. Embora você possa definir mais variáveis de env personalizadas em arquivos `.env.[mode]`, você pode desejar obter o TypeScript IntelliSense para variáveis de env definidas pelo usuário com o prefixo `VITE_`.

Para conseguir, você pode criar um `env.d.ts` no `src` diretório, em seguida, extenda `ImportMetaEnv` assim:

```typescript
interface ImportMetaEnv {
  VITE_APP_TITLE: string
  // more env variables...
}
```

## Modos

Por padrão, o servidor dev (`serve` comando) é executado no modo `development`, e o `build` comando é executado no modo `production`.

Isso significa que, durante a execução `vite build`, ele carregará as variáveis env de `.env.production` se houver uma:

```
# .env.production
VITE_APP_TITLE=My App
```

Em seu aplicativo, você pode renderizar o título usando `import.meta.env.VITE_APP_TITLE`.

No entanto, é importante entender que modo é um conceito mais amplo do que apenas desenvolvimento versus produção. Um exemplo típico é que você pode querer um modo de "teste" em que deve ter um comportamento semelhante ao da produção, mas com variáveis env ligeiramente diferentes da produção.

Você pode sobrescrever o modo padrão usado para um comando passando o `--mode` sinalizador de opção. Por exemplo, se você deseja construir seu aplicativo para nosso modo de teste hipotético:

```bash
vite build --mode staging
```

E para obter o comportamento que desejamos, precisamos de um arquivo `.env.staging`:

```
# .env.staging
NODE_ENV=production
VITE_APP_TITLE=My App (staging)
```

Agora, seu aplicativo de teste deve ter um comportamento semelhante ao de produção, mas exibindo um título diferente da produção.
