# Migração da v1

## Mudança das opções de configuração

- As seguintes opções foram removidas e devem ser implementadas por meio de [plugins](./api-plugin):

  - `resolvers`
  - `transforms`
  - `indexHtmlTransforms`

- `jsx` e `enableEsbuild` foram removidos; Use o novo [`esbuild`](/config/#esbuild) ao invés.

- [CSS related options](/config/#css-modules) agora estão aninhadas em `css`.

- Todas [build-specific options](/config/#build-options) agora estão aninhadas em `build`.

  - `rollupInputOptions` e `rollupOutputOptions` são substituídos por [`build.rollupOptions`](/config/#build-rollupoptions).
  - `esbuildTarget` agora é  [`build.target`](/config/#build-target).
  - `emitManifest` agora é  [`build.manifest`](/config/#build-manifest).
  - As seguintes opções de compilação foram removidas, uma vez que podem ser obtidas por meio de plugin hooks ou outras opções:
    - `entry`
    - `rollupDedupe`
    - `emitAssets`
    - `emitIndex`
    - `shouldPreload`
    - `configureBuild`

- Todas [server-specific options](/config/#server-options) agora estão aninhadas em
  `server`.

  - `hostname` agora é [`server.host`](/config/#server-host).
  - `httpsOptions` foi removido. [`server.https`](/config/#server-https) pode aceitar diretamente o objeto de opções.
  - `chokidarWatchOptions` agora é [`server.watch`](/config/#server-watch).

- [`assetsInclude`](/config/#assetsInclude) agora espera `string | RegExp | (string | RegExp)[]` em vez de uma função.

- Todas as opções específicas do Vue são removidas; Passe as opções para o plug-in do Vue.

## Mudança de comportamento de alias

[`alias`](/config/#alias) agora está sendo passado para `@rollup/plugin-alias` e não requer mais barras de início / fim. O comportamento agora é uma substituição direta, portanto, a chave de alias de diretório no estilo 1.0 deve remover a barra final:

```diff
- alias: { '/@foo/': path.resolve(__dirname, 'some-special-dir') }
+ alias: { '/@foo': path.resolve(__dirname, 'some-special-dir') }
```

Como alternativa, você pode usar o `[{ find: RegExp, replacement: string }]` formato de opção para um controle mais preciso.

## Suporte Vue 

O core do Vite 2.0 agora é independente de estrutura. O suporte Vue agora é fornecido via [`@vitejs/plugin-vue`](https://github.com/vitejs/vite/tree/main/packages/plugin-vue). Basta instalá-lo e adicioná-lo na configuração do Vite:

```js
import vue from '@vitejs/plugin-vue'
import { defineConfig } from 'vite'

export default defineConfig({
  plugins: [vue()]
})
```

### Transformações de blocos personalizados

Um plug-in personalizado pode ser usado para transformar blocos personalizados do Vue como o mostrado abaixo:

```ts
// vite.config.js
import vue from '@vitejs/plugin-vue'
import { defineConfig } from 'vite'

const vueI18nPlugin = {
  name: 'vue-i18n',
  transform(code, id) {
    if (!/vue&type=i18n/.test(id)) {
      return
    }
    if (/\.ya?ml$/.test(id)) {
      code = JSON.stringify(require('js-yaml').safeLoad(code.trim()))
    }
    return `export default Comp => {
      Comp.i18n = ${code}
    }`
  }
}

export default defineConfig({
  plugins: [vue(), vueI18nPlugin]
})
```

## Suporte React 

O suporte React Fast Refresh agora é fornecido via [`@vitejs/plugin-react-refresh`](https://github.com/vitejs/vite/tree/main/packages/plugin-react-refresh).

## Alteração de API HMR

`import.meta.hot.acceptDeps()` foram descontinuados. [`import.meta.hot.accept()`](./api-hmr#hot-accept-deps-cb) agora pode aceitar dependências únicas ou múltiplas.

## Mudança do formato do Manifest

O manifesto de construção agora usa o seguinte formato:

```json
{
  "index.js": {
    "file": "assets/index.acaf2b48.js",
    "imports": [...]
  },
  "index.css": {
    "file": "assets/index.7b7dbd85.css"
  }
  "asset.png": {
    "file": "assets/asset.0ab0f9cd.png"
  }
}
```

Para os JS chunks de entrada, ele também lista seus chunks importados que podem ser usados para renderizar diretivas de pré-carregamento.

## Para Autores de Plugin

O Vite 2 usa uma interface de plug-in completamente redesenhada que estende os plug-ins Rollup. Leia o novo [Plugin Development Guide](./api-plugin).

Algumas dicas gerais sobre a migração de um plug-in v1 para v2:

- `resolvers` -> use o [`resolveId`](https://rollupjs.org/guide/en/#resolveid) hook
- `transforms` -> use o [`transform`](https://rollupjs.org/guide/en/#transform) hook
- `indexHtmlTransforms` -> use o [`transformIndexHtml`](./api-plugin#transformindexhtml) hook
- Serving virtual files -> use [`resolveId`](https://rollupjs.org/guide/en/#resolveid) + [`load`](https://rollupjs.org/guide/en/#load) hooks
- Adicionando `alias`, `define` ou outras opções de configuração -> use o [`config`](./api-plugin#config) hook

Uma vez que a maior parte da lógica deve ser feita por meio de plugin hooks em vez de middlewares, a necessidade de middlewares é bastante reduzida. O aplicativo de servidor interno agora é uma boa e velha instância de [connect](https://github.com/senchalabs/connect) en vez de Koa.
