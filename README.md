# Vite Docs Pt-br

Este repositório visa traduzir a documentação oficial do [Vite](https://vitejs.dev/) para português. Após a tradução ser feita, será criado um novo repositório na organização do Vite para abrigar esta documentação em pt-br. Você pode ter acesso à mais informações no [canal do Discord ViteLand](https://discord.gg/vBC9pUP7)

## Como contribuir

Antes de começar a traduzir, crie uma issue com a parte que você deseja traduzir e dê o `assignee` para você mesmo. Dessa forma, deixamos explícito em qual parte estamos trabalhando e não corremos o risco de ter duas pessoas traduzindo a mesma parte.

Ao terminar de traduzir, abra um pull request para a branch `main`, [referenciando nos comentários](https://docs.github.com/en/issues/tracking-your-work-with-issues/linking-a-pull-request-to-an-issue) a issue que será resolvida.

Por exemplo, você pode escrever algo tipo:

```
closes #54

// ou

resolves #54
```

E depois que for mergeada, a issue irá fechar automaticamente =)

## Como rodar o projeto localmente

É necessário usar `yarn` pois estamos usando `resolutions` no package.json para controlar as versões das sub dependências do vitepress, visto que [há um bug](https://github.com/vuejs/vitepress/issues/358) nesse momento (10/08/2021).

```sh
# Instale as dependências
$ yarn

# Rode o servidor local
$ yarn docs
```

O servidor local vai abrir em `localhost:3000`

