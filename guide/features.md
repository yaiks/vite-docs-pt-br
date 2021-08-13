# Funcionalidades

Num contexto mais simples, desenvolver usando Vite não é muito diferente de usar um servidor de arquivos estático. Entretanto, Vite oferece muitas melhorias sobre a importação de módulos _ESM_ nativo para dar suporte à diversas funcionalidades que geralmente são vistas em configurações de empacotadores.

## Resolvendo dependências do NPM e _Pré-Bundling_

Importações nativas de ES (ECMAScript) não suportam o seguinte formato de importação:

```js
import { someMethod } from 'my-dep'
```

O código acima irá lançar um erro no browser. Vite irá detectar esse tipo de importação em todos os arquivos da sua aplicação e performar o seguinte:

1. [Pré empacotar](./dep-pre-bundling) esses módulos para melhorar a velocidade de carregamento da página e converter _CommonJS_ / módulos _UMD_ para _ESM_. Esse passo de pré empacotar é feito com [esbuild](http://esbuild.github.io/) e torna o start inicial de Vite significantemente mais rápido do que qualquer empacotador baseado em Javascript.

2. Reescreve as importações para URLs válidas tipo `/node_modules/.vite/my-dep.js?v=f3sf2ebd`, para que os browsers possam importá-los corretamente.

**Dependencias são fortemente cacheadas**

Vite cacheia requisições de dependências via cabeçalhos HTTP, então se você deseja editar/debugar localmente uma dependência, siga os [seguintes](./dep-pre-bundling#browser-cache) passos.

## _Hot Module Replacement_

Vite oferece uma [API de HMR](./api-hmr) sobre _ESM_ nativo. Frameworks com capacidades de HMR podem tirar vantagem da API para prover atualizações instantâneas e precisas, sem necessidade de recarregar a página ou acabar com o estado local da aplicação. Vite oferece integrações de _HMR_ para [Vue Single File Components](https://github.com/vitejs/vite/tree/main/packages/plugin-vue) e [React Fast Refresh](https://github.com/vitejs/vite/tree/main/packages/plugin-react-refresh). Há também integrações oficiais para Preact através do [@prefresh/vite](https://github.com/JoviDeCroock/prefresh/tree/main/packages/vite).

Note que você não precisa configurá-los manualmente - quando você [cria um app através do `create-vite`](./), o template selecionado já terá essas integrações pré-configuradas.

## TypeScript

Vite suporta importações de arquivos `.ts` por padrão.

Vite performa apenas transpilações em arquivos `.ts` e **NÃO** faz checagem de tipos. Ele assume que a checagem de tipos é papel da sua IDE e do processo de _build_ (você pode rodar `tsc --noEmit` no script de _build_ ou instalar `vue-tsc` e rodar `vue-tsc --noEmit` para checar seus arquivos `*.vue`).

Vite usa [esbuild](https://github.com/evanw/esbuild) para transpilar Typescript em Javascript, sendo que é 20~30x mais rápido que o `tsc` puro, e atualizações de _HMR_ podem refletir no browser em menos de 50ms.

Já que `esbuild` performa apenas transpilação sem informação de tipos, ele não suporta certas funcionalidades como `const enum` e importações implícitas de tipagem.
Você precisa definir `"isolatedModules": true` em seu `tsconfig.json` sobre `compilerOptions` para que o TS possa te avisar sobre funcionalidades que não executam com transpilação isolada.

### Tipagem de clientes

A tipagem padrão de Vite é para sua API em Node.js. Para interceptar o ambiente do lado do cliente em uma aplicação Vite, adicione um arquivo de declaração `d.ts`:

```typescript
/// <reference types="vite/client" />
```

Voc6e também pode adicionar `vite/client` ao `compilerOptions.types` em seu `tsconfig`:

```json
{
  "compilerOptions": {
    "types": ["vite/client"]
  }
}
```

Isso irá prover os seguintes tipos de _shims_:

- Importação de _assets_ (ex. importando um arquivo `.svg`)
- Tipagem para [variáveis de ambiente](./env-and-mode#env-variables) injetadas pelo Vite em `import.meta.env`
- Tipagem para [API HMR](./api-hmr) em `import.meta.hot`

## Vue

Vite oferece suporte de primeira classe para Vue:

- Suporte para Vue 3 SFC através de [@vitejs/plugin-vue](https://github.com/vitejs/vite/tree/main/packages/plugin-vue)
- Suporte para Vue 3 JSX através de [@vitejs/plugin-vue-jsx](https://github.com/vitejs/vite/tree/main/packages/plugin-vue-jsx)
- Suporte para Vue 2 através de [underfin/vite-plugin-vue2](https://github.com/underfin/vite-plugin-vue2)

## JSX

Arquivos `.jsx` e `.tsx` também são suportados por padrão. Transpilação de JSX também é feito através do [esbuild](https://esbuild.github.io), e o resultado padrão é o estilo de React 16. Suporte no esbuild para JSX no estilo de React 17 está sendo acompanhado [aqui](https://github.com/evanw/esbuild/issues/334).

Usuários de Vue devem usar o plugin oficial [@vitejs/plugin-vue-jsx](https://github.com/vitejs/vite/tree/main/packages/plugin-vue-jsx), que oferece funcionalidades específicas do Vue 3 como HMR, resolução de componente global, diretivas e _slots_.

Se você não está usando JSX com React ou Vue, `jsxFactory` e `jsxFragment` customizados podem ser configurados usando a [opção `esbuild`](/config/#esbuild). Por exemplo para Preact:

```js
// vite.config.js
export default defineConfig({
  esbuild: {
    jsxFactory: 'h',
    jsxFragment: 'Fragment'
  }
})
```

Mais detalhes na [documentação do esbuild](https://esbuild.github.io/content-types/#jsx).

Você pode injetar os _helpers_ de JSX usando `jsxInject` (que é uma opção exclusiva do Vite) para evitar importações manuais:

```js
// vite.config.js
export default defineConfig({
  esbuild: {
    jsxInject: `import React from 'react'`
  }
})
```

## CSS

Importar arquivos de `.css` irá injetar seu conteúdo na página via tag `<style>` com suporte para _HMR_. Você também pode acessar a string do CSS processado como o exportado padrão do módulo.

### `@import` Inlining and Rebasing

Vite é pré configurado para suportar CSS `@import` _inline_ através do `postcss-import`. Os diferentes _alias_ que o Vite pode atribuir ao `@import` também são respeitadas. Além disso, todas as referências ao CSS `url()`, mesmo se os arquivos importados estiverem em diretórios diferentes, são automaticamente interpretados para garantir assertividade.

As diferentes _alias_ de `@import` e interpretação de URL também são suportadas para arquivos SASS e LESS (veja [Pré-processadores de CSS](#css-pre-processors)).

### PostCSS

Se o projeto contém uma configuração válida de PostCSS (qualquer formato suportado por [postcss-load-config](https://github.com/postcss/postcss-load-config), ex: `postcss.config.js`), ele será automaticamente aplicado à todas as importações de CSS.

### CSS Modules

Todo arquivo de CSS com prefixo `.module.css` é considerado um [arquivo CSS modules](https://github.com/css-modules/css-modules). Importar esse arquivo irá retornar o objeto do módulo correspondente:

```css
/* example.module.css */
.red {
  color: red;
}
```

```js
import classes from './example.module.css'
document.getElementById('foo').className = classes.red
```
O comportamento do _CSS modules_ pode ser configurado via [opções de `css.modules`](/config/#css-modules).

Se `css.modules.localsConvention` for habilitada para receber `camelCase` (ex: `localsConvention: 'camelCaseOnly'`), você também pode usar importações nomeadas:

```js
// .apply-color -> applyColor
import { applyColor } from './example.module.css'
document.getElementById('foo').className = applyColor
```

### Pré-processadores de CSS

Como Vite apenas visa atingir browsers modernos, é recomendado usar variáveis de CSS nativo com plugins de PostCSS que implementam _CSSWG drafts_ (ex: [postcss-nesting](https://github.com/jonathantneal/postcss-nesting)) e CSS compatível com padrões futuros.

Dito isso, Vite também oferece suporte padrão para arquivos `.scss`, `.sass`, `.less`, `.styl` e `.stylus`. Não é necessário instalar plugins específicos para Vite, mas o pré-processador usado deve ser instalado:

```bash
# .scss and .sass
npm install -D sass

# .less
npm install -D less

# .styl and .stylus
npm install -D stylus
```

Se estiver usando _Vue SFC_, isso automaticamente habilita o uso de `<style lang="sass">` e etc.

Vite aprimora a resolução de `@import` para Sass e Less, para que os `alias` do Vite sejam respeitadas. Além disso, referências à `url()` relativas dentro de arquivos Sass/Less importados que estão em diretórios diferentes da raíz são automaticamente interpretados para garantir assertividade.

`@import` _alias_ e interpretação de url não são suportados por Stylus devido às suas restrições de API.

Você também pode usar CSS Modules combinado com pré-processadores ao colocar `.module` antes da extensão do arquivo, por exemplo `style.module.scss`.

## Arquivos estáticos

Importar um arquivo estático irá retornar sua URL pública resolvida no momento em que for servida:

```js
import imgUrl from './img.png'
document.getElementById('hero-img').src = imgUrl
```

_Queries_ especiais podem modificar como os arquivos são carregados:

```js
// Carregar os arquivos explicitamente como URL
import assetAsURL from './asset.js?url'
```

```js
// Carregar arquivos como strings
import assetAsString from './shader.glsl?raw'
```

```js
// Carregar web workers
import Worker from './worker.js?worker'
```

```js
// Web Workers inlined como strings base64 no momento do build
import InlineWorker from './worker.js?worker&inline'
```

Mais detalhes em [Lidando com Arquivos Estáticos](./assets).

## JSON

JSON files can be directly imported - named imports are also supported:

```js
// import the entire object
import json from './example.json'
// import a root field as named exports - helps with treeshaking!
import { field } from './example.json'
```

## Glob Import

Vite supports importing multiple modules from the file system via the special `import.meta.glob` function:

```js
const modules = import.meta.glob('./dir/*.js')
```

The above will be transformed into the following:

```js
// code produced by vite
const modules = {
  './dir/foo.js': () => import('./dir/foo.js'),
  './dir/bar.js': () => import('./dir/bar.js')
}
```

You can then iterate over the keys of the `modules` object to access the corresponding modules:

```js
for (const path in modules) {
  modules[path]().then((mod) => {
    console.log(path, mod)
  })
}
```

Matched files are by default lazy loaded via dynamic import and will be split into separate chunks during build. If you'd rather import all the modules directly (e.g. relying on side-effects in these modules to be applied first), you can use `import.meta.globEager` instead:

```js
const modules = import.meta.globEager('./dir/*.js')
```

The above will be transformed into the following:

```js
// code produced by vite
import * as __glob__0_0 from './dir/foo.js'
import * as __glob__0_1 from './dir/bar.js'
const modules = {
  './dir/foo.js': __glob__0_0,
  './dir/bar.js': __glob__0_1
}
```

Note that:

- This is a Vite-only feature and is not a web or ES standard.
- The glob patterns are treated like import specifiers: they must be either relative (start with `./`) or absolute (start with `/`, resolved relative to project root).
- The glob matching is done via `fast-glob` - check out its documentation for [supported glob patterns](https://github.com/mrmlnc/fast-glob#pattern-syntax).

## WebAssembly

Pre-compiled `.wasm` files can be directly imported - the default export will be an initialization function that returns a Promise of the exports object of the wasm instance:

```js
import init from './example.wasm'

init().then((exports) => {
  exports.test()
})
```

The init function can also take the `imports` object which is passed along to `WebAssembly.instantiate` as its second argument:

```js
init({
  imports: {
    someFunc: () => {
      /* ... */
    }
  }
}).then(() => {
  /* ... */
})
```

In the production build, `.wasm` files smaller than `assetInlineLimit` will be inlined as base64 strings. Otherwise, they will be copied to the dist directory as an asset and fetched on-demand.

## Web Workers

A web worker script can be directly imported by appending `?worker` or `?sharedworker` to the import request. The default export will be a custom worker constructor:

```js
import MyWorker from './worker?worker'

const worker = new MyWorker()
```

The worker script can also use `import` statements instead of `importScripts()` - note during dev this relies on browser native support and currently only works in Chrome, but for the production build it is compiled away.

By default, the worker script will be emitted as a separate chunk in the production build. If you wish to inline the worker as base64 strings, add the `inline` query:

```js
import MyWorker from './worker?worker&inline'
```

## Build Optimizations

> Features listed below are automatically applied as part of the build process and there is no need for explicit configuration unless you want to disable them.

### CSS Code Splitting

Vite automatically extracts the CSS used by modules in an async chunk and generates a separate file for it. The CSS file is automatically loaded via a `<link>` tag when the associated async chunk is loaded, and the async chunk is guaranteed to only be evaluated after the CSS is loaded to avoid [FOUC](https://en.wikipedia.org/wiki/Flash_of_unstyled_content#:~:text=A%20flash%20of%20unstyled%20content,before%20all%20information%20is%20retrieved.).

If you'd rather have all the CSS extracted into a single file, you can disable CSS code splitting by setting [`build.cssCodeSplit`](/config/#build-csscodesplit) to `false`.

### Preload Directives Generation

Vite automatically generates `<link rel="modulepreload">` directives for entry chunks and their direct imports in the built HTML.

### Async Chunk Loading Optimization

In real world applications, Rollup often generates "common" chunks - code that is shared between two or more other chunks. Combined with dynamic imports, it is quite common to have the following scenario:

![graph](/images/graph.png)

In the non-optimized scenarios, when async chunk `A` is imported, the browser will have to request and parse `A` before it can figure out that it also needs the common chunk `C`. This results in an extra network roundtrip:

```
Entry ---> A ---> C
```

Vite automatically rewrites code-split dynamic import calls with a preload step so that when `A` is requested, `C` is fetched **in parallel**:

```
Entry ---> (A + C)
```

It is possible for `C` to have further imports, which will result in even more roundtrips in the un-optimized scenario. Vite's optimization will trace all the direct imports to completely eliminate the roundtrips regardless of import depth.
