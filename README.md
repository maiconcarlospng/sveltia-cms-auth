# Autenticador do Sveltia CMS

Este projeto é um pequeno serviço de autenticação executado no [Cloudflare Workers](https://workers.cloudflare.com/). Ele permite que usuários do [Sveltia CMS](https://sveltiacms.app/en/) entrem usando o GitHub ou o GitLab.

No site do GPENSC, esse serviço é utilizado como intermediário de autenticação entre o painel administrativo do Sveltia CMS e o GitHub. Ele processa o fluxo OAuth e devolve a autorização ao painel; não é o próprio painel administrativo nem o site público.

> [!IMPORTANT]
> **Na maioria dos casos, você não precisa deste autenticador.** O Sveltia CMS oferece diferentes métodos de autenticação para GitHub e GitLab. Escolha o mais adequado ao projeto antes de configurar este cliente OAuth.

> [!IMPORTANT]
> Embora seja compatível com o Netlify CMS (hoje Decap CMS), este autenticador foi desenvolvido para instalações modernas do Sveltia CMS. Consulte a [documentação do Sveltia CMS](https://sveltiacms.app/en/docs/successor-to-netlify-cms) para conhecer as diferenças entre os produtos e entender por que o Decap CMS não é recomendado para projetos novos.

## Quando usar

### Você não precisa dele se…

- **Está migrando do Netlify CMS ou do Decap CMS**
  - Se o site está hospedado no Netlify, você pode continuar usando o cliente OAuth integrado do Netlify com o Sveltia CMS, sem alterar a configuração do CMS.
  - Se já utiliza outro [cliente OAuth de terceiros](https://decapcms.org/docs/external-oauth-clients/), pode continuar usando-o com o Sveltia CMS sem modificar a configuração.
- **Está usando o GitHub**
  - Se somente você ou pessoas com conhecimentos técnicos utilizarão o CMS, prefira o [método de token de acesso](https://sveltiacms.app/en/docs/backends/github#access-token).
- **Está usando o GitLab**
  - Se somente você ou pessoas com conhecimentos técnicos utilizarão o CMS, prefira o [método de token de acesso](https://sveltiacms.app/en/docs/backends/gitlab#access-token).
  - Se pessoas sem conhecimentos técnicos precisarem entrar no CMS, use a [autorização PKCE no lado do cliente](https://sveltiacms.app/en/docs/backends/gitlab#pkce-authorization).

### Você pode precisar dele se…

- **Está usando o GitHub e pessoas sem conhecimentos técnicos precisam entrar no CMS**
  - Este autenticador oferece uma experiência mais simples do que o método de token de acesso, que exige que cada usuário crie e gerencie seu próprio token de acesso pessoal.

> [!NOTE]
> Usuários do GitLab não precisam deste autenticador. O suporte ao GitLab é incluído aqui apenas para fins de completude.

## Como usar

### Etapa 1. Implante este projeto no Cloudflare Workers

Crie uma conta no Cloudflare e use o botão abaixo para começar a implantação.

[![Implantar no Cloudflare Workers](https://deploy.workers.cloudflare.com/button)](https://deploy.workers.cloudflare.com/?url=https://github.com/maiconcarlospng/sveltia-cms-auth)

Como alternativa, clone o repositório e execute [o comando wrangler deploy](https://developers.cloudflare.com/workers/wrangler/commands/#deploy) localmente.

Depois da implantação, abra o painel do Cloudflare, selecione o serviço `sveltia-cms-auth` e localize a URL do Worker, que terá um formato semelhante a `https://sveltia-cms-auth.<SUBDOMINIO>.workers.dev`. Copie essa URL para utilizá-la nas etapas 2 e 4.

### Etapa 2. Registre o Worker como um aplicativo OAuth

#### GitHub

[Registre um novo aplicativo OAuth no GitHub](https://github.com/settings/applications/new) seguindo as [instruções oficiais](https://docs.github.com/en/apps/oauth-apps/building-oauth-apps/creating-an-oauth-app). Use a URL do Worker obtida na etapa 1.

Preencha os campos assim:

- **Nome do aplicativo:** `Sveltia CMS Authenticator` (ou outro nome de sua preferência)
- **URL da página inicial:** `https://github.com/maiconcarlospng/sveltia-cms-auth` (ou outra URL adequada)
- **Descrição do aplicativo:** opcional
- **URL de retorno da autorização (Authorization callback URL):** `<URL_DO_SEU_WORKER>/callback`

Depois de registrar o aplicativo, clique em **Generate a new client secret** (Gerar um novo segredo do cliente). O GitHub exibirá o **Client ID** e o **Client Secret**, que serão usados na etapa 3.

#### GitLab

[Registre um novo aplicativo OAuth no GitLab](https://gitlab.com/-/user_settings/applications), seguindo as [instruções oficiais](https://docs.gitlab.com/ee/integration/oauth_provider.html#create-a-user-owned-application). Use a URL do Worker obtida na etapa 1.

Preencha os campos assim:

- **Nome:** `Sveltia CMS Authenticator` (ou outro nome de sua preferência)
- **URI de redirecionamento:** `<URL_DO_SEU_WORKER>/callback`
- **Confidencial:** Sim
- **Escopos (Scopes):** somente `api`

Depois de registrar o aplicativo, o GitLab exibirá o **Application ID** e o **Secret**, que serão usados na etapa 3.

### Etapa 3. Configure o Worker

Volte à página do serviço `sveltia-cms-auth` no painel do Cloudflare. Acesse **Settings** (Configurações) > **Variables** (Variáveis) e adicione as variáveis de ambiente a seguir. Consulte também a [documentação do Cloudflare sobre variáveis de ambiente](https://developers.cloudflare.com/workers/platform/environment-variables/#environment-variables-via-the-dashboard).

#### GitHub

- `GITHUB_CLIENT_ID`: o **Client ID** obtido na etapa 2.
- `GITHUB_CLIENT_SECRET`: o **Client Secret** obtido na etapa 2. Clique em **Encrypt** (Criptografar) para ocultar o valor.
- `GITHUB_HOSTNAME`: necessário apenas se você utiliza o GitHub Enterprise Server. Valor padrão: `github.com`.
- `GITHUB_ALLOW_WORKFLOW_SCOPE`: opcional. Defina como `true` para permitir que o site solicite o escopo `workflow` por meio da opção `auth_scope` do Sveltia CMS. Isso permite que o CMS mantenha sincronizado o fork de um colaborador do [Open Authoring](https://sveltiacms.app/en/docs/workflows/open#authentication-scope) quando os arquivos de workflow do GitHub Actions do site forem alterados. Esse escopo permite modificar workflows em todos os repositórios nos quais o usuário pode enviar alterações; por segurança, ele não é solicitado por padrão.

#### GitLab

- `GITLAB_CLIENT_ID`: o **Application ID** obtido na etapa 2.
- `GITLAB_CLIENT_SECRET`: o **Secret** obtido na etapa 2. Clique em **Encrypt** (Criptografar) para ocultar o valor.
- `GITLAB_HOSTNAME`: necessário apenas se você utiliza uma instalação própria do GitLab. Valor padrão: `gitlab.com`.

#### GitHub e GitLab

- `ALLOWED_DOMAINS`: opcional, mas fortemente recomendado. Define os domínios dos sites autorizados a usar o Worker, por exemplo, `www.exemplo.com`.
  - Para informar vários domínios, separe-os por vírgula: `www.exemplo.com, www.exemplo.org`.
  - Você pode usar um curinga (`*`) para corresponder a subdomínios. Por exemplo, `*.exemplo.com` corresponde a `www.exemplo.com`, `blog.exemplo.com` e `docs.api.exemplo.com`, mas não a `exemplo.com`.
  - Para permitir o domínio principal e seus subdomínios, use `exemplo.com, *.exemplo.com`.
  - A lista tem duas funções: impede que outros sites utilizem o seu Worker sem autorização, consumindo seus recursos, e evita que obtenham um token de acesso por meio dele. O autenticador só entrega o token a uma página servida por um dos domínios autorizados.

> [!IMPORTANT]
> A variável `ALLOWED_DOMAINS` é opcional, mas é uma medida de segurança e de prevenção contra abusos. **Recomendamos fortemente configurá-la** com o domínio do seu site.

Salve as variáveis e implante o Worker.

### Etapa 4. Atualize a configuração do CMS

Abra o arquivo `admin/config.yml`, localmente ou no GitHub, e adicione a propriedade `base_url` dentro de `backend`, utilizando a URL do Worker obtida na etapa 1:

~~~diff
 backend:
   name: github # ou gitlab
   repo: usuario/repositorio
   branch: main
+  base_url: <URL_DO_SEU_WORKER>
~~~

Salve e envie a alteração para o repositório. Depois que a nova configuração for publicada, você poderá entrar remotamente no Sveltia CMS usando sua conta do GitHub ou do GitLab.

## Perguntas frequentes

### Por que preciso configurar esse serviço?

Tecnicamente, seria possível hospedar o Sveltia CMS Authenticator em um servidor próprio do projeto e permitir que qualquer pessoa o utilizasse, como o Netlify faz. O custo provavelmente seria baixo, pois se trata de um script pequeno e de curta duração. Porém, manter um serviço desse tipo envolve responsabilidades legais, de privacidade e de segurança que o projeto original não pode assumir.

O Sveltia CMS é um projeto pessoal de [@kyoshino](https://github.com/kyoshino). Por isso, o autenticador não é oferecido como um serviço hospedado para todos (SaaS): quem precisa dele deve implantá-lo e mantê-lo por conta própria.

## Agradecimentos

Este projeto foi inspirado no [netlify-cms-oauth-firebase](https://github.com/Herohtar/netlify-cms-oauth-firebase).
