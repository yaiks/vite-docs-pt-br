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

Vite trata o CSS como um cidadão de primeira classe de module graph e apoia as seguintes linhas de pensamentos fora da caixa :

- **Solucionador do aprimoramento**: Os caminhos `@import` e `url()` em CSS, são aprimorados com o solucionador do Vite respeitando apelidos e dependências do npm.
- **Realocamento de URL**: `url()` os caminhos são realocados automaticamente, independentemente de onde o arquivo é importado.
- **Particionando um código CSS**: Um JS chunk fragmentado por código também emite um arquivo CSS correspondente, que é carregado automaticamente em paralelo com o JS chunk quando solicitado.

### Suporte de renderização do lado do servidor (SSR)

O Vite 2.0 é fornecido com [suporte a SSR experimental](https://vitejs.dev/guide/ssr.html). O Vite fornece APIs para carregar e atualizar com eficiência o código-fonte baseado em ESM no Node.js durante o desenvolvimento (quase como HMR do lado do servidor) e externaliza automaticamente as dependências compatíveis com CommonJS para melhorar o desenvolvimento e a velocidade de construção SSR. O servidor de produção pode ser completamente desacoplado do Vite, e a mesma configuração pode ser facilmente adaptada para realizar pré-renderização / SSG.

O Vite SSR é fornecido como um recurso de baixo nível e esperamos ver frameworks de alto nível aproveitando-o sob o capô.

### Suporte para navegador Opt-in Legacy

O Vite se destina a navegadores modernos com suporte a ESM nativo por padrão, mas você também pode optar por oferecer suporte a navegadores legacy por meio do [@vitejs/plugin-legacy](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy). O plugin gera automaticamente pacotes duplos moderno/legacy e oferece o pacote certo com base na detecção de recursos do navegador, garantindo um código mais eficiente em navegadores modernos que os suportam.

## De uma Chance!

São muitos recursos, mas começar com o Vite é simples! Você pode ativar um aplicativo com tecnologia Vite literalmente em um minuto, começando com o seguinte comando (certifique-se de ter Node.js >=12):

```bash
npm init @vitejs/app
```

Em seguida, verifique [o guia](https://vitejs.dev/guide/) para ver o que o Vite oferece pronto para uso. Você também pode verificar o código-fonte no [GitHub](https://github.com/vitejs/vite), acompanhar as atualizações no [Twitter](https://twitter.com/vite_js) ou participar de discussões com outro usuários do Vite em nosso [servidor no Discord](http://chat.vitejs.dev/).
