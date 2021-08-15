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

## Gerando Diretivas Pré-Carregadas

`vite build` suporta a opção `--ssrManifest` que irá gerar o arquivo `ssr-manifest.json` no diretório resultante do _build_:

```diff
- "build:client": "vite build --outDir dist/client",
+ "build:client": "vite build --outDir dist/client --ssrManifest",
```

O script acima irá gerar `dist/client/ssr-manifest.json` para o _build_ do cliente (Sim, o manifest SSR é gerado a partir do build do cliente pois queremos mapear o ID do módulo com os arquivos do cliente). O manifesto contém o mapeamento de IDs de módulos com seus arquivos e pedaços associados.

Para se beneficiar do manifesto, frameworks precisam prover uma maneira de coletar os IDs dos módulos dos componentes que foram usados durante uma chamada no servidor.

`@vitejs/plugin-vue` suporta essa funcionalidade por padrão e automaticamente registra IDs de módulos dos componentes usados ao contexto correto do Vue SSR:

```js
// src/entry-server.js
const ctx = {}
const html = await vueServerRenderer.renderToString(app, ctx)
// ctx.modules agora é um Set de IDs de módulos que foram usados durante a renderização
```

No _branch_ de produção de `server.js`, nós precisamos ler e passar o manifesto para a função `render` exportada por `src/entry-server.js`.
Isso irá providenciar informações o suficiente para renderizar uma diretiva pré carregada para arquivos usados por rotas assíncronas! Veja [demo](https://github.com/vitejs/vite/blob/main/packages/playground/ssr-vue/src/entry-server.js) para um exemplo completo.

## Pré Renderização / SSG

Se as rotas e os dados necessários para certas rotas são conhecidas antecipadamente, nós podemos pré renderizar essa rotas em HTML estático usando a mesma lógica de SSR para produção. Isso também pode ser considerado uma forma de Geração de Sites Estáticos (SSG). Veja [demo de script de pré renderização](https://github.com/vitejs/vite/blob/main/packages/playground/ssr-vue/prerender.js) para um exemplo funcional.

## SSR Dependências Externas

Muitas dependências usam tanto arquivos ESM como CommonJS. Quando rodar SSR, a dependência que oferece um _build_ em CommonJS pode ser "externalizado" do sistema de transformação / módulos de SSR do Vite, para acelerar tanto dev quanto build. Por exemplo, ao invés de baixar a versão ESM pré empacotada de React e então transformá-la de volta em código compatível com NodeJS, é mais eficiente simplesmente `require('react')`. Isso também aumenta a velocidade do _build_ do pacote SSR.

Vite performa a externalização de SSR automatizada baseada nas seguintes heurísticas:

- Se um ponto de entrada de uma dependência ESM resolvida e seu ponto de entrada padrão em Node são diferentes, sua entrada padrão de Node é provavelmente um _build_ CommonJS que pode ser externalizado. Por exemplo, `vue` irá automaticamente externalizar pois gera tanto _builds_ ESM quanto CommonJS.

- Por outro lado, Vite irá checar caso o ponto de entrada do pacote contém sintaxe válida de ESM - se não, o pacote é provavelmente CommonJS e irá ser externalizado. Como exemplo, `react-dom` será automaticamente externalizado pois apenas especifica uma única entrada que está em formato CommonJS.

Se estas heurísticas levam à erros, você pode manualmente ajustar essas configurações usando `ssr.external` e `ssr.noExternal`nas opções de configuração.

No futuro, essas heurísticas irão ser aprimoradas para detectar se um projeto possui `type: module` habilitado, para que Vite possa externalizar dependêncuas que oferecem _builds_ ESM compatíveis com Node, importando-os durante SSR com `import()` dinâmico.

:::warning Usando _aliases_
Se você configurou _aliases_ que redirecionam um pacote para outro, talvez você queira _alias_ os pacotes atuais do `node_modules` ao invés de fazê-los funcionar para dependêncuas externalizadas de SSR. Tanto [Yarn](https://classic.yarnpkg.com/en/docs/cli/add/#toc-yarn-add-alias) e [pnpm](https://pnpm.js.org/en/aliases) suportam _aliasing_ através do prefixo `npm:`.
:::

## Lógica de Plugin específica para SSR

Alguns frameworks como Vue ou Svelte compilam componentes em diferentes formatos baseados em cliente vs SSR. Para suportam transformações condicionais, Vite passa um argumento adicional `ssr` para o seguinte _hook_ de plugin:

- `resolveId`
- `load`
- `transform`

**Exemplo:**

```js
export function mySSRPlugin() {
  return {
    name: 'my-ssr',
    transform(code, id, ssr) {
      if (ssr) {
        // performa transformações específicas de SSR...
      }
    }
  }
}
```

## Alvo SSR

O alvo padrão para _build_ SSR é um ambiente de node, mas você também pode rodar o servidor em um Web Worker. Resoluções de entrada de pacotes são diferentes para cada plataforma. Você pode configurar o alvo para Web Worker usando o `ssr.target` setado para `'webworker'`.
