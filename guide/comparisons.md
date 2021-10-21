# Comparações com outras soluções No-Bundler

## Snowpack

[Snowpack](https://www.snowpack.dev/) também é um servidor de desenvolvimento ESM nativo no-bundle que é muito semelhante em escopo ao Vite. Além de diferentes detalhes de implementação, os dois projetos compartilham muito em termos de vantagens técnicas em relação às ferramentas tradicionais. O pre-bundling de dependência do Vite também é inspirado no Snowpack v1 (agora [`esinstall`](https://github.com/snowpackjs/snowpack/tree/main/esinstall)). Algumas das principais diferenças entre os dois projetos são:

**Build de produção**

A saída de compilação padrão do Snowpack é descompactada: ele transforma cada arquivo em módulos compilados separados, que podem então ser alimentados em diferentes "otimizadores" que realizam o bundle real. O benefício disso é que você pode escolher entre diferentes pacotes finais para atender a necessidades específicas (por exemplo, webpack, Rollup ou mesmo esbuild), a desvantagem é que é uma experiência um pouco fragmentada - por exemplo, o otimizador esbuild ainda é instável, o otimizador de Rollup não é oficialmente mantido e diferentes otimizadores têm diferentes saídas e configurações.

Vite opta por uma integração mais profunda com um único bundler (Rollup) para proporcionar uma experiência mais ágil. Também permite que o Vite ofereça suporte a uma [API Universal de Plugins](./api-plugin) que funciona tanto para desenvolvimento quanto para produção.

Devido a um processo de compilação mais integrado, o Vite oferece suporte a uma ampla gama de recursos que atualmente não estão disponíveis nos otimizadores de compilação do Snowpack:

- [Suporte para várias páginas](./build#app-de-varias-paginas)
- [Modo biblioteca](./build#modo-biblioteca)
- [Separação automática de código CSS (Code Splitting)](./features#separacao-de-codigo-css-code-splitting)
- [Otimização do carregamento de pedaços assíncronos](./features#otimizacao-do-carregamento-de-pedacos-assincronos)
- [Plugin oficial de modo legado](https://github.com/vitejs/vite/tree/main/packages/plugin-legacy) que gera pacotes duplos modernos/legados e entrega automaticamente o pacote certo com base no suporte do navegador.

**Pré-empacotamento de dependência mais rápido**

Vite usa [esbuild](https://esbuild.github.io/) em vez de Rollup para pré-empacotamento de dependência. Isso resulta em melhorias de desempenho significativas em termos de inicialização do servidor a frio e re-empacotamento em invalidações de dependência.

**Suporte à Monorepo**

O Vite foi projetado para lidar com configurações de monorepo e temos usuários usando-o com sucesso com Yarn, Yarn 2 e monorepo com base em PNPM.

**Suporte à pré-processadores CSS**

O Vite fornece suporte mais refinado para Sass e Less, incluindo resolução melhorada de `@import` (aliases e dependências npm) e [rebase automático de `url()` para arquivos importados](./features#import-inlining-and-rebasing).

**Suporte Vue de primeira classe**

O Vite foi inicialmente criado para servir como a futura base das ferramentas [Vue.js](https://vuejs.org/). Embora a partir do 2.0 Vite agora seja totalmente agnóstico à frameworks, o plugin oficial do Vue ainda fornece suporte de primeira classe para o formato de componente de arquivo único do Vue, cobrindo todos os recursos avançados, como resolução de referência de recurso de modelo, `<script setup>`, `<style module>`, blocos personalizados e mais. Além disso, Vite fornece HMR refinado para Vue SFCs. Por exemplo, atualizar o `<template>` ou `<style>` de um SFC executará atualizações rápidas sem redefinir seu estado.

## WMR

[WMR](https://github.com/preactjs/wmr) da equipe Preact fornece um conjunto de recursos semelhante, e o suporte do Vite 2.0 para a interface do plugin Rollup é inspirado nele.

O WMR é projetado principalmente para projetos [Preact](https://preactjs.com/) e oferece recursos mais integrados, como pré-renderização. Em termos de escopo, está mais próximo de um meta framework do Preact, com a mesma ênfase no tamanho compacto do próprio Preact. Se você estiver usando o Preact, o WMR provavelmente oferecerá uma experiência mais aprimorada.

## @web/dev-server

[@web/dev-server](https://modern-web.dev/docs/dev-server/overview/) (anteriormente `es-dev-server`) é um ótimo projeto e baseado em Koa do Vite 1.0 a configuração do servidor foi inspirada nele.

`@web/dev-server` é um nível um pouco inferior em termos de escopo. Ele não fornece integrações de estrutura oficial e requer ajustes manuais em uma configuração de Rollup para a compilação de produção.

No geral, o Vite é uma ferramenta mais opinativa / de nível superior que visa fornecer um fluxo de trabalho mais out-of-the-box. Dito isso, o projeto guarda-chuva `@web` contém muitas outras ferramentas excelentes que também podem beneficiar os usuários do Vite.
