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

Arquivos JSON podem ser diretamente importados - importações nomeadas também são suportadas:

```js
// importar o objeto inteiro
import json from './example.json'
// importar um campo específico como importação nomeada - ajuda o _treeshaking_!
import { field } from './example.json'
```
## Importação de Glob

Vite suporta a importação de múltiplos módulos do sistema de arquivos através da função especial `import.meta.glob`:

```js
const modules = import.meta.glob('./dir/*.js')
```

O trecho acima será transformado no seguinte:

```js
// código produzido pelo vite
const modules = {
  './dir/foo.js': () => import('./dir/foo.js'),
  './dir/bar.js': () => import('./dir/bar.js')
}
```

Você pode iterar entre as chaves do objeto `modules` para acessar os módulos correspondentes:

```js
for (const path in modules) {
  modules[path]().then((mod) => {
    console.log(path, mod)
  })
}
```

Arquivos correspondentes são carregados com `lazy load` por padrão através de importações dinâmicas e serão separadas em diferentes `chunks` durante o _build_. Se você prefere importar todos os módulos diretamente (ex: dependendo da aplicação de side-effects desses módulos), você pode usar `import.meta.globEager`.

```js
const modules = import.meta.globEager('./dir/*.js')
```

O trecho acima será transformado em:

```js
// código produzido pelo vite
import * as __glob__0_0 from './dir/foo.js'
import * as __glob__0_1 from './dir/bar.js'
const modules = {
  './dir/foo.js': __glob__0_0,
  './dir/bar.js': __glob__0_1
}
```

Note que:

- Essa é uma funcionalidade específica do Vite e não um padrão da Web ou _ES_ (EcmaScript).
- O pattern do glob é tratado como especificadores de importação: eles devem ser ou relativos (começando com `./`) ou absolutos (começando com `/`, resolvendo relativos à raíz do projeto)
- A correspondência de glob é feita via `fast-glob` - veja a documentação para [patterns de glob suportados](https://github.com/mrmlnc/fast-glob#pattern-syntax).

## WebAssembly

Arquivos `.wasm` pré compilados podem ser importados diretamente - a exportação padrão será uma função inicializadora que retorna uma _Promise_ de objetos exportados da instância wasm:

```js
import init from './example.wasm'

init().then((exports) => {
  exports.test()
})
```
A função `init` também pode ter acesso ao objeto `imports` que é passado ao `WebAssembly.instantiate` como segundo argumento:

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

No _build_ de produção, arquivos `.wasm` menores que `assetInlineLimit` serão usados _inline_ como strings base64. Caso contrário, eles serão copiados para o diretório _dist_ como um arquivo e requisitados sob demanda.

## Web Workers

Um script web worker pode ser importado diretamente ao acrescentar `?worker` ou `?sharedworker` à requisição de importação. A exportação padrão será o construtor do worker customizado:

```js
import MyWorker from './worker?worker'

const worker = new MyWorker()
```

O script _worker_ também pode usar declarações de `import` ao invés de `importScripts()` - note que durante o ambiente de desenvolvimento isso fica à cargo do suporte nativo do browser e atualmente só funciona no Chrome, mas para _builds_ de produção é compilado normalmente.

Por padrão, o script _worker_ será emitido como um pedaço separado no _build_ de produção. Se você deseja usar o worker como uma string base64 _inline_, adicione a _query_ `inline`:

```js
import MyWorker from './worker?worker&inline'
```

## Otimizações de Build

> Funcionalidades listadas abaixo são automaticamente aplicadas como parte do processo de build e não há necessidade de configurações explícitas a menos que você queira desabilitá-las.

### Separação de código CSS (_Code Splitting_)

Vite automaticamente extrai o CSS usado por módulos em um pedaço assíncrono e gera um arquivo separado para ele. O arquivo de CSS é automaticamente carregado através da tag `<link>` quando o pedaço assíncrono é requisitado, e este pedaço assíncrono só será interpretado após o CSS ser carregado, a fim de evitar [FOUC](https://en.wikipedia.org/wiki/Flash_of_unstyled_content#:~:text=A%20flash%20of%20unstyled%20content,before%20all%20information%20is%20retrieved.).

Se você prefere ter todo o CSS extraído em um único arquivo, você pode disabilitar a separação de código CSS ao definir [`build.cssCodeSplit`](/config/#build-csscodesplit) para `false`.

### Geração de Pre Carregamento de Diretivas

Vite automaticamente gera diretivas `<link rel="modulepreload">` para pedaços definidos na entrada e suas importações diretas no HTML gerado.

### Otimização do Carregamento de Pedaços Assíncronos

Em aplicações reais, Rollup frequentemente gera pedaços _"common"_ - código que é compartilhado entre dois ou mais pedaços. Combinado com importações dinâmicas, é muito comum presenciar os seguintes cenários:

![graph](/images/graph.png)

No cenário não otimizado, quando o pedaço assíncrono `A` é importado, o browser precisará requisitar e analisar `A` antes de descobrir que também precisa do pedaço `C`. Isso resulta em uma requisição de rede extra:

```
Entry ---> A ---> C
```

Vite automaticamente reescreve chamadas de importação dinâmica de _code-split_ com um passo de pré carregamento, para que quando `A` for requisitado, `C` é requisitado **em paralelo**:

```
Entry ---> (A + C)
```

É possível que `C` tenha outras importações, o que resultará em outras requisições extra de rede em um cenário não otimizado. A otimização do Vite irá reconhecer todas as importações diretas para eliminar completamente as requisições extra, não importando a profundidade de importações.
