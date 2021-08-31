# JavaScript API

As APIs de Javascript do Vite são completamente tipadas e é recomendado usar Typescript ou habilitar a checagem de tipos de JS no VSCode para tirar vantagem das validações e _intellisense_.

## `createServer`

**Assinatura de Tipo:**

```ts
async function createServer(inlineConfig?: InlineConfig): Promise<ViteDevServer>
```

**Exemplo de Uso:**

```js
const { createServer } = require('vite')

;(async () => {
  const server = await createServer({
    // qualquer opção de configuração de usuário válida, mais `mode` e `configFile`
    configFile: false,
    root: __dirname,
    server: {
      port: 1337
    }
  })
  await server.listen()
})()
```

## `InlineConfig`

A interface do `InlineConfig` extende `UserConfig` com propriedades adicionais:

- `configFile`: especifica o arquivo de configuração a ser usado. Se não for definido, Vite irá tentar resolver algum arquivo da raíz automaticamente. Defina o valor `false` para desabilitar a resolução automatica.
- `envFile`: Defina como `false` para desabilitar arquivos `.env`.

## `ViteDevServer`

```ts
interface ViteDevServer {
  /**
   * O objeto de configuração Vite resolvido
   */
  config: ResolvedConfig
  /**
   * Uma instância de app conectada
   * - Pode ser usada para anexar _middlewares_ customizados ao servidor de desenvolvimento.
   * - Também pode ser usado como função de gerenciamento de um servidor http customizado ou como _middleware_ em qualquer framework de Node.js
   *
   * https://github.com/senchalabs/connect#use-middleware
   */
  middlewares: Connect.Server
  /**
   * Instância nativa de servidor http em Node.
   * Será null quando estiver em modo _middleware_
   */
  httpServer: http.Server | null
  /**
   * Instância de Chokidar watcher.
   * https://github.com/paulmillr/chokidar#api
   */
  watcher: FSWatcher
  /**
   * Servidor Web sockets com o método `send(payload)`.
   */
  ws: WebSocketServer
  /**
   * Container de plugin Rollup que pode rodar _hooks_ em um determinado arquivo.
   */
  pluginContainer: PluginContainer
  /**
   * Grafo de módulos que acompanha as relações de importações, mapeamento de url para arquivo
   * e estado do hmr.
   */
  moduleGraph: ModuleGraph
  /**
   * Resolve programaticamente, carrega e transforma a URL e pega o resultado
   * sem passar pelo pipeline de requisição http.
   */
  transformRequest(
    url: string,
    options?: TransformOptions
  ): Promise<TransformResult | null>
  /**
   * Aplicado a transformação nativa do Vite para HTML e qualquer plugin que o HTML transforma.
   */
  transformIndexHtml(url: string, html: string): Promise<string>
  /**
   * Utilidade para transformar um arquivo com esbuild.
   * Pode ser útil para certos plugins.
   */
  transformWithEsbuild(
    code: string,
    filename: string,
    options?: EsbuildTransformOptions,
    inMap?: object
  ): Promise<ESBuildTransformResult>
  /**
   * Carrega uma determinada URL como uma instância de módulo para SSR.
   */
  ssrLoadModule(
    url: string,
    options?: { isolated?: boolean }
  ): Promise<Record<string, any>>
  /**
   * Ajusta o erro de stacktrace de ssr.
   */
  ssrFixStacktrace(e: Error): void
  /**
   * Inicia o servidor.
   */
  listen(port?: number, isRestart?: boolean): Promise<ViteDevServer>
  /**
   * Pausa o server.
   */
  close(): Promise<void>
}
```

## `build`

**Assinatura de Tipo:**

```ts
async function build(
  inlineConfig?: InlineConfig
): Promise<RollupOutput | RollupOutput[]>
```

**Exemplo de Uso:**

```js
const path = require('path')
const { build } = require('vite')

;(async () => {
  await build({
    root: path.resolve(__dirname, './project'),
    build: {
      base: '/foo/',
      rollupOptions: {
        // ...
      }
    }
  })
})()
```

## `resolveConfig`

**Assinatura de Tipo:**

```ts
async function resolveConfig(
  inlineConfig: InlineConfig,
  command: 'build' | 'serve',
  defaultMode?: string
): Promise<ResolvedConfig>
```
