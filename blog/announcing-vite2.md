---
sidebar: false
---

# Anunciando Vite 2.0

<p style="text-align:center">
  <img src="../images/logo.svg" style="height:200px">
</p>

Hoje estamos animados em anunciar o lançamento oficial do Vite 2.0!

Vite (Palavra francesa para "rápido", pronunciada `/vit/`) é um novo tipo de ferramenta para desenvolvimento frontend web. Pense em uma combinação de dev server + bundler combo, porém mais enxuta e rápida. Ele aproveita o suporte a [módulos ES nativos](https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Guide/Modules) do navegador e ferramentas escritas em linguagens de compilação para nativas, como a [esbuild](https://esbuild.github.io/) para oferecer uma experiência de desenvolvimento rápido e moderna.

Para ter uma ideia de como o Vite é rápido, confira [este vídeo de comparação](https://twitter.com/amasad/status/1355379680275128321) de inicialização de uma aplicação React no Repl.it usando Vite vs. `create-react-app` (CRA).

Se você nunca ouviu falar de Vite antes e gostaria de saber mais sobre ele, verifique a [lógica por trás do projeto](https://vitejs.dev/guide/why.html). Se você estiver interessado em saber como o Vite difere de outras ferramentas semelhantes, verifique as [comparações](https://vitejs.dev/guide/comparisons.html).

## O que há de novo em 2.0

Já que decidimos refatorar completamente os internos antes que 1.0 saísse do RC, esta é de fato a primeira versão estável do Vite. Dito isso, o Vite 2.0 traz muitas melhorias importantes em relação à sua encarnação anterior:

### Framework Agnostic Core

A ideia original do Vite começou como um [protótipo hacky que atende aos componentes de arquivo único do Vue em vez do ESM nativo](https://github.com/vuejs/vue-dev-server). O Vite 1 foi uma continuação dessa ideia com HMR implementado no topo.

O Vite 2.0 pega o que aprendemos ao longo do caminho e é redesenhado do zero com uma arquitetura interna mais robusta. Agora é completamente independente de estrutura e todo o suporte específico de estrutura é delegado a plugins. Existem agora [modelos oficiais para Vue, React, Preact, Lit Element](https://github.com/vitejs/vite/tree/main/packages/create-vite) e esforços contínuos da comunidade para integração Svelte.

### Novo formato de plugin e API

Inspirado no [WMR](https://github.com/preactjs/wmr), o novo sistema de plugin estende a interface de plugin do Rollup e é [compatível com muitos plugins de Rollup](https://vite-rollup-plugins.patak.dev/) prontos para uso. Os plugins podem usar hooks compatíveis com Rollup, com hooks e propriedades específicas do Vite adicionais para ajustar o comportamento exclusivo do Vite (por exemplo, diferenciando dev vs. build ou custom handling do HMR).

A [API programática](https://vitejs.dev/guide/api-javascript.html) também foi bastante aprimorada para facilitar ferramentas / estruturas de nível superior criadas com base no Vite.

### esbuild Powered Dep Pre-Bundling

Como o Vite é um servidor de desenvolvimento ESM nativo, ele pré-agrupa dependências para reduzir o número de solicitações do navegador e lidar com a conversão de CommonJS para ESM. Anteriormente, o Vite fazia isso usando Rollup, e agora no 2.0, usa `esbuild`, o que resulta em um pre-bundling de dependência 10-100x mais rápido. Como referência, a inicialização a frio de um aplicativo de teste com dependências pesadas como React Material UI levava anteriormente 28 segundos em um Macbook Pro M1 e agora leva cerca de ~1,5 segundos. Espere melhorias semelhantes se você estiver mudando de uma configuração tradicional baseada em bundler.

### Suporte CSS de primeira classe

Vite trata o CSS como um cidadão de primeira classe do gráfico do módulo e oferece os seguintes suportes:

- **Resolver enhancement**: `@import` and `url()` paths in CSS are enhanced with Vite's resolver to respect aliases and npm dependencies.
- **URL rebasing**: `url()` paths are automatically rebased regardless of where the file is imported from.
- **CSS code splitting**: a code-split JS chunk also emits a corresponding CSS file, which is automatically loaded in parallel with the JS chunk when requested.

### Server-Side Rendering (SSR) Support

Vite 2.0 ships with [experimental SSR support](https://vitejs.dev/guide/ssr.html). Vite provides APIs to efficiently load and update ESM-based source code in Node.js during development (almost like server-side HMR), and automatically externalizes CommonJS-compatible dependencies to improve development and SSR build speed. The production server can be completely decoupled from Vite, and the same setup can be easily adapted to perform pre-rendering / SSG.

Vite SSR is provided as a low-level feature and we are expecting to see higher level frameworks leveraging it under the hood.

### Opt-in Legacy Browser Support

Vite targets modern browsers with native ESM support by default, but you can also opt-in to support legacy browsers via the official [@vitejs/plugin-legacy](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy). The plugin automatically generates dual modern/legacy bundles, and delivers the right bundle based on browser feature detection, ensuring more efficient code in modern browsers that support them.

## Give it a Try!

That was a lot of features, but getting started with Vite is simple! You can spin up a Vite-powered app literally in a minute, starting with the following command (make sure you have Node.js >=12):

```bash
npm init @vitejs/app
```

Then, check out [the guide](https://vitejs.dev/guide/) to see what Vite provides out of the box. You can also check out the source code on [GitHub](https://github.com/vitejs/vite), follow updates on [Twitter](https://twitter.com/vite_js), or join discussions with other Vite users on our [Discord chat server](http://chat.vitejs.dev/).
