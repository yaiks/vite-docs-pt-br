# Exportando para Produção

Quando chegar a hora de exportar seu aplicativo para produção, simplesmente execute o comando `vite build`. Por padrão, ele usa `<root>/index.html` como o ponto de entrada para o build e gera um bundle da aplicação que é adequado para ser servido por um serviço de hospedagem estática. Confira [Fazendo o deploy de um Site Estático](./static-deploy) para obter guias sobre serviços populares.

## Compatibilidade do Browser

O bundle de produção pressupõe suporte para JavaScript moderno. Por padrão, o vite visa navegadores que suportam a [tag de script nativa do ESM](https://caniuse.com/es6-module) e [importação dinâmica nativa do ESM](https://caniuse.com/es6-module-dynamic-import). Como referência, vite usa esta consulta [browserslist](https://github.com/browserslist/browserslist):

```
defaults and supports es6-module and supports es6-module-dynamic-import, not opera > 0, not samsung > 0, not and_qq > 0
```

Você pode especificar alvos personalizados por meio da [opção de configuração `build.target`](/config/#build-target), onde o alvo mais baixo é `es2015`.

Observe que, por padrão, o Vite lida apenas com transformações de sintaxe e **não cobre polyfills por padrão**. Você pode verificar em [Polyfill.io](https://polyfill.io/v3/), que é um serviço que gera pacotes polyfill automaticamente com base na string UserAgent do browser do usuário.

Browsers legados podem ser suportados por meio do [@vitejs/plugin-legacy](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy), que irá gerar automaticamente chunks legados e polyfills de recursos correspondentes a linguagem ES. Os chunks legados são carregados condicionalmente apenas em navegadores que não têm suporte nativo a ESM.

## Caminho Base para a Public

- Relacionado: [Manuseando os Assets Estáticos](./assets)

Se você estiver compilando seu projeto em um caminho público aninhado, simplesmente especifique a [opção de configuração `base`](/config/#base) e todos os caminhos dos assets serão reescritos de acordo. Esta opção também pode ser especificada com uma flag por linha de comando, por exemplo, `vite build --base=/my/public/path/`.

URLs de assets importados por JS, referências CSS `url()` e referências de assets em seus arquivos `.html` são todos ajustados automaticamente para respeitar esta opção durante a compilação.

A exceção é quando você precisa concatenar URLs dinamicamente. Nesse caso, você pode usar a variável `import.meta.env.BASE_URL` injetada globalmente que será o caminho de base público. Observe que esta variável é substituída estaticamente durante a compilação, então ela deve aparecer exatamente como está (ou seja, `import.meta.env['BASE_URL']` não funcionará).

## Personalizando o Build

O build pode ser personalizado por meio de várias [opções de configuração de build](/config/#build-options). Especificamente, você pode ajustar diretamente as [opções de Rollup](https://rollupjs.org/guide/en/#big-list-of-options) subjacentes por meio de `build.rollupOptions`:

```js
// vite.config.js
module.exports = defineConfig({
  build: {
    rollupOptions: {
      // https://rollupjs.org/guide/en/#big-list-of-options
    }
  }
})
```

Por exemplo, você pode especificar várias saídas de Rollup com plugins que são aplicados apenas durante a construção.

## Reconstruir nas alterações dos arquivos

Você pode habilitar o watcher de rollup com `vite build --watch`. Ou você pode ajustar diretamente as [`WatcherOptions`](https://rollupjs.org/guide/en/#watch-options) subjacentes via `build.watch`:

```js
// vite.config.js
module.exports = defineConfig({
  build: {
    watch: {
      // https://rollupjs.org/guide/en/#watch-options
    }
  }
})
```

## App de várias páginas

Suponha que você tenha a seguinte estrutura de código-fonte:

```
├── package.json
├── vite.config.js
├── index.html
├── main.js
└── nested
    ├── index.html
    └── nested.js
```

Durante o desenvolvimento, simplesmente navegue ou crie um link para `/nested/` - funciona como esperado, como para um servidor de arquivos estático normal.

Durante a compilação, tudo que você precisa fazer é especificar vários arquivos `.html` como pontos de entrada:

```js
// vite.config.js
const { resolve } = require('path')
const { defineConfig } = require('vite')

module.exports = defineConfig({
  build: {
    rollupOptions: {
      input: {
        main: resolve(__dirname, 'index.html'),
        nested: resolve(__dirname, 'nested/index.html')
      }
    }
  }
})
```

Se você especificar uma raiz diferente, lembre-se de que `__dirname` ainda será a pasta do seu arquivo vite.config.js ao resolver os caminhos de entrada. Portanto, você precisará adicionar sua entrada `root` aos argumentos para o `resolve`.

## Modo Biblioteca

Quando você está desenvolvendo uma biblioteca orientada para o browser, provavelmente passa a maior parte do tempo em uma página de teste/demo que importa sua biblioteca real. Com o Vite, você pode usar seu `index.html` para essa finalidade para obter uma experiência de desenvolvimento suave.

Quando chegar a hora de compilar sua biblioteca para distribuição, use a [opção de configuração `build.lib`](/config/#build-lib). Certifique-se de também externalizar quaisquer dependências que você não deseja agrupar em sua biblioteca, por exemplo, `vue` ou `react`:

```js
// vite.config.js
const path = require('path')
const { defineConfig } = require('vite')

module.exports = defineConfig({
  build: {
    lib: {
      entry: path.resolve(__dirname, 'lib/main.js'),
      name: 'MyLib',
      fileName: (format) => `my-lib.${format}.js`
    },
    rollupOptions: {
      // Certifique-se de externalizar dependências que não devem ser agrupadas
      // em sua biblioteca
      external: ['vue'],
      output: {
        // Fornece variáveis ​​globais para usar na construção UMD
        // para dependências externalizadas
        globals: {
          vue: 'Vue'
        }
      }
    }
  }
})
```

Ao executar `vite build` com esta configuração será usada uma predefinição de Rollup que é orientada para bibliotecas de remessa e produz dois formatos de pacote: `es` e `umd` (configurável via `build.lib`):

```
$ vite build
building for production...
[write] my-lib.es.js 0.08kb, brotli: 0.07kb
[write] my-lib.umd.js 0.30kb, brotli: 0.16kb
```

`Package.json` recomendado para sua biblioteca::

```json
{
  "name": "my-lib",
  "files": ["dist"],
  "main": "./dist/my-lib.umd.js",
  "module": "./dist/my-lib.es.js",
  "exports": {
    ".": {
      "import": "./dist/my-lib.es.js",
      "require": "./dist/my-lib.umd.js"
    }
  }
}
```
