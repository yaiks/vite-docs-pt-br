# Usando Plugins

Vite pode ser complementado com plugins, que são baseados na interface de plugin bem desenhada do Rollup com algumas opções extra específicas do Vite. Isso significa que usuários do Vite podem aproveitar a maturidade do ecossistema de plugins do Rollup, ao mesmo tempo que podem extender as funcionalidades de servidor de desenvolvimento e SSR se necessário.

## Adicionando um Plugin

Para usar um plugin, é necessário adicioná-lo ao `devDependencies` do projeto e incluir na lista de `plugins` no arquivo de configuração `vite.config.js`. Por exemplo, para oferecer suporte à browsers legados, o plugin oficial [@vitejs/plugin-legacy](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy) pode ser usado:

```
$ npm i -D @vitejs/plugin-legacy
```

```js
// vite.config.js
import legacy from '@vitejs/plugin-legacy'
import { defineConfig } from 'vite'

export default defineConfig({
  plugins: [
    legacy({
      targets: ['defaults', 'not IE 11']
    })
  ]
})
```

`plugins` também aceita pré configurações incluindo múltiplos plugins como elemento único. Isso é útil para funcionalidades complexas (como integrações de framework) que são implementados usando diversos plugins. A lista será normalizada internamente.

Plugins com valor _falsy_ serão ignorados, o que pode ser útil para ativar ou desativar plugins.

## Descobrindo Plugins

:::tip NOTA
Vite busca oferecer suporte para padrões comuns de desenvolvimento web. Antes de procurar um plugin para Vite ou compatível com Rollup, dê uma olhada no [Guia de Funcionalidades](../guide/features.md). Muitos casos onde um plugins seria necessário num projeto Rollup já são cobertos por padrão no Vite.
:::

Dê uma olhada na [sessão de Plugins](../plugins/) para informações sobre plugins oficiais. Plugins da comunidade são listados em [awesome-vite](https://github.com/vitejs/awesome-vite#plugins). Para plugins compatíveis com Rollup, veja [Vite Rollup Plugins](https://vite-rollup-plugins.patak.dev) para uma lista de plugins oficiais compatíveis com Rollup e com instruções de uso, ou o [Sessão de Compatibilidade com Plugins Rollup](../guide/api-plugin#rollup-plugin-compatibility) no caso de não estar listado lá.

Você também pode encontrar plugins que seguem as [convensões recomendadas](./api-plugin.md#conventions) usando a [busca npm para vite-plugin](https://www.npmjs.com/search?q=vite-plugin&ranking=popularity) para plugins Vite ou a [busca npm for rollup-plugin](https://www.npmjs.com/search?q=rollup-plugin&ranking=popularity) para plugins Rollup.

## Forçando a Ordenação dos Plugins

Para compatibilidade com alguns plugins de Rollup, talvez seja necessário reforçar a ordem do plugin ou apenas aplicá-lo em tempo de build. isso deve ser um detalhe de implementação para plugins Vite. Você pode forçar a posição de um plugin usando o modificador `enforce`:

- `pre`: invoca o plugin antes dos plugins principais do Vite
- default: invoca o plugin depois dos plugins principais do Vite
- `post`: invoca o plufin depois dos plugins de build do Vite

```js
// vite.config.js
import image from '@rollup/plugin-image'
import { defineConfig } from 'vite'

export default defineConfig({
  plugins: [
    {
      ...image(),
      enforce: 'pre'
    }
  ]
})
```

Veja o [Guia de API dos Plugins](./api-plugin.md#plugin-ordering) para informações detalhadas, procure pela propriedade `enforce` e instruções de uso para plugins popularese na lista de compatibilidade [Vite Rollup Plugins](https://vite-rollup-plugins.patak.dev).

## Aplicação Condicional

Por padrão, plugins são invocados para servidor e build. Em casos onde um plugin precisa ser aplicado condicionalmente durante servidor ou build, use a propriedade `apply` para apenas invocá-lo durante `'build'` ou `'serve'`:

```js
// vite.config.js
import typescript2 from 'rollup-plugin-typescript2'
import { defineConfig } from 'vite'

export default defineConfig({
  plugins: [
    {
      ...typescript2(),
      apply: 'build'
    }
  ]
})
```

## Criando Plugins

Veja o [Guia de API dos Plugins](./api-plugin.md) para documentação de criação de plugins.
