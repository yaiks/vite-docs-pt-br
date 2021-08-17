# HMR (Hot Module Reload) API

:::tip Nota
Essa é a API HMR do lado do cliente. Para lidar com atualizações HMR em plugins, veja [handleHotUpdate](./api-plugin#handlehotupdate).

A API manual do HMR é direcionada principalmente para autores de frameworks e ferramentas. Como um usuário final, HMR provavelmente já é gerenciada pelo template inicial do seu framework de escolha.
:::

Vite expõe sua API manual de HMR através do objeto especial `import.meta.hot`:

```ts
interface ImportMeta {
  readonly hot?: {
    readonly data: any

    accept(): void
    accept(cb: (mod: any) => void): void
    accept(dep: string, cb: (mod: any) => void): void
    accept(deps: string[], cb: (mods: any[]) => void): void

    dispose(cb: (data: any) => void): void
    decline(): void
    invalidate(): void

    on(event: string, cb: (...args: any[]) => void): void
  }
}
```

## Guardião Condicional Obrigatório

Primeiramente, garanta que todo uso da API de HMR será guardada por uma condicional para que o código possa ser _tree-shaken_ em produção:

```js
if (import.meta.hot) {
  // código HMR
}
```

## `hot.accept(cb)`

Para um módulo se _auto-aceitar_, use `import.meta.hot.accept` com um _callback_ que recebe o módulo atualizado:

```js
export const count = 1

if (import.meta.hot) {
  import.meta.hot.accept((newModule) => {
    console.log('updated: count is now ', newModule.count)
  })
}
```

Um módulo que "aceita" _hot updates_ é considerado um **HMR boundary**.

Note que o _HMR_ do Vite não troca o módulo importado original: se um módulo _HMR boundary_ reexporta importações de uma dependência, então ele é responsável por atualizar essas reexportações (e essas exportações precisam usar `let`). Ainda, importadores acima da cadeia a partir do módulo _HMR boundary_ não serão notificados da mudança.

Essa implementação de HMR simplificada é suficiente para a maioria dos casos de uso, enquanto permite que Vite pule etapas custosas do trabalho de gerar módulos _proxy_.

## `hot.accept(deps, cb)`

Um módulo também pode aceitar atualizações de dependências diretas sem atualizar a si mesmo:

```js
import { foo } from './foo.js'

foo()

if (import.meta.hot) {
  import.meta.hot.accept('./foo.js', (newFoo) => {
    // o callback recebe o módulo ./foo.js atualizado
    newFoo.foo()
  })

  // Também pode aceitar uma lista de módulos dependentes
  import.meta.hot.accept(
    ['./foo.js', './bar.js'],
    ([newFooModule, newBarModule]) => {
      // o callback recebe os módulos atualizados na lista
    }
  )
}
```

## `hot.dispose(cb)`

Um módulo que se _"auto-aceita"_ ou um módulo que espera ser aceito por outros pode usar `hot.dispose` para limpar qualquer efeito colateral persistente criado por sua cópia atualizada:

```js
function setupSideEffect() {}

setupSideEffect()

if (import.meta.hot) {
  import.meta.hot.dispose((data) => {
    // limpa efeito colateral
  })
}
```

## `hot.data`

O objeto `import.meta.hot.data` é persistido entre várias instâncias diferentes do mesmo módulo atualizado. Pode ser usado para passar informação de uma versão prévia do módulo para a próxima.

## `hot.decline()`

Chamar `import.meta.hot.decline()` indica que esse módulo não é passível de _hot updates_, e o browser deve performar um recarregamento total se esse módulo for encontrado durante as atualizações HMR.

## `hot.invalidate()`

Por ora, chamar `import.meta.hot.invalidate()` simplesmente recarrega a página.

## `hot.on(event, cb)`

Ouve um evento de HMR.

Os seguintes eventos de HMR são disparados por Vite automaticamente:

- `'vite:beforeUpdate'` quando uma atualização está para ser aplicada (ex: um módulo será substituído)
- `'vite:beforeFullReload'` quando um recarregamento total está para acontecer
- `'vite:beforePrune'` quando módulos que não são mais necessários estão para ser eliminados
- `'vite:error'` quando um erro acontece (ex: erro de sintaxe)

Eventos customizados de HMR também podem ser enviado de plugins. Veja [handleHotUpdate](./api-plugin#handlehotupdate) para mais detalhes.
