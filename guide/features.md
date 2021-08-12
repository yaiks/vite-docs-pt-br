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

Importing `.css` files will inject its content to the page via a `<style>` tag with HMR support. You can also retrieve the processed CSS as a string as the module's default export.

### `@import` Inlining and Rebasing

Vite is pre-configured to support CSS `@import` inlining via `postcss-import`. Vite aliases are also respected for CSS `@import`. In addition, all CSS `url()` references, even if the imported files are in different directories, are always automatically rebased to ensure correctness.

`@import` aliases and URL rebasing are also supported for Sass and Less files (see [CSS Pre-processors](#css-pre-processors)).

### PostCSS

If the project contains valid PostCSS config (any format supported by [postcss-load-config](https://github.com/postcss/postcss-load-config), e.g. `postcss.config.js`), it will be automatically applied to all imported CSS.

### CSS Modules

Any CSS file ending with `.module.css` is considered a [CSS modules file](https://github.com/css-modules/css-modules). Importing such a file will return the corresponding module object:

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

CSS modules behavior can be configured via the [`css.modules` option](/config/#css-modules).

If `css.modules.localsConvention` is set to enable camelCase locals (e.g. `localsConvention: 'camelCaseOnly'`), you can also use named imports:

```js
// .apply-color -> applyColor
import { applyColor } from './example.module.css'
document.getElementById('foo').className = applyColor
```

### CSS Pre-processors

Because Vite targets modern browsers only, it is recommended to use native CSS variables with PostCSS plugins that implement CSSWG drafts (e.g. [postcss-nesting](https://github.com/jonathantneal/postcss-nesting)) and author plain, future-standards-compliant CSS.

That said, Vite does provide built-in support for `.scss`, `.sass`, `.less`, `.styl` and `.stylus` files. There is no need to install Vite-specific plugins for them, but the corresponding pre-processor itself must be installed:

```bash
# .scss and .sass
npm install -D sass

# .less
npm install -D less

# .styl and .stylus
npm install -D stylus
```

If using Vue single file components, this also automatically enables `<style lang="sass">` et al.

Vite improves `@import` resolving for Sass and Less so that Vite aliases are also respected. In addition, relative `url()` references inside imported Sass/Less files that are in different directories from the root file are also automatically rebased to ensure correctness.

`@import` alias and url rebasing are not supported for Stylus due to its API constraints.

You can also use CSS modules combined with pre-processors by prepending `.module` to the file extension, for example `style.module.scss`.

## Static Assets

Importing a static asset will return the resolved public URL when it is served:

```js
import imgUrl from './img.png'
document.getElementById('hero-img').src = imgUrl
```

Special queries can modify how assets are loaded:

```js
// Explicitly load assets as URL
import assetAsURL from './asset.js?url'
```

```js
// Load assets as strings
import assetAsString from './shader.glsl?raw'
```

```js
// Load Web Workers
import Worker from './worker.js?worker'
```

```js
// Web Workers inlined as base64 strings at build time
import InlineWorker from './worker.js?worker&inline'
```

More details in [Static Asset Handling](./assets).

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
