# Static Asset Handling

- Relacionado: [Public Base Path](./build#public-base-path)
- Relacionado: [`assetsInclude` config option](/config/#assetsinclude)

## Improtando Asset como URL

Importando um asset estatico irá retornar uma URL publica:

```js
import imgUrl from './img.png'
document.getElementById('hero-img').src = imgUrl
```

Por exemplo, `imgUrl` será `/img.png` durante o desenvolvimento, e etnão `/assets/img.2d8efhg.png` na build de produção.

Esse comportamento é similar ao `file-loader` do webpack. A diferença é que a ordem de importação pode ser feita utilizando caminhos absolutos (baseados na razi do projeto durante desenvolvimento) ou caminhos relativos.

- `url()` referencia no CSS e é tratada da mesma maneira.

- Se usando o plugin do Vue, referencia a assets no template SFC são automaticamente convertidos em importações.

- Imagens comuns, midia e fontes são detectados como assets automaticamente. Você pode extender a lista interna usando a opção [`assetsInclude`](/config/#assetsinclude).

- Assets referenciados são incluidos como parte da construção do grafico assets, terão os nomes de arquivos passados no hash e podem ser processados em puglins para otimização.

- Assets menores em butes que a opção [`assetsInlineLimit`](/config/#build-assetsinlinelimit) serão colocados inline como URLs base64 data.

### Importações com URL Explicitas

Assets que não são incluidos na lista interna ou na `assetsInclude`, podem ser explicitamente importados por uma URL utilizando o sufixo `?url`. Isso é util, por exemplo, para importar [Houdini Paint Worklets](https://houdini.how/usage).

```js
import workletURL from 'extra-scalloped-border/worklet.js?url'
CSS.paintWorklet.addModule(workletURL)
```

### Importando Asset como String

Assets podem ser importados como strings utilizando o sufixo `?raw`.

```js
import shaderString from './shader.glsl?raw'
```

### Importando Script como um Worker

Scripts podem ser importados como web workers com o sufixo `?worker` or `?sharedworker`.

```js
// Separate chunk in the production build
import Worker from './shader.js?worker'
const worker = new Worker()
```

```js
// sharedworker
import SharedWorker from './shader.js?sharedworker'
const sharedWorker = new SharedWorker()
```

```js
// Inlined as base64 strings
import InlineWorker from './shader.js?worker&inline'
```

Veja [Web Worker section](./features.md#web-workers) para mais detalhes.

## O diretorio `public`

Se você tem assets que são:

- Nunca referenciados no codigo fonte (ex. `robots.txt`)
- Devem reter o mesmo nome exato de arquivo (sem passar pelo hashing)
- ...ou você simplesmente não quer quer ter de importar um asset primeiro para obter sua URL

Então você pode colocar o asset em um diretorio especial `public` dentro da pasta raiz do projeto. Assets nesse diretorio são servidos no caminho `/` durante o desenvolvimento, e copiado para a raiz da dist como é.

Os padrões do diretorio `<root>/public`, podem ser configurados pela opção [`publicDir`](/config/#publicdir).

Note que:

- Você deve sempre referenciar assets `public` utilizando caminhos absolutos - por exemplo, `public/icon.png` deve ser referenciado no codigo fonte como`/icon.png`.
- Assets em `public` não podem ser importado pelo JavaScript.

## nova URL(url, import.meta.url)

[import.meta.url](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/import.meta) é um recurso nativo que expõe a URL do modulo atual. Combinado com o construtor de [URL nativa](https://developer.mozilla.org/en-US/docs/Web/API/URL), nos podemos obter a URL resolvida de um asset utilizando caminho relativo de um modulo JavaScript.

```js
const imgUrl = new URL('./img.png', import.meta.url)

document.getElementById('hero-img').src = imgUrl
```

Isso functiona nativamente em navegadores modernos - na verdade, Vite não precisa processar esse codigo durante desenvolvimento!

Esse padrão também suporta URLs dinamicas via template literals:

```js
function getImageUrl(name) {
  return new URL(`./dir/${name}.png`, import.meta.url).href
}
```

Durante a build e produção, Vite vai perfomar as tranformações necessaria para que as URL ainda apontem para os locais corretos mesmo depois do build e hash dos assets.

::: Nota de perigo: Não funciona com SSR
Esse padrão não funciona se você esta usando Vite para Server-Side Rendering, porque `import.meta.url` tem diferenças semanticas entre navegadores e Node.js. O servidor não pode determinar as URL do cliente no futuro.
:::
