# Primeiros passos

## Visão geral

Vite (palavra francesa para "rápido", se pronuncia `/vit/`) é uma ferramenta de _build_ que busca fornecer uma experiência de desenvolvimento rápida e suave para projetos web modernos. Vite consiste basicamente de duas partes:

- Um servidor de desenvolvimento que oferece [amplas melhorias de funcionalidade](./features) sobre [ES modules nativo](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules), por exemplo um [Hot Module Replacement (HMR)](./features#hot-module-replacement) extremamente rápido.

- Um comando de _build_ que empacota seu código usando [Rollup](https://rollupjs.org), pré-configurado para entregar arquivos estáticos extremamente otimizados para produção.

Vite é opinativo e vem com configurações sensíveis por padrão, mas também é extremamente customizável através da [API de Plugins](./api-plugin) e [API de Javascript](./api-javascript) com total suporte para tipagem.

Você pode aprender mais sobre o racional por trás do projeto na seção [Por quê Vite](./why).

## Suporte de Browsers

- O _build_ padrão atinge browsers que suportam tanto [ESM nativo através de tags scripts](https://caniuse.com/es6-module) quanto [importação dinâmica de ESM nativo](https://caniuse.com/es6-module-dynamic-import). Browsers legados podem ser suportados através do plugin oficial [@vitejs/plugin-legacy](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy) - veja a seção [Construindo para Produção](./build) para mais detalhes.

## Começando seu primeiro projeto VIte

::: tip Nota de Compatibilidade
Vite exige uma versão do [Node.js](https://nodejs.org/en/) >=12.0.0.
:::

Com NPM:

```bash
$ npm init vite@latest
```

Com Yarn:

```bash
$ yarn create vite
```

Com PNPM:

```bash
$ pnpx create-vite
```

Agora é só seguir os comandos que aparecerem no terminal!

Você também pode especificar diretamente o nome do projeto e o template a ser usado através de argumentos na linha de comando. Por exemplo, para começar um projeto Vite + Vue, rode:

```bash
# npm 6.x
npm init vite@latest my-vue-app --template vue

# npm 7+, traços duplos são necessários:
npm init vite@latest my-vue-app -- --template vue

# yarn
yarn create vite my-vue-app --template vue
```

Receitas de templates suportados incluem:

- `vanilla`
- `vanilla-ts`
- `vue`
- `vue-ts`
- `react`
- `react-ts`
- `preact`
- `preact-ts`
- `lit-element`
- `lit-element-ts`
- `svelte`
- `svelte-ts`

Veja [create-vite](https://github.com/vitejs/vite/tree/main/packages/create-vite) para mais detalhes sobre cada template.

## Templates da Comunidade

_create-vite_ é uma ferramenta para começar um projeto rapidamente, a partir de um template básico de um framework popular. Confira _Awesome Vite_ para [templates mantidos pela comunidade](https://github.com/vitejs/awesome-vite#templates) que incluem outras ferramentas ou usam outros frameworks. Você pode usar uma ferramenta como [degit](https://github.com/Rich-Harris/degit) para começar seu projeto com um desses templates.

```bash
npx degit user/project my-project
cd my-project

npm install
npm run dev
```

Se o projeto usa `main` como _branch_ padrão, coloque o sufixo `#main` no repositório do projeto.

```bash
npx degit user/project#main my-project
```

## `index.html` e a Raíz do Projeto

Uma coisa que você deve ter reparado é que em um projeto Vite, o `index.html` é um elemento central e não apenas colocado dentro da pasta `public`. Isso é intencional: durante o desenvolvimento, Vite é um servidor e o `index.html` é o ponto de entrada da aplicação.

Vite trata o `index.html` como um código fonte e parte do grafo de módulos. Vite interpreta o `<script type="module" src="...">` que referencia seu código fonte em Javascript. Até mesmo `<script type="module">` _inline_ e CSS referenciado através de `<link href>` se beneficiam das funcionalidades do Vite. Ainda, URLs dentro de `index.html` são automaticamente interpretadas, então não há necessidade em usar marcadores de posição especiais como `%PUBLIC_URL%`.

Similar à servidores http estáticos, Vite usufrui do conceito de "diretório raíz" de onde seus arquivos são servidos. Você os verá sendo referenciados como `<root>` ao longo da documentação. URLs absolutas no seu código fonte serão interpretadas usando a raíz do projeto como base, então você pode escrever código como se estivesse usando um servidor estático de arquivos normal (exceto que vite é muito mais poderoso!). Vite também é capaz de lidar com dependências que "resolvem" fora da raíz do sistema de arquivos, o que o torna utilizável até mesmo em projetos baseados em monorepo.

Vite também suporta [aplicações multi página](./build#multi-page-app), com múltiplas entradas de `.html`.

#### Especificando uma raíz alternativa

Rodar `vite` inicia o servidor de desenvolvimento usando o diretório atual de trabalho como raíz. Você pode especificar uma raíz alternativa com `vite serve some/sub/dir`.

## Interface da Linha de Comando

Em um projeto no qual Vite está instalado, você pode usar o binário de `vite` em seu script do npm, ou rodar diretamente com `npx vite`. Aqui está o script de npm padrão de um projeto iniciado com Vite:

```json
{
  "scripts": {
    "dev": "vite", // inicia o servidor de desenvolvimento
    "build": "vite build", // build para produção
    "serve": "vite preview" // oferece preview local para o build de produção
  }
}
```
Você pode especificar argumentos adicionais na linha de comando como `--port` ou `--https`. Para uma lista completa de argumentos de linha de comando, rode `npx vite --help` no seu projeto.

## Usando versões do projeto que não foram lançadas

Se você não quer esperar um novo lançamento acontecer para testar novas funcionalidades, você precisará clonar o [repositório do vite](https://github.com/vitejs/vite) na sua máquina local e então fazer o _build_ e linkar você mesmo ([Yarn 1.x](https://classic.yarnpkg.com/lang/en/) é necessário):

```bash
git clone https://github.com/vitejs/vite.git
cd vite
yarn
cd packages/vite
yarn build
yarn link
```
Então vá para o seu projeto baseado em vite e rode `yarn link vite`. Agora reinicie o servidor de desenvolvimento (`yarn dev`) para usar as novas funcionalidades!

## Comunidade

Se você tem alguma dúvida ou precisa de ajuda, contate a comunidade no canal do [Discord](https://discord.gg/4cmKdMfpU5) e nas [Discussões do Github](https://github.com/vitejs/vite/discussions).
