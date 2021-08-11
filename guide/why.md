# Por quê Vite

## Os problemas

Antes do _ES modules_ estar disponível nos browsers, desenvolvedores não tinham nenhum mecanismo nativo para escrever Javascript de forma modular. Por isso todos nós ficamos familiarizados com o conceito de "empacotar": usar ferramentas que percorrem, processam e concatenam os módulos internos do nosso projeto em arquivos que podem rodar no browser.

Ao longo do tempo vimos ferramentas como [webpack](https://webpack.js.org/), [Rollup](https://rollupjs.org) e [Parcel](https://parceljs.org/), que melhoraram muito a experiência de desenvolvimento no frontend.

Porém, conforme criamos aplicações cada vez mais ambiciosas, a quantidade de Javascript que precisamos gerenciar aumenta exponencialmente. Não é incomum ver projetos de grande escala com milhares de módulos. Estamos começando a atingir um gargalo de performance para ferramentas baseadas em Javascript: pode-se levar uma quantidade enorme de tempo (às vezes minutos!) para subir um servidor de desenvolvimento, e mesmo com HMR, edições em um arquivo podem demorar segundos para refletir no browser. Esse tipo de resposta lenta pode afetar a produtividade e satisfação dos desenvolvedores.

Vite busca endereçar esses problemas aproveitando dos novos avanços no ecossistema: a disponibilidade de _ES modules_ nativo no browser e o surgimento de ferramentas de Javascript escritas em linguagens que compilam para nativo.

### Demora para iniciar o servidor

Quando iniciamos um servidor de desenvolvimento, uma aplicação baseada em "empacotamento" precisa percorrer e contruir sua aplicação inteira antes de serví-la.

Vite melhora o tempo de início do servidor de desenvolvimento dividindo os módulos de uma aplicação em duas categorias: **dependências** e **código fonte**.

- **Dependências** em geral são códigos Javascript que não mudam frequentemente durante o desenvolvimento. Algumas dependências grandes (ex: biblioteca de componentes com centenas de módulos) também são custosas para processar. Dependências também podem ser enviadas em diversos formatos (ex: ESM ou CommonJS).

  Vite [pré empacota as dependências](./dep-pre-bundling) usando [esbuild](https://esbuild.github.io/). Esbuild é escrito em Go e pré empacota dependências 10-100x mais rápido que ferramentas baseadas em Javascript.

- **Código fonte** geralmente contém Javascript não puro que precisa ser transformado (ex: JSX, CSS ou componentes Vue/Svelte), e serão editados frequentemente. Ainda, nem todo código fonte precisa ser carregado ao mesmo tempo (ex: com _code-splitting_ baseado por rota)

  Vite serve o código fonte como [ESM nativo](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/Modules). Isso basicamente deixa o browser assumir parte do trabalho de um "empacotador": Vite só precisa transformar e servir o código fonte sob demanda, conforme o browser requisitar. Código carregado através de importações dinâmicas e condicionais só serão processados se forem usadas na tela atual.

  ![servidor de desenvolvimento baseado em empacotador](/images/bundler.png)

  ![servidor de desenvolvimento baseado em ESM](/images/esm.png)

### Atualizações lentas

Quando um arquivo é editado em uma aplicação baseada em "empacotamento", é ineficiente reconstruir todo o pacote por razões óbvias: a velocidade de atualização diminui de acordo com o tamanho da aplicação.

Um servidor de desenvolvimento faz o empacotamento em memória para que só precise invalidar uma parte do grafo de módulos quando uma arquivo é alterado, mas ainda precisa reconstruir todo o pacote e recarregar a página da web. Reconstruir o pacote pode ser custoso, e recarregar a página acaba com o estado atual da aplicação. É por isso que alguns empacotadores oferecem suporte ao _Hot Module Replacement_ (HMR): a possibilidade do módulo substituir a si mesmo sem afetar o resto da página. Isso ajuda a melhorar a experiência do desenvolvedor - porém na prática vemos que até a velocidade das atualizações com HMR diminui conforme a aplicação cresce.

Com Vite, HMR é performado através de ESM nativo. Quando um arquivo é editado, Vite precisa apenas invalidar a cadeia entre o módulo editado e seus pares mais próximos (na maioria das vezes apenas o módulo em si), tornando as atualizações de HMR consistentemente rápidas independente do tamanho da aplicação.

Vite também tira proveito dos headers de HTTP para aumentar a velocidade de recarregamentos de toda a página (novamente, deixa o browser fazer mais trabalho por nós): requisições de módulos de código fonte são feitos condicionalmente via `304 Not Modified`, e requisições de módulos de dependência são fortemente cacheados via `Cache-Control: max-age=31536000,immutable`, para que não acessem o servidor uma vez que forem cacheados.

Quando você experimentar o quão rápido Vite é, duvidamos que você volte a desenvolver com empacotadores.

## Por quê empacotar para produção?

Apesar do ESM nativo ser largamente suportado atualmente, usar módulos ESM separados em produção ainda é ineficiente (mesmo com HTTP/2) devido às requisições de rede adicionais causadas por importações de módulos aninhados. Para obter a melhor performance de carregamento em produção, ainda é melhor empacotar seu código e usar técnicas como _tree-shaking_, _lazy-loading_ e _chunk splitting_ (para um cache melhor).

Garantir um resultado otimizado e consistência comportamental entre servidor de desenvolvimento e build de produção não é fácil. É por isso que Vite vem com um [comando de build](./build) pré configurado que reúne várias [otimizações de performance](./features#build-optimizations) por padrão.

## Por quê não empacotar usando esbuild?

Apesar do `esbuild` ser super rápido e um ótimo empacotador para bibliotecas, algumas funcionalidades importantes para empacotar _aplicações_ ainda estão sendo construídas - particularmente _code-splitting_ (separação de código) e gerenciamento de CSS. A essa altura, Rollup está mais maduro e é mais flexível nesses quesitos. Dito isso, nós não excluímos a possibilidade de usar `esbuild` para fazer build em produção quando essas funcionalidades estiverem prontas.

## Como Vite é diferente de X?

Você pode conferir a sessão de [Comparações](./comparisons) para mais detalhes de como Vite é diferente de outras ferramentas similares.
