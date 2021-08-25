# Fazendo o deploy de um Site Estático

O guia irá assumir os seguintes aspectos:

- Você está usuando o destino de build padrão (`dist`). Esse destino [pode ser alterado usando `build.outDir`](https://vitejs.dev/config/#build-outdir), e você pode ignorar algumas instruções desse duia se for o caso.
- Você está usando npm. Você pode usar comandos equivalentes para rodar os scripts caso esteja usando Yarn ou outro gerenciador de pacotes.
- O Vite está instalado como uma dependencia local no seu projeto, e você já fez o setup com os seguintes scripts do npm:
```json
{
  "scripts": {
    "build": "vite build",
    "preview": "vite preview"
  }
}
```

É importante ressaltar que o comando `vite preview` é previsto para funcionar em seu ambiente local e não em produção.

::: tip NOTA
Esse guia vai trazer instruções para realizar um deploy estático do seu site em Vite. Vite também possui um suporte experimental para Server SIde Rendering. SSR é um framework que suporta rodar em uma aplicação em Node.js, pré-renderizando o HTML e hidratando o conteúdo no cliente. Veja o [Guia de SSR](./ssr) para aprender sobre essa funcionalidade. Caso você esteja procurando por uma integração com frameworks de server-side, busque em  [Guia de Integração com Backend](./backend-integration).
 On the other hand, if you are looking for integration with traditional server-side frameworks, check out the [Backend Integration guide](./backend-integration) instead.
:::


## Building a aplicação

Você deve rodar o comando `npm run build` para buildar a aplicação.

```bash
$ npm run build
```

Por padrão, o resultado do build será localizado na pasta `dist`. Você deve fazer o deploy da pasta `dist` para sua plataforma de escolha.

### Testando sua aplicação localmente

Uma vez que você já realizou o build do projeto, você deve testar ele localmente com o comando `npm run preview`.

```bash
$ npm run build
$ npm run preview
```

O comando `preview` vai levantar um servidor web estático no seu local que irá servir os arquivos que estão na pasta `dist` em http://localhost:5000. É uma maneira fácil de verificar se está tudo ok com o seu build de produção no seu ambiente local.

Você pode configurar a porta do servidor passando a flag `--port` como argumento.

```json
{
  "scripts": {
    "preview": "vite preview --port 8080"
  }
}
```

Agora o comando `preview` method vai rodar o servidor em http://localhost:8080.

## GitHub Pages

1. Cofigure corretamente o campo `base` no arquivo `vite.config.js`.

   Se você estiver fazendo o deploy para `https://<USERNAME>.github.io/`, você pode omiti o `base` que permanecerá por default como `'/'`.

   Se você estiver fazendo o deploy para `https://<USERNAME>.github.io/<REPO>/`, por exemplo, seu repositório está em `https://github.com/<USERNAME>/<REPO>`, então configure o `base` para `'/<REPO>/'`.

2. Dentro do seu projeto, crie o arquivo `deploy.sh` com o seguinte conteúdo (com as linhas demarcadas descomentadas apropriadamente), e execute para o deploy:

   ```bash{13,20,23}
   #!/usr/bin/env sh

   # Abortar em caso de erro
   set -e

   # build
   npm run build

   # Navegar para o diretório gerado pelo build
   cd dist

   # se você estiver fazendo o deploy para um domínio customizado
   # echo 'www.example.com' > CNAME

   git init
   git add -A
   git commit -m 'deploy'

   #  se você estiver fazendo o deploy para https://<USERNAME>.github.io
   # git push -f git@github.com:<USERNAME>/<USERNAME>.github.io.git master

   #  se você estiver fazendo o deploy para https://<USERNAME>.github.io/<REPO>
   # git push -f git@github.com:<USERNAME>/<REPO>.git master:gh-pages

   cd -
   ```

::: dica
Você também pode rodar o script anterior na sua configuração de CI para um deploy automatizado a cada push.
:::

### GitHub Pages e Travis CI

1. Cofigure corretamente o campo `base` no arquivo `vite.config.js`.

   Se você estiver fazendo o deploy para `https://<USERNAME>.github.io/`, você pode omiti o `base` que permanecerá por default como `'/'`.

   Se você estiver fazendo o deploy para `https://<USERNAME>.github.io/<REPO>/`, por exemplo, seu repositório está em `https://github.com/<USERNAME>/<REPO>`, então configure o `base` para `'/<REPO>/'`.

2. Crie um arquivo chamado `.travis.yml` na raiz do seu projeto.

3. Execute `npm install` localmente e faço o commit do arquiivo de lock gerado (`package-lock.json`).

4. Use o template de deploy do GitHub Pages, e siga os passos da [documentação do Travis CI](https://docs.travis-ci.com/user/deployment/pages/).

   ```yaml
   language: node_js
   node_js:
     - lts/*
   install:
     - npm ci
   script:
     - npm run build
   deploy:
     provider: pages
     skip_cleanup: true
     local_dir: dist
     # Um token gerado no GitHub que permite o Travis de fazer o push do código no seu repositório.
     # Defina na página de configurações do Travis no seu repositório, como uma variável segura.
     github_token: $GITHUB_TOKEN
     keep_history: true
     on:
       branch: master
   ```

## GitLab Pages e GitLab CI

1. Cofigure corretamente o campo `base` no arquivo `vite.config.js`.

   Se você estiver fazendo o deploy para `https://<USERNAME>.github.io/`, você pode omiti o `base` que permanecerá por default como `'/'`.

   Se você estiver fazendo o deploy para `https://<USERNAME>.github.io/<REPO>/`, por exemplo, seu repositório está em `https://github.com/<USERNAME>/<REPO>`, então configure o `base` para `'/<REPO>/'`.

2. Crie um arquivo chamado `.gitlab-ci.yml` na raíz do seu projeto com o seguinte conteúdo. Isso irá realizar o build e o  deploy do seu site toda vez que você fizer uma alteração no seu conteúdo:

   ```yaml
   image: node:16.5.0
   pages:
     stage: deploy
     cache:
       key:
         files:
           - package-lock.json
         prefix: npm
       paths:
         - node_modules/
     script:
       - npm install
       - npm run build
       - cp -a dist/. public/
     artifacts:
       paths:
         - public
     rules:
       - $CI_COMMIT_BRANCH == $CI_DEFAULT_BRANCH
   ```

## Netlify

1. No [Netlify](https://netlify.com), configure um novo projeto do Github com os seguintes parâmetros:

   - **Build Command:** `vite build` or `npm run build`
   - **Publish directory:** `dist`

2. Aperte o botão de deploy.

## Google Firebase

1. Garanta que você tenha o [firebase-tools](https://www.npmjs.com/package/firebase-tools) instalado.

2. Crie os arquivos `firebase.json` e `.firebaserc` na raíz do seu projeto com o seguinte conteúdo:

   `firebase.json`:

   ```json
   {
     "hosting": {
       "public": "dist",
       "ignore": []
     }
   }
   ```

   `.firebaserc`:

   ```js
   {
     "projects": {
       "default": "<YOUR_FIREBASE_ID>"
     }
   }
   ```

3. Depois execute o comando `npm run build`, e faça o deploy usando o comando `firebase deploy`.

## Surge

1. Primeiro instale o [surge](https://www.npmjs.com/package/surge), se você ainda não o tenha instalado.

2. Execute `npm run build`.

3. Faça o deploy para o Surge executando: `surge dist`.

Você também pode realizar o deploy para um [domínio customizado](http://surge.sh/help/adding-a-custom-domain) adicionando `surge dist seudominio.com`.

## Heroku

1. Instale [Heroku CLI](https://devcenter.heroku.com/articles/heroku-cli).

2. Cria uma conta no Heroku [signing up](https://signup.heroku.com).

3. Execute `heroku login` e preencha com suas credenciais do Heroku:

   ```bash
   $ heroku login
   ```

4. Crie um arquivo chamado `static.json` na raíz do seu projeto com o seguinte conteúdo:

   `static.json`:

   ```json
   {
     "root": "./dist"
   }
   ```

   Essa é a configuração do seu site; leia mais em [heroku-buildpack-static](https://github.com/heroku/heroku-buildpack-static).

5. Configure o remote do Heroku no git:

   ```bash
   # version change
   $ git init
   $ git add .
   $ git commit -m "My site ready for deployment."

   # cria um novo app com o nome passado
   $ heroku apps:create example

   # Define o buildpack para sites estáticos
   $ heroku buildpacks:set https://github.com/heroku/heroku-buildpack-static.git
   ```

6. Faça o deploy do seu site:

   ```bash
   # publique o site
   $ git push heroku master

   # abre o nvaegador para visualizar a Dashboard do Heroku CI
   $ heroku open
   ```

## Vercel

Para fazer o deploy da sua aplicação Vite com o [Vercel para Git](https://vercel.com/docs/git), garanta que você tenha realizado o push do seu projeto em um repositório git.

Vá até https://vercel.com/import/git e importe o seu projeto no Vercel usando o Git de sua escolha (GitHub, GitLab or BitBucket). Siga a interface para selecionar o `package.json` na raíz do seu projeto e sobrescreva o passo de build usando o comando `npm run build` e o diretório de destino será o `./dist`

![Sobrescreva a configuração do Vercel](../images/vercel-configuration.png)

Depois que o seu projeto for importado, todos os pushs que forem realizados irão gerar um Deploy Preview, e todas as mudanças na Branch de Produção (geralmente "main") irá resultar em um Deploy de Produção.

Uma vez realizado o deploy, você irá receber uma URL para visualizar sua aplicação online, como a seguinte: https://vite.vercel.app

## Azure Static Web Apps

Você pode rapidamente realizar o deploy da sua aplicação Vite com o serviço Microsoft Azure [Static Web Apps](https://aka.ms/staticwebapps). Você vai precisarÇ

- Uma conta Azure e uma subscription key. Você pode criar uma [conta Azure gratuita aqui](https://azure.microsoft.com/free). 
- Sua aplicação hospedada no [GitHub](https://github.com).
- A [Extensão SWA](https://marketplace.visualstudio.com/items?itemName=ms-azuretools.vscode-azurestaticwebapps) no [Visual Studio Code](https://code.visualstudio.com).

Instale a extensão no VS Code e navegue até a raiz do seu projeto. Abra a extensão Static Web Apps, faça o login na Azure, e clique no '+' para criar um novo Static Web App. Você deverá selecionar a subscription key que será usada.

Siga os passos iniciados pela extensão que você definiu o nome da aplicação, escolhe o framework predefinido, defina a raiz da aplicação (geralmente `/`) e aponte a localização dos arquivos de build `/dist`. Será executada e criada uma Github action no seu repositório com a pasta `.github`.

A Github Action irá trabalhar no deploy da sua aplicação (acompanhe o progresso na aba de Actions do seu repositório), e quando completa com sucesso, você pode visualizar o seu site no endereço configurado na extensão ao clicar no botão 'Browse Website' que irá aparecer quando a Github Action será executada.
