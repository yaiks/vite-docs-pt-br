---
home: true
heroImage: /logo.svg
actionText: Começar
actionLink: /guide/

altActionText: Saiba mais
altActionLink: /guide/why

features:
  - title: 💡 Servidor instantâneo
    details: Arquivos sob demanda servidos através de ESM nativo, sem empacotamento necessário!
  - title: ⚡️ HMR Super rápido
    details: Hot Module Replacement (HMR) rápido, independente do tamanho da aplicação.
  - title: 🛠️ Rico em funcionalidades
    details: Por padrão, oferece suporte para TypeScript, JSX, CSS e mais.
  - title: 📦 Build otimizado
    details: Build com Rollup pré configurado, com suporte para múltiplas páginas e modo biblioteca.
  - title: 🔩 Plugins universais
    details: Interface do Rollup plugin compartilhada entre dev e build.
  - title: 🔑 APIs completamente tipadas
    details: APIs programáticas e flexíveis com tipagem completa em Typescript.
footer: MIT Licensed | Copyright © 2019-present Evan You & Vite Contributors
---

<div class="frontpage sponsors">
  <h2>Patrocinadores</h2>
  <a v-for="{ href, src, name, id } of sponsors" :href="href" target="_blank" rel="noopener" aria-label="sponsor-img">
    <img :src="src" :alt="name" :id="`sponsor-${id}`">
  </a>
  <br>
  <a href="https://github.com/sponsors/yyx990803" target="_blank" rel="noopener">Seja um patrocinador no GitHub</a>
</div>

<script setup>
import sponsors from './.vitepress/theme/sponsors.json'
</script>
