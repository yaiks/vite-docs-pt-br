# Renderização no Servidor (SSR)

:::warning Experimental
Suporte à SSR ainda é experimental e você pode encontrar bugs e casos não suportados. Use ciente dos riscos.
:::

:::tip Nota
SSR se refere especificamente à frameworks de front-end (por exemplo React, Preact, Vue e Svelte) que suportam o uso da mesma aplicação no ambiente Node.js, pré renderizando para HTML e finalmente _hidratando_ no cliente. Se você está buscando por uma integração com frameworks tradicionais de servidor, confira o [Guia de Integração Backend](./backend-integration).

O guia a seguir também assume experiência prévia com SSR no seu frameworkde escolha, e focará apenas nos detalhes específicos da integração com Vite.
:::

:::warning API de Baixo Nível
Essa é uma API de baixo nível para autores de bilbiotecas e frameworks. Se seu objetivo é criar uma aplicação, confira as ferramentas e plugins de SSR de alto nível em [Seção Awesome Vite SSR](https://github.com/vitejs/awesome-vite#ssr). Dito isso, muitas aplicações são criadas com sucesso usando a API de baixo nível nativa do Vite.
:::

:::tip Ajuda
Se você tem dúvidas, a comunidade é geralmente prestativa no [Canal do Discord do Vite #ssr](https://discord.gg/PkbxgzPhJv).
:::

## Projetos de Exemplo

Vite oferece suporte nativo à renderização no servidor (SSR). O _playground_ do Vite contém exemplos de configurações com SSR para Vue 3 e React, que podem ser usados como referências para este guia:

- [Vue 3](https://github.com/vitejs/vite/tree/main/packages/playground/ssr-vue)
- [React](https://github.com/vitejs/vite/tree/main/packages/playground/ssr-react)

## Estrutura da Fonte

Uma aplicação típica de SSR terá a seguinte estrutura de arquivos:

```
- index.html
- src/
  - main.js          # exporta código da aplicação env-agnostic (universal)
  - entry-client.js  # monta o app ao elemento DOM
  - entry-server.js  # renderiza o app usando a API SSR do framework
```

O arquivo `index.html` precisa referenciar `entry-client.js` e incluir um marcador de posição onde a marcação deve ser injetada:

```html
<div id="app"><!--ssr-outlet--></div>
<script type="module" src="/src/entry-client.js"></script>
```

Você pode usar qualquer marcado de posição ao invés de `<!--ssr-outlet-->`, contanto que possa ser precisamente substituido.

## Lógica Condicional

Se você precisa performar alguma lógica condicional baseada em SRR _versus_ cliente, você pode usar:

```js
if (import.meta.env.SSR) {
  // ... lógica apenas do servidor
}
```

Isso será estaticamente substituido durante o build então irá permitir _tree-shaking_ de partes não usadas.

## Configurando o Servidor de Desenvolvimento

Ao criar uma aplicação SSR, você provavelmente quer ter controle total do seu servidor e desacomplar Vite do ambiente de produção. Portanto é recomendado usar Vite em modo _middleware_. Aqui está um exemplo com [express](https://expressjs.com/):

**server.js**

```js{17-19}
const fs = require('fs')
const path = require('path')
const express = require('express')
const { createServer: createViteServer } = require('vite')

async function createServer() {
  const app = express()

  // Create vite server in middleware mode. This disables Vite's own HTML
  // serving logic and let the parent server take control.
  //
  // If you want to use Vite's own HTML serving logic (using Vite as
  // a development middleware), using 'html' instead.
  const vite = await createViteServer({
    server: { middlewareMode: 'ssr' }
  })
  // use vite's connect instance as middleware
  app.use(vite.middlewares)

  app.use('*', async (req, res) => {
    // serve index.html - we will tackle this next
  })

  app.listen(3000)
}

createServer()
```

Aqui `vite` é uma instância de [ViteDevServer](./api-javascript#vitedevserver). `vite.middlewares` é uma instância de [Connect](https://github.com/senchalabs/connect) que pode ser usado como um _middleware_ em qualquer framework compatível com Node.js.

O próximo passo é implementar o gerenciador `*` para servir HTML renderizado no servidor:

```js
app.use('*', async (req, res) => {
  const url = req.originalUrl

  try {
    // 1. Ler o index.html
    let template = fs.readFileSync(
      path.resolve(__dirname, 'index.html'),
      'utf-8'
    )

    // 2. Aplica o transformador de HTML do Vite. Isso injeta o cliente HMR do Vite e
    //    também aplica o transformador de HTML dos plugins do Vite, ex: _global preambles_
    //    do pacote @vitejs/plugin-react-refresh
    template = await vite.transformIndexHtml(url, template)

    // 3. Carrega a entrada do servidor. vite.ssrLoadModule automaticamente transforma
    //    seu código fonte ESM para ser usado no Node.js! Não tem necessidade de empacotadores
    //    e oferece invalidação similar ao HMR.
    const { render } = await vite.ssrLoadModule('/src/entry-server.js')

    // 4. renderiza do HTML do app. Isso assume que as funções `render` exportado
    //    do entry-server.js chamam as APIs apropriadas de SSR dos frameworks
    //    e.g. ReactDOMServer.renderToString()
    const appHtml = await render(url)

    // 5. Injeta o HTML renderizado do app no template.
    const html = template.replace(`<!--ssr-outlet-->`, appHtml)

    // 6. Envia o HTML de volta
    res.status(200).set({ 'Content-Type': 'text/html' }).end(html)
  } catch (e) {
    // Se um erro é pego, deixa o Vite ajustar o _stacktrace_ para mapear de volta
    // ao código fonte.
    vite.ssrFixStacktrace(e)
    console.error(e)
    res.status(500).end(e.message)
  }
})
```

O script `dev` em `package.json` também deve ser alterado para usar o script de servidor:

```diff
  "scripts": {
-   "dev": "vite"
+   "dev": "node server"
  }
```

## Construindo para Produção

Para subir um projeto SSR em produção, nós precisamos:

1. Produzir um build no cliente normalmente;
2. Produzir um build SSR, que pode ser diretamente carregado via `require()` para que não haja necessidade de usar o `ssrLoadModule` do Vite;

Nosso script em `package.json` ficará assim:

```json
{
  "scripts": {
    "dev": "node server",
    "build:client": "vite build --outDir dist/client",
    "build:server": "vite build --outDir dist/server --ssr src/entry-server.js "
  }
}
```

Note que a flag `--ssr` que indica o _build_ SSR. Ela também deve especificar a entrada do SSR.

Então, no `server.js` nós precisamos adicionar algumas lógicas específicas de produção checando o `process.env.'NODE_ENV'`:

- Ao invés de ler a raíz `index.html`, use o `dist/client/index.html` como template, já que contém os links corretos para arquivos do build no cliente.

- Ao invés de `await vite.ssrLoadModule('/src/entry-server.js')`, use `require('./dist/server/entry-server.js')` (esse arquivo é o resultado do build SSR).

- Mova a criação e todos os usos do servidor de desenvolvimento `vite` para partes condicionais apenas ao ambiente de dev, então adicione _middlewares_ para servir arquivos estáticos de `dist/client`.

Veja [Vue](https://github.com/vitejs/vite/tree/main/packages/playground/ssr-vue) e [React](https://github.com/vitejs/vite/tree/main/packages/playground/ssr-react) demos para configurações funcionais.

## Generating Preload Directives

`vite build` supports the `--ssrManifest` flag which will generate `ssr-manifest.json` in build output directory:

```diff
- "build:client": "vite build --outDir dist/client",
+ "build:client": "vite build --outDir dist/client --ssrManifest",
```

The above script will now generate `dist/client/ssr-manifest.json` for the client build (Yes, the SSR manifest is generated from the client build because we want to map module IDs to client files). The manifest contains mappings of module IDs to their associated chunks and asset files.

To leverage the manifest, frameworks need to provide a way to collect the module IDs of the components that were used during a server render call.

`@vitejs/plugin-vue` supports this out of the box and automatically registers used component module IDs on to the associated Vue SSR context:

```js
// src/entry-server.js
const ctx = {}
const html = await vueServerRenderer.renderToString(app, ctx)
// ctx.modules is now a Set of module IDs that were used during the render
```

In the production branch of `server.js` we need to read and pass the manifest to the `render` function exported by `src/entry-server.js`. This would provide us with enough information to render preload directives for files used by async routes! See [demo source](https://github.com/vitejs/vite/blob/main/packages/playground/ssr-vue/src/entry-server.js) for full example.

## Pre-Rendering / SSG

If the routes and the data needed for certain routes are known ahead of time, we can pre-render these routes into static HTML using the same logic as production SSR. This can also be considered a form of Static-Site Generation (SSG). See [demo pre-render script](https://github.com/vitejs/vite/blob/main/packages/playground/ssr-vue/prerender.js) for working example.

## SSR Externals

Many dependencies ship both ESM and CommonJS files. When running SSR, a dependency that provides CommonJS builds can be "externalized" from Vite's SSR transform / module system to speed up both dev and build. For example, instead of pulling in the pre-bundled ESM version of React and then transforming it back to be Node.js-compatible, it is more efficient to simply `require('react')` instead. It also greatly improves the speed of the SSR bundle build.

Vite performs automated SSR externalization based on the following heuristics:

- If a dependency's resolved ESM entry point and its default Node entry point are different, its default Node entry is probably a CommonJS build that can be externalized. For example, `vue` will be automatically externalized because it ships both ESM and CommonJS builds.

- Otherwise, Vite will check whether the package's entry point contains valid ESM syntax - if not, the package is likely CommonJS and will be externalized. As an example, `react-dom` will be automatically externalized because it only specifies a single entry which is in CommonJS format.

If this heuristics leads to errors, you can manually adjust SSR externals using `ssr.external` and `ssr.noExternal` config options.

In the future, this heuristics will likely improve to detect if the project has `type: "module"` enabled, so that Vite can also externalize dependencies that ship Node-compatible ESM builds by importing them via dynamic `import()` during SSR.

:::warning Working with Aliases
If you have configured aliases that redirects one package to another, you may want to alias the actual `node_modules` packages instead to make it work for SSR externalized dependencies. Both [Yarn](https://classic.yarnpkg.com/en/docs/cli/add/#toc-yarn-add-alias) and [pnpm](https://pnpm.js.org/en/aliases) support aliasing via the `npm:` prefix.
:::

## SSR-specific Plugin Logic

Some frameworks such as Vue or Svelte compiles components into different formats based on client vs. SSR. To support conditional transforms, Vite passes an additional `ssr` argument to the following plugin hooks:

- `resolveId`
- `load`
- `transform`

**Example:**

```js
export function mySSRPlugin() {
  return {
    name: 'my-ssr',
    transform(code, id, ssr) {
      if (ssr) {
        // perform ssr-specific transform...
      }
    }
  }
}
```

## SSR Target

The default target for the SSR build is a node environment, but you can also run the server in a Web Worker. Packages entry resolution is different for each platform. You can configure the target to be Web Worker using the `ssr.target` set to `'webworker'`.
