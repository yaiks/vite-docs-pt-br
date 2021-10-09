# Configurando o Vite

## Arquivo de configuração

### Resolvendo o arquivo de configuração

Ao executar o `vite` na linha de comando, o Vite tentará automaticamente resolver um arquivo de configuração chamado `vite.config.js` dentro da [raiz do projeto](/guide/#index-html-e-a-raiz-do-projeto).

O arquivo de configuração mais básico se parece com este:

```js
// vite.config.js
export default {
  // opções de configuração
}
```

Observe que, o Vite suporta o uso de sintaxe de módulos ES no arquivo de configuração, mesmo se o projeto não estiver usando Node ESM nativo via `type:"module"`. Nesse caso, o arquivo de configuração é pré-processado automaticamente antes do carregamento.

Você também pode especificar explicitamente um arquivo de configuração por linha de comando com a opção `--config` (resolvido em relação a `cwd`):

```bash
vite --config my-config.js
```

### Configurando o Intellisense

Como o Vite vem com tipagens TypeScript, você pode aproveitar o intellisense da sua IDE com jsdoc type hints:

```js
/**
 * @type {import('vite').UserConfig}
 */
const config = {
  // ...
}

export default config
```

Como alternativa, você pode usar o helper `defineConfig`, que deve fornecer intellisense sem a necessidade de anotações jsdoc:

```js
import { defineConfig } from 'vite'

export default defineConfig({
  // ...
})
```

O Vite também oferece suporte direto a arquivos de configuração TS. Você também pode usar `vite.config.ts` com o helper `defineConfig`.

### Configuração Condicional

Se a configuração precisar de condicionais, determine as opções com base no command (`serve` ou` build`) ou no [mode](/guide/env-and-mode) sendo usado, ele pode exportar uma função:

```js
export default defineConfig(({ command, mode }) => {
  if (command === 'serve') {
    return {
      // configuração específica para serve
    }
  } else {
    return {
      // configuração específica para build
    }
  }
})
```

### Configuração Assíncrona

Se a configuração precisar chamar uma função assíncrona, ele poderá exportar uma função assíncrona:

```js
export default defineConfig(async ({ command, mode }) => {
  const data = await asyncFunction()
  return {
    // configuração específica para build
  }
})
```

## Opções Compartilhadas

### root

- **Type:** `string`
- **Default:** `process.cwd()`

  Diretório raiz do projeto (onde `index.html` está localizado). Pode ser um caminho absoluto ou relativo ao local do próprio arquivo de configuração.

  Consulte [Raiz do Projeto](/guide/#index-html-e-a-raiz-do-projeto) para obter mais detalhes.

### base

- **Type:** `string`
- **Default:** `/`

  Base para o Caminho público quando servido em desenvolvimento ou produção. Os valores válidos incluem:

  - Nome do caminho de URL absoluto, por exemplo `/foo/`
  - URL completo, por exemplo `https://foo.com/`
  - String vazia or `./` (for embedded deployment)

  Consulte [Caminho Base para a Public](/guide/build#caminho-base-para-a-public) para obter mais detalhes.

### mode

- **Type:** `string`
- **Default:** `'development'` para desenvolvimento, `'production'` para produção

  Specifying this in config will override the default mode for **both serve and build**. This value can also be overridden via the command line `--mode` option.

  Especificar isso na configuração substituirá o modo padrão para ambos **desenvolvimento e produção**. Este valor também pode ser sobrescrito através da opção `--mode` da linha de comando.

  Consulte [Variáveis ​​e Modos de Ambiente](/guide/env-and-mode) para obter mais detalhes.

### define

- **Type:** `Record<string, string>`

  Define substituições de constantes globais. As entradas serão definidas como globais durante o desenvolvimento e substituídas estaticamente durante a compilação para produção.

  - A partir de `2.0.0-beta.70`, valores de string serão usados ​​como expressões brutas, então se definir uma constante de string, ela precisa ser explicitamente citada (por exemplo, com `JSON.stringify`).

  - As substituições são realizadas apenas quando a correspondência é cercada por limites de palavras (`\ b`).

  Por ser implementado como substituições de texto diretas sem nenhuma análise de sintaxe, recomendamos o uso de `define` apenas para CONSTANTES.

  Por exemplo, `process.env.FOO` e `__APP_VERSION__` são bons ajustes. Mas `process` ou `global` não devem ser colocados nesta opção. As variáveis ​​podem ser shim ou polyfill em vez disso.

### plugins

- **Type:** ` (Plugin | Plugin[])[]`

  Array de plugins a serem usados. Os plugins falsos são ignorados e arrays de plugins são reduzidos. Consulte [Plugin API](/guide/api-plugin) para obter mais detalhes sobre os plugins do Vite.

### publicDir

- **Type:** `string | false`
- **Default:** `"public"`

  Diretório para servir arquivos estáticos simples. Os arquivos neste diretório são servidos em `/` durante o desenvolvimento e copiados para a raiz de `outDir` durante a compilação, e são sempre servidos ou copiados como estão, sem transformação. O valor pode ser um caminho absoluto do sistema de arquivos ou um caminho relativo à raiz do projeto.

  Definindo `publicDir` como` false` desativa este recurso.

  Consulte [O Diretório `public`](/guide/assets#the-public-directory) para obter mais detalhes.

### cacheDir

- **Type:** `string`
- **Default:** `"node_modules/.vite"`

  Diretório para salvar arquivos de cache. Os arquivos neste diretório são dependências pré-empacotadas ou alguns outros arquivos de cache gerados pelo vite, o que pode melhorar o desempenho. Você pode usar a opção `--force` ou excluir manualmente o diretório para regenerar os arquivos de cache. O valor pode ser um caminho absoluto do sistema de arquivos ou um caminho relativo à raiz do projeto.

### resolve.alias

- **Type:** `Record<string, string> | Array<{ find: string | RegExp, replacement: string }>`

  Será passado para `@rollup/plugin-alias` como suas [opões de entradas](https://github.com/rollup/plugins/tree/master/packages/alias#entries). Pode ser um objeto ou um array de pares `{ localizar, substituir }`.

  Ao criar aliases para caminhos do sistema de arquivos, sempre use caminhos absolutos. Os valores de aliases relativos serão usados ​​no estado em que se encontram e não serão resolvidos nos caminhos do sistema de arquivos.

  Resolução personalizada mais avançada pode ser obtida por meio dos [plugins](/guide/api-plugin).

### resolve.dedupe

- **Type:** `string[]`

  Se você tiver cópias duplicadas da mesma dependência em seu aplicativo (provavelmente devido ao hoisting ou pacotes vinculados em monorepos), use esta opção para forçar o Vite a sempre resolver as dependências listadas para a mesma cópia (da raiz do projeto).

### resolve.conditions

- **Type:** `string[]`

  Condições adicionais permitidas ao resolver [Exportações Condicionais](https://nodejs.org/api/packages.html#packages_conditional_exports) de um pacote.

  Um pacote com exportações condicionais pode ter o seguinte campo `exports` em seu` package.json`:

  ```json
  {
    "exports": {
      ".": {
        "import": "./index.esm.js",
        "require": "./index.cjs.js"
      }
    }
  }
  ```

  Aqui, `import` e` require` são "condições". As condições podem ser aninhadas e devem ser especificadas da mais específica para a menos específica.

  O Vite tem uma lista de "condições permitidas" e irá corresponder à primeira condição que está na lista permitida. As condições padrão permitidas são: `import`, `module`, `browser`, `default` e `production / development` com base no modo atual. A opção de configuração `resolve.conditions` permite especificar condições adicionais permitidas.

### resolve.mainFields

- **Type:** `string[]`
- **Default:** `['module', 'jsnext:main', 'jsnext']`

  Lista de campos no `package.json` para quando tentar resolver o ponto de entrada de um pacote. Observe que isso tem menos precedência do que as exportações condicionais resolvidas no campo `exports`: se um ponto de entrada for resolvido com sucesso nas `exports`, o campo principal será ignorado.

### resolve.extensions

- **Type:** `string[]`
- **Default:** `['.mjs', '.js', '.ts', '.jsx', '.tsx', '.json']`

  Lista de extensões de arquivo a serem experimentadas para importações que omitem extensões. Observe que **NÃO** é recomendado omitir extensões para tipos de importação personalizados (por exemplo, `.vue`), uma vez que pode interferir na IDE e no suporte de tipo.

### css.modules

- **Type:**

  ```ts
  interface CSSModulesOptions {
    scopeBehaviour?: 'global' | 'local'
    globalModulePaths?: RegExp[]
    generateScopedName?:
      | string
      | ((name: string, filename: string, css: string) => string)
    hashPrefix?: string
    /**
     * default: 'camelCaseOnly'
     */
    localsConvention?: 'camelCase' | 'camelCaseOnly' | 'dashes' | 'dashesOnly'
  }
  ```

  Configure o comportamento do CSS modules. As opções são passadas para o [postcss-modules](https://github.com/css-modules/postcss-modules).

### css.postcss

- **Type:** `string | (postcss.ProcessOptions & { plugins?: postcss.Plugin[] })`

  Configuração PostCSS embutida (espera o mesmo formato que `postcss.config.js`), ou um caminho personalizado para pesquisar a configuração PostCSS (o padrão é a raiz do projeto). A pesquisa é feita usando [postcss-load-config](https://github.com/postcss/postcss-load-config).

  Observe que, se uma configuração inline for fornecida, o Vite não pesquisará outras fontes de configuração PostCSS.

### css.preprocessorOptions

- **Type:** `Record<string, object>`

  Especifique as opções a serem transmitidas aos pré-processadores CSS. Exemplo:

  ```js
  export default defineConfig({
    css: {
      preprocessorOptions: {
        scss: {
          additionalData: `$injectedColor: orange;`
        }
      }
    }
  })
  ```

### json.namedExports

- **Type:** `boolean`
- **Default:** `true`

  Deve ser compatível com importações nomeadas de arquivos `.json`.

### json.stringify

- **Type:** `boolean`
- **Default:** `false`

  Se definido como `true`, o JSON importado será transformado em `export default JSON.parse("...")`, que tem um desempenho significativamente maior do que os Object literals, especialmente quando o arquivo JSON é grande.

  Habilitar isso desabilita as importações nomeadas.

### esbuild

- **Type:** `ESBuildOptions | false`

  `ESBuildOptions` estende as próprias [opções de transformação do ESbuild](https://esbuild.github.io/api/#transform-api). O caso de uso mais comum é personalizar JSX:

  ```js
  export default defineConfig({
    esbuild: {
      jsxFactory: 'h',
      jsxFragment: 'Fragment'
    }
  })
  ```

  Por padrão, o ESBuild é aplicado aos arquivos `ts`, `jsx` e `tsx`. Você pode personalizar isso com `esbuild.include` e `esbuild.exclude`, ambos esperam o tipo de `string | RegExp | (string | RegExp) []`.

  Além disso, você também pode usar `esbuild.jsxInject` para injetar automaticamente importações auxiliares JSX para cada arquivo transformado por ESBuild:

  ```js
  export default defineConfig({
    esbuild: {
      jsxInject: `import React from 'react'`
    }
  })
  ```

  Defina como `false` para desabilitar as transformações ESbuild.

### assetsInclude

- **Type:** `string | RegExp | (string | RegExp)[]`
- **Relacionado:** [Static Asset Handling](/guide/assets)

  Especifique os tipos de arquivo adicionais a serem tratados como ativos estáticos para que:

  - Eles serão excluídos do pipeline de transformação do plugin quando referenciados em HTML ou solicitados diretamente por `fetch` ou XHR.

  - Importá-los do JS retornará sua string de URL resolvida (isso pode ser sobrescrito se você tiver um plugin `enforce: 'pre'` para lidar com o tipo de asset de forma diferente).

  A lista de tipo de assets integrados pode ser encontrada [aqui](https://github.com/vitejs/vite/blob/main/packages/vite/src/node/constants.ts).

### logLevel

- **Type:** `'info' | 'warn' | 'error' | 'silent'`

  Ajuste a verbosidade da saída do console. O padrão é `'info'`.

### clearScreen

- **Type:** `boolean`
- **Default:** `true`

  Defina como `false` para evitar que Vite limpe a tela do terminal ao registrar certas mensagens. Via linha de comando, use `--clearScreen false`.

### envDir

- **Type:** `string`
- **Default:** `root`

  O diretório a partir do qual os arquivos `.env` são carregados. Pode ser um caminho absoluto ou relativo à raiz do projeto.

  Consulte [aqui](/guide/env-and-mode#arquivos-env) para mais informações sobre arquivos de ambiente.

## Server Options

### server.host

- **Type:** `string`
- **Default:** `'127.0.0.1'`

  Especifique em quais endereços IP o servidor deve escutar.
  Defina como `0.0.0.0` para ouvir em todos os endereços, incluindo LAN e endereços públicos.

  Isso pode ser definido via CLI usando `--host 0.0.0.0` ou `--host`.

### server.port

- **Type:** `number`

  Especifique a porta do servidor. Observe que se a porta já estiver sendo usada, o Vite tentará automaticamente a próxima porta disponível, portanto, pode não ser a porta real em que o servidor acaba escutando.

### server.strictPort

- **Type:** `boolean`

  Defina como `true` para sair se a porta já estiver em uso, em vez de tentar automaticamente a próxima porta disponível.

### server.https

- **Type:** `boolean | https.ServerOptions`

  Ative TLS + HTTP/2. Observe que isso diminui para TLS apenas quando a [opção `server.proxy`](#server-proxy) também é usada.

  O valor também pode ser um [objeto de opções](https://nodejs.org/api/https.html#https_https_createserver_options_requestlistener) passado para `https.createServer ()`.

### server.open

- **Type:** `boolean | string`

  Abre automaticamente o aplicativo no navegador ao iniciar o servidor. Quando o valor for uma string, ele será usado como o nome do caminho da URL. Se você deseja abrir o servidor em um navegador específico de sua preferência, pode definir o env `process.env.BROWSER` (por exemplo, `firefox`). Veja [o pacote `open`](https://github.com/sindresorhus/open#app) para obter mais detalhes.

  **Exemplo:**

  ```js
  export default defineConfig({
    server: {
      open: '/docs/index.html'
    }
  })
  ```

### server.proxy

- **Type:** `Record<string, string | ProxyOptions>`

  Configure regras de proxy personalizadas para o servidor de desenvolvimento. Espera um objeto de pares `{key: options}`. Se a chave começar com `^`, será interpretada como `RegExp`. A opção `configure` pode ser usada para acessar a instância do proxy.

  Usa [`http-proxy`](https://github.com/http-party/node-http-proxy). Opções completas [aqui](https://github.com/http-party/node-http-proxy#options).

  **Exemplo:**

  ```js
  export default defineConfig({
    server: {
      proxy: {
        // string abreviada
        '/foo': 'http://localhost:4567',
        // com opções
        '/api': {
          target: 'http://jsonplaceholder.typicode.com',
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/api/, '')
        },
        // com RegEx
        '^/fallback/.*': {
          target: 'http://jsonplaceholder.typicode.com',
          changeOrigin: true,
          rewrite: (path) => path.replace(/^\/fallback/, '')
        },
        // Usando a instância do proxy
        '/api': {
          target: 'http://jsonplaceholder.typicode.com',
          changeOrigin: true,
          configure: (proxy, options) => {
            // proxy será uma instância de 'http-proxy'
          }),
        }
      }
    }
  })
  ```

### server.cors

- **Type:** `boolean | CorsOptions`

  Configure o CORS para o servidor de desenvolvimento. Isso é habilitado por padrão e permite qualquer origem. Passe um [objeto de opções](https://github.com/expressjs/cors) para ajustar o comportamento ou `false` para desativar.

### server.force

- **Type:** `boolean`
- **Relacionado:** [Dependency Pre-Bundling](/guide/dep-pre-bundling)

  Defina como `true` para forçar o pré-empacotamento de dependência.

### server.hmr

- **Type:** `boolean | { protocol?: string, host?: string, port?: number, path?: string, timeout?: number, overlay?: boolean, clientPort?: number, server?: Server }`

  Desative ou configure a conexão HMR (nos casos em que o websocket HMR deve usar um endereço diferente do servidor http).

  Defina `server.hmr.overlay` como `false` para desabilitar a sobreposição de erro do servidor.

  `clientPort` é uma opção avançada que sobrescreve a porta apenas no lado do cliente, permitindo a você servir o websocket em uma porta diferente da que o código do cliente procura. Útil se você estiver usando um proxy SSL na frente do seu servidor de desenvolvimento.

  Ao usar `server.middlewareMode` e `server.https`, configurar `server.hmr.server` para seu servidor HTTPS irá processar solicitações de conexão segura HMR através de seu servidor. Isso pode ser útil ao usar certificados autoassinados.

### server.watch

- **Type:** `object`

  Opções do watcher do sistema de arquivos para passar para o [chokidar](https://github.com/paulmillr/chokidar#api).

  Ao executar o Vite no subsistema Windows para Linux (WSL) 2, se a pasta do projeto residir em um sistema de arquivos Windows, você precisará definir esta opção para `{ usePolling: true }`. Isso se deve a [uma limitação do WSL2](https://github.com/microsoft/WSL/issues/4739) com o sistema de arquivos do Windows.

### server.middlewareMode

- **Type:** `'ssr' | 'html'`

  Crie o servidor Vite em modo de middleware. (sem um servidor HTTP)

  - `'ssr'` irá desabilitar a lógica de serviço HTML do próprio Vite para que você deva servir` index.html` manualmente.
  - `'html'` irá habilitar a lógica de serviço HTML do próprio Vite.

- **Relacionado:** [SSR - Configurando o Servidor de Desenvolvimento](/guide/ssr#configurando-o-servidor-de-desenvolvimento)

- **Exemplo:**

```js
const express = require('express')
const { createServer: createViteServer } = require('vite')

async function createServer() {
  const app = express()

  // Crie o servidor vite no modo de middleware.
  const vite = await createViteServer({
    server: { middlewareMode: 'ssr' }
  })
  // Use a instância de conexão do vite como middleware
  app.use(vite.middlewares)

  app.use('*', async (req, res) => {
    // Se `middlewareMode` é `'ssr'`, deve servir `index.html` aqui.
    // Se `middlewareMode` for `'html'`, não há necessidade de servir `index.html`
    // porque Vite fará isso.
  })
}

createServer()
```

### server.fs.strict

- **Experimental**
- **Type:** `boolean`
- **Default:** `false` (mudará para `true` em versões futuras)

  Restrinja o envio de arquivos para fora da raiz do espaço de trabalho.

### server.fs.allow

- **Experimental**
- **Type:** `string[]`

  Restrinja os arquivos que podem ser servidos via `/@fs/`. Quando `server.fs.strict` é definido como `true`, acessar arquivos fora desta lista de diretórios resultará em 403.

  Vite irá pesquisar a raiz do espaço de trabalho potencial e usá-la como padrão. Um espaço de trabalho válido atendeu às seguintes condições, caso contrário, retornará para a [raiz do projeto](/guide/#index-html-e-a-raiz-do-projeto).

  - contém o campo `workspaces` em `package.json`
  - contém um dos seguintes arquivos
    - `pnpm-workspace.yaml`

  Aceita um caminho para especificar a raiz do espaço de trabalho customizado. Pode ser um caminho absoluto ou relativo a [raiz do projeto](/guide/#index-html-e-a-raiz-do-projeto). Por exemplo:

  ```js
  export default defineConfig({
    server: {
      fs: {
        // Permitir servir arquivos de um nível acima da raiz do projeto
        allow: ['..']
      }
    }
  })
  ```

## Build Options

### build.target

- **Type:** `string`
- **Default:** `'modules'`
- **Relacionado:** [Compatibilidade do Browser](/guide/build#compatibilidade-do-browser)

  Destino de compatibilidade do browser para o pacote final. O valor padrão é um valor especial Vite, `'modules'`, que tem como alvo [navegadores com suporte a módulo ES nativo](https://caniuse.com/es6-module).

  Outro valor especial é `'esnext'` - que assume suporte para importações dinâmicas nativas e transpilará o menos possível:

  - Se a opção [`build.minify`](#build-minify) for `'terser'` (o padrão), `'esnext'` será forçado a descer para `'es2019'`.
  - Em outros casos, não realizará nenhuma transpilação.

  A transformação é realizada com esbuild e o valor deve ser uma [opção de destino esbuild](https://esbuild.github.io/api/#target) válida. Os destinos personalizados podem ser uma versão ES (por exemplo, `es2015`), um browser com versão (por exemplo, `chrome58`) ou um array de várias strings de destino.

  Observe que a compilação falhará se o código contiver recursos que não podem ser transpilados com segurança pelo esbuild. Veja [esbuild docs](https://esbuild.github.io/content-types/#javascript) para mais detalhes.

### build.polyfillModulePreload

- **Type:** `boolean`
- **Default:** `true`

  Deve injetar automaticamente o [módulo de pré-carregamento do polyfill](https://guybedford.com/es-module-preloading-integrity#modulepreload-polyfill).

  Se definido como `true`, o polyfill é injetado automaticamente no módulo proxy de cada entrada `index.html`. Se a construção estiver configurada para usar uma entrada personalizada não-html via `build.rollupOptions.input`, então é necessário importar manualmente o polyfill em sua entrada personalizada:

  ```js
  import 'vite/modulepreload-polyfill'
  ```

  Observação: o polyfill **não** se aplica ao [Modo de biblioteca](/guide/build#modo-biblioteca). Se precisar oferecer suporte a browsers sem importação dinâmica nativa, você provavelmente deve evitar usá-lo em sua biblioteca.

### build.outDir

- **Type:** `string`
- **Default:** `dist`

  Especifique o diretório de saída (relativo a [raiz do projeto](/guide/#index-html-e-a-raiz-do-projeto)).

### build.assetsDir

- **Type:** `string`
- **Default:** `assets`

  Especifique o diretório para aninhar os assets gerados (em relação a `build.outDir`).

### build.assetsInlineLimit

- **Type:** `number`
- **Default:** `4096` (4kb)

  Assets importados ou referenciados que são menores do que esse limite serão incluídos em linha como URLs de base64 para evitar solicitações HTTP extras. Defina como `0` para desabilitar completamente o inlining.

  ::: tip Nota
  Se você especificar `build.lib`, `build.assetsInlineLimit` será ignorado e os assets sempre serão embutidos, independentemente do tamanho do arquivo.
  :::

### build.cssCodeSplit

- **Type:** `boolean`
- **Default:** `true`

  Habilite/desabilite a divisão de código CSS. Quando habilitado, o CSS importado em blocos assíncronos será embutido no próprio bloco assíncrono e inserido quando o bloco for carregado.

  Se desativado, todos os CSS em todo o projeto serão extraídos em um único arquivo CSS.

### build.sourcemap

- **Type:** `boolean | 'inline' | 'hidden'`
- **Default:** `false`

  Gere sourcemaps de produção. Se `true`, um arquivo sourcemap separado será criado. Se `'inline'`, o sourcemap será anexado ao arquivo de saída resultante como um URI de dados. `'hidden'` funciona como` true` exceto que os comentários do sourcemap correspondentes nos arquivos agrupados são suprimidos.

### build.rollupOptions

- **Type:** [`RollupOptions`](https://rollupjs.org/guide/en/#big-list-of-options)

  Personalize diretamente o pacote Rollup subjacente. São as mesmas opções que podem ser exportadas de um arquivo de configuração de Rollup e serão mescladas com as opções de Rollup internas do Vite. Consulte [documentação de opções de rollup](https://rollupjs.org/guide/en/#big-list-of-options) para obter mais detalhes.

### build.commonjsOptions

- **Type:** [`RollupCommonJSOptions`](https://github.com/rollup/plugins/tree/master/packages/commonjs#options)

  Opções para passar para [@rollup/plugin-commonjs](https://github.com/rollup/plugins/tree/master/packages/commonjs).

### build.dynamicImportVarsOptions

- **Type:** [`RollupDynamicImportVarsOptions`](https://github.com/rollup/plugins/tree/master/packages/dynamic-import-vars#options)

  Opções para passar para [@rollup/plugin-dynamic-import-vars](https://github.com/rollup/plugins/tree/master/packages/dynamic-import-vars).

### build.lib

- **Type:** `{ entry: string, name?: string, formats?: ('es' | 'cjs' | 'umd' | 'iife')[], fileName?: string | ((format: ModuleFormat) => string) }`
- **Relacionado:** [Modo de biblioteca](/guide/build#modo-biblioteca)

  Construa como uma biblioteca. `entry` é necessária uma vez que a biblioteca não pode usar HTML como entrada. `name` é a variável global exposta e é necessário quando `formats` inclui `'umd'` ou `'iife'`. Os formatos (`formats`) padrão são `['es', 'umd']`. `fileName` é o nome da saída do arquivo do pacote, o padrão `fileName` é a opção de nome do package.json, também pode ser definido como uma função tomando o `format` como argumento.

### build.manifest

- **Type:** `boolean`
- **Default:** `false`
- **Relacionado:** [Backend Integration](/guide/backend-integration)

  Quando definido como `true`, a compilação também irá gerar um arquivo `manifest.json` que contém um mapeamento de nomes de arquivos de assets sem hash para suas versões com hash, que podem então ser usados por uma estrutura de servidor para renderizar os links de assets corretos.

### build.minify

- **Type:** `boolean | 'terser' | 'esbuild'`
- **Default:** `'terser'`

  Defina como `false` para desativar a minificação ou especifique o minificador a ser usado. O padrão é [Terser](https://github.com/terser/terser) que é mais lento, mas produz pacotes menores na maioria dos casos. A minificação do Esbuild é significativamente mais rápida, mas resultará em pacotes ligeiramente maiores.

### build.terserOptions

- **Type:** `TerserOptions`

  [Opções de minify](https://terser.org/docs/api-reference#minify-options) adicionais para passar para o Terser.

### build.cleanCssOptions

- **Type:** `CleanCSS.Options`

  Opções de construtor para passar para [clean-css](https://github.com/jakubpawlowicz/clean-css#constructor-options).

### build.write

- **Type:** `boolean`
- **Default:** `true`

  Defina como `false` para desativar a gravação do pacote no disco. Isso é usado principalmente em [chamadas `build()` programáticas](/guide/api-javascript#build) onde mais pós-processamento do pacote é necessário antes de gravar no disco.

### build.emptyOutDir

- **Type:** `boolean`
- **Default:** `true` se `outDir` estiver dentro do `root`

  Por padrão, o Vite irá esvaziar o `outDir` na compilação se estiver dentro da raiz do projeto. Ele emitirá um aviso se `outDir` estiver fora do root para evitar a remoção acidental de arquivos importantes. Você pode definir explicitamente essa opção para suprimir o aviso. Isso também está disponível via linha de comando como `--emptyOutDir`.

### build.brotliSize

- **Type:** `boolean`
- **Default:** `true`

  Ativar/desativar relatórios de tamanho compactado brotli. A compactação de arquivos de saída grandes pode ser lenta, portanto, desabilitar isso pode aumentar o desempenho de compilação para projetos grandes.

### build.chunkSizeWarningLimit

- **Type:** `number`
- **Default:** `500`

  Limite para avisos de tamanho de bloco (em kbs).

### build.watch

- **Type:** [`WatcherOptions`](https://rollupjs.org/guide/en/#watch-options)`| null`
- **Default:** `null`

  Defina como `{}` para habilitar o watcher de rollup. Isso é usado principalmente em casos que envolvem plugins somente de compilação ou processos de integração.

## Dep Optimization Options

- **Relacionado:** [Pré-empacotamento de dependência](/guide/dep-pre-bundling)

### optimizeDeps.entries

- **Type:** `string | string[]`

  Por padrão, o Vite rastreará seu index.html para detectar dependências que precisam ser pré-agrupadas. Se build.rollupOptions.input for especificado, Vite rastreará esses pontos de entrada.

  Se nenhum deles atender às suas necessidades, você pode especificar entradas personalizadas usando esta opção - o valor deve ser um [padrão fast-glob](https://github.com/mrmlnc/fast-glob#basic-syntax) ou um array de padrões que são relativos à raiz do projeto vite. Isso substituirá a inferência de entradas padrão.

### optimizeDeps.exclude

- **Type:** `string[]`

  Dependências a serem excluídas do pré-empacotamento.

  :::warning CommonJS
  As dependências do CommonJS não devem ser excluídas da otimização. Se uma dependência ESM tiver uma dependência CommonJS aninhada, ela também não deverá ser excluída.
  :::

### optimizeDeps.include

- **Type:** `string[]`

  Por padrão, os pacotes vinculados que não estão dentro de `node_modules` não são pré-empacotados. Use esta opção para forçar um pacote vinculado a ser pré-empacotado.

### optimizeDeps.keepNames

- **Type:** `boolean`
- **Default:** `false`

  O bundler às vezes precisa renomear símbolos para evitar colisões.
  Defina como `true` para manter a propriedade `name` em funções e classes.
  Veja [`keepNames`](https://esbuild.github.io/api/#keep-names).

## SSR Options

:::warning Experimental
As opções de SSR podem ser ajustadas em versões menores.
:::

- **Relacionado:** [SSR Dependências Externas](/guide/ssr#ssr-dependencias-externas)

### ssr.external

- **Type:** `string[]`

  Força a externalização de dependências para SSR.

### ssr.noExternal

- **Type:** `string | RegExp | (string | RegExp)[]`

  Impedir que dependências listadas sejam externalizadas para SSR.

### ssr.target

- **Type:** `'node' | 'webworker'`
- **Default:** `node`

  Alvo de compilação para o servidor SSR.
