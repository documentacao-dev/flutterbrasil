---
ia-translate: true
title: Compilar e lançar um app web
description: Como preparar e lançar um app web.
shortTitle: Web
---

Durante um ciclo típico de desenvolvimento,
você testa um app usando `flutter run -d chrome`
(por exemplo) na linha de comando.
Isso compila uma versão _debug_ do seu app.

Esta página ajuda você a preparar uma versão _release_
do seu app e cobre os seguintes tópicos:

* [Compilando o app para release](#building-the-app-for-release)
* [Hashing de conteúdo web e cache HTTP](#web-content-hashing)
* [Deploy de WebAssembly em produção](#deploy-webassembly-in-production)
* [Deploy para a web](#deploying-to-the-web)
* [Deploy para Firebase Hosting](#deploying-to-firebase-hosting)
* [Manipulando imagens na web](#handling-images-on-the-web)
* [Minificação](#minification)
* [Viewport de navegadores móveis e SafeArea](#mobile-browser-viewport-and-safearea)
* [Incorporando um app Flutter em uma página HTML][embed-html]

[embed-html]: #embedding-a-flutter-app-into-an-html-page

## Compilando o app para release {:#building-the-app-for-release}

Compile o app para deployment usando o comando `flutter build web`.

```console
flutter build web
```

Para compilar sua aplicação com WebAssembly (Wasm), passe a flag `--wasm`:

```console
flutter build web --wasm
```

Isso
gera o app, incluindo os assets, e coloca os arquivos no
diretório `/build/web` do projeto.

Para validar o build release do seu app,
inicie um servidor web (por exemplo,
`python -m http.server 8000`,
ou usando o package [dhttpd][]),
e abra o diretório /build/web. Navegue até
`localhost:8000` no seu navegador
(dado o exemplo python SimpleHTTPServer)
para visualizar a versão release do seu app.

## Flags de build adicionais {:#additional-build-flags}

Passe `--release` (padrão), `--profile` ou `--debug` para `flutter build web`
dependendo de você estar fazendo deploy para produção, fazendo profiling no Chrome
DevTools ou testando com assertions:

| Modo de build | WebAssembly (`--wasm`) | JavaScript (padrão) | Finalidade |
| :--- | :--- | :--- | :--- |
| `--release` (padrão) | `-O2` (minificado, símbolos removidos) | `-O4` (minificado) | Deploy em produção |
| `--profile` | `-O2 --no-strip-wasm` (nomes legíveis de funções e tipos) | `-O4` (símbolos não minificados) | Profiling de performance e memória |
| `--debug` | `-O0` (assertions habilitadas) | `-O1` (assertions habilitadas) | Testes de desenvolvimento |

{:.table .table-striped}

Para sobrescrever o nível de otimização padrão do compilador, passe
`--optimization-level=LEVEL` (`-O0` a `-O4` para `dart2wasm`; `-O1`
a `-O4` para `dart2js`).
Para detalhes de otimização do `dart2wasm`,
consulte [Compilação Dart WebAssembly][dart-wasm].

### Source maps

Para gerar source maps para simbolização em serviços de relatório de falhas,
passe a flag `--source-maps`:

```console
flutter build web --source-maps
```

Gerar source maps cria arquivos `.js.map` ou `.wasm.map` separados
(ou ambos, dependendo do seu alvo de build) no diretório `build/web`
sem adicionar sobrecarga binária à saída compilada.

:::warning Exclua source maps da hospedagem pública
Por padrão, ferramentas padrão de deploy web fazem upload de todos os arquivos em `build/web/`.
Se os source maps forem publicados publicamente, o DevTools do navegador
os baixará automaticamente, expondo nomes de símbolos não minificados,
hierarquias de classes e caminhos de arquivos locais.

Faça upload dos arquivos `.map` para seu serviço privado de monitoramento de erros
(como Sentry ou Datadog) durante o CI/CD, e exclua os arquivos `*.map`
da hospedagem web pública (por exemplo, usando `"ignore": ["**/*.map"]`
no `firebase.json` ou removendo-os antes do deploy).
:::

### Hashing de conteúdo web (--web-content-hash) {:#web-content-hashing}

:::version-note
O hashing de conteúdo web (`--web-content-hash`) requer Flutter 3.50 ou posterior.
:::

Para habilitar o cache busting automático baseado em conteúdo entre deploys, passe
`--web-content-hash` para `flutter build web`:

```console
flutter build web --wasm --web-content-hash
```

<<<<<<< pt
Quando `--web-content-hash` está habilitado, o Flutter incorpora um hash de conteúdo
SHA-256 truncado em 8 caracteres nos entrypoints compilados e em todos os arquivos sob
`build/web/assets/**`, e registra os caminhos com hash em `flutter_bootstrap.js`
(`_flutter.buildConfig`):
||||||| base_en
When `--web-content-hash` is enabled, Flutter embeds an 8-character truncated
SHA-256 content hash into compiled entrypoints and all files under
`build/web/assets/**`, and records the hashed paths in `flutter_bootstrap.js`
(`_flutter.buildConfig`):
=======
When you pass `--web-content-hash`, Flutter inserts an eight-character
truncated SHA-256 content hash (`HASH`) before the file extension of compiled
entrypoints and every file under `build/web/assets/**`.
For example, `main.dart.js` becomes `main.dart.HASH.js`, `logo.png` becomes
`logo.HASH.png`, and the extensionless `NOTICES` file becomes `NOTICES.HASH`.
The flag works with release, profile, and debug builds,
and with both JavaScript-only and `--wasm` builds.

Flutter records the hashed entrypoint and asset manifest filenames in the
`_flutter.buildConfig` object.
Flutter writes this object to `flutter_bootstrap.js`, and also to `index.html`
when `index.html` inlines
`{% raw %}{{flutter_bootstrap_js}}{% endraw %}` or
`{% raw %}{{flutter_build_config}}{% endraw %}`.
The hashed asset manifests then map each asset key to its hashed filename,
so `rootBundle`, `Image.asset`, and fonts resolve hashed assets automatically.
>>>>>>> new_en

| Grupo de artefatos | Arquivos em `build/web` | `Cache-Control` recomendado |
| :--- | :--- | :--- |
<<<<<<< pt
| **Arquivos imutáveis com hash** | `main.dart.<hash>.js`, `main.dart.<hash>.wasm`, `main.dart.<hash>.mjs` (além de `.js.map` e `.mjs.map`), e todos os arquivos em `assets/**` (`AssetManifest.<hash>.bin`, `FontManifest.<hash>.json`, `NOTICES.<hash>`, fontes, imagens e `shaders/**`) | `public, max-age=31536000, immutable` |
| **Arquivos de bootstrap e raiz sem hash** | `index.html`, `flutter_bootstrap.js`, `flutter.js`, `manifest.json`, `version.json`, `404.html`, `icons/**`, `canvaskit/**` empacotados (com `--no-web-resources-cdn`) e `main.dart.wasm.map` (com `--source-maps`) | `no-cache` (ou `max-age=0, must-revalidate`) |
||||||| base_en
| **Hashed immutable files** | `main.dart.<hash>.js`, `main.dart.<hash>.wasm`, `main.dart.<hash>.mjs` (plus `.js.map` and `.mjs.map`), and all files under `assets/**` (`AssetManifest.<hash>.bin`, `FontManifest.<hash>.json`, `NOTICES.<hash>`, fonts, images, and `shaders/**`) | `public, max-age=31536000, immutable` |
| **Unhashed bootstrap and root files** | `index.html`, `flutter_bootstrap.js`, `flutter.js`, `manifest.json`, `version.json`, `404.html`, `icons/**`, bundled `canvaskit/**` (with `--no-web-resources-cdn`), and `main.dart.wasm.map` (with `--source-maps`) | `no-cache` (or `max-age=0, must-revalidate`) |
=======
| **Hashed immutable files** | `main.dart.HASH.js`, `main.dart.HASH.wasm`, `main.dart.HASH.mjs`, their `.js.map` and `.mjs.map` source maps (if you host them), and all files under `assets/**` (including `AssetManifest.HASH.bin`, `AssetManifest.bin.HASH.json`, `FontManifest.HASH.json`, `NOTICES.HASH`, fonts, images, `shaders/**`, and `packages/**`) | `public, max-age=31536000, immutable` |
| **Unhashed bootstrap and root files** | `index.html`, `flutter_bootstrap.js`, `flutter.js`, `flutter.js.map`, `manifest.json`, `version.json`, `favicon.png`, `icons/**`, other files copied from `web/` (such as `404.html`), `flutter_service_worker.js`, `precache_manifest.json`, `canvaskit/**`, and `main.dart.wasm.map` (with `--source-maps`) | `no-cache` (or `max-age=0, must-revalidate`) |
>>>>>>> new_en

{:.table .table-striped}

<<<<<<< pt
Para um exemplo completo de `firebase.json`, consulte o [FAQ da Web][web-faq-caching].
||||||| base_en
For a complete `firebase.json` example, refer to the [Web FAQ][web-faq-caching].
=======
By default, the app loads CanvasKit and Skwasm (`canvaskit.wasm`, `skwasm.wasm`,
and related files) from the `www.gstatic.com` CDN under a URL that is versioned
by the Flutter engine revision.
Flutter still copies an unhashed `canvaskit/` directory into `build/web`,
which the app uses only when you build with `--no-web-resources-cdn`.
Because those files keep the same names across Flutter upgrades,
don't serve your own copy of `canvaskit/**` with `immutable` caching.

:::important
When you build with `--web-content-hash`, Flutter renames files under
`build/web/assets/` on disk.
Code that references them by a fixed URL, such as `assets/...` paths in
`index.html`, CSS, or a JavaScript `fetch()` call, receives a `404 Not Found`
status code.
Load assets through `rootBundle` or the asset APIs instead.
:::

Running `flutter build web --web-content-hash` fails with an error in any of
the following cases:

* The `web/index.html` file or a custom `web/flutter_bootstrap.js` file
  references `main.dart.js` or calls the deprecated `loadEntrypoint()` method.
* The `web/index.html` file neither loads nor inlines `flutter_bootstrap.js`.
* A custom `web/flutter_bootstrap.js` file omits the
  `{% raw %}{{flutter_build_config}}{% endraw %}` token.

To migrate an older project, run `flutter create . --platforms web`
or follow [Flutter web app initialization][web-init-bootstrapping].

For a complete `firebase.json` example, refer to the [Web FAQ][web-faq-caching].
>>>>>>> new_en

:::note
`--web-content-hash` ainda não suporta imports adiados (`deferred as`) em
builds JavaScript (`dart2js`) nem WebAssembly (`dart2wasm` /
`--enable-wasm-deferred-loading`).
:::

#### Deploys sobrepostos para abas ativas do navegador {:#overlay-deployments-for-active-browser-tabs}

Quando você faz deploy de uma nova versão (`N`), usuários com uma aba do navegador já aberta
executando a versão anterior (`N-1`) podem buscar sob demanda um asset, fonte ou
imagem cujo nome de arquivo carrega o hash de conteúdo da versão `N-1`.

Para evitar erros `404` no meio da sessão durante rollouts, use uma estratégia de
**deploy sobreposto** (overlay deployment):

* Mantenha pelo menos os arquivos `main.dart.*` e `assets/**` com hash da versão
  anterior (`N-1`) no seu CDN ou bucket de armazenamento junto com os arquivos
  com hash da versão `N` (ou remova hashes antigos após uma janela de tolerância de 24–72 horas).
* Atualize `index.html` e `flutter_bootstrap.js` atomicamente por último.
* Exclua `/assets/**`, `/canvaskit/**`, `/icons/**` e `/main.dart.*` das
  regras de rewrite catch-all para `/index.html` de single-page applications (SPA),
  para que assets ausentes retornem um `404` genuíno em vez de `200 text/html`.

#### Service workers personalizados (`precache_manifest.json`) {:#custom-service-workers-precache-manifest-json}

Quando `--web-content-hash` está habilitado, `flutter build web` grava uma lista de
materiais em `build/web/precache_manifest.json`:

```json title="build/web/precache_manifest.json"
{
  "version": 1,
  "entries": [
    {
      "url": "flutter_bootstrap.js",
      "hash": "a1b2c3d4",
      "size": 10432,
      "urlHashed": false
    },
    {
      "url": "main.dart.f215ce97.wasm",
      "hash": "f215ce97",
      "size": 1842104,
      "urlHashed": true
    }
  ]
}
```

Service workers de PWA personalizados ou pipelines do [Workbox][] podem consumir
`precache_manifest.json` diretamente (passando `revision: entry.urlHashed ? null : entry.hash`
para que arquivos com hash na URL não sejam armazenados em cache duas vezes com parâmetros de query).

Each `url` value is a path relative to `build/web`, without a leading `/` or
`--base-href` prefix, and the manifest sorts entries by `url`.
The manifest omits dotfiles, `flutter_service_worker.js`, source maps
(`*.map`), `*.symbols` files, and `*.info.json` files.
When you build with `--no-web-resources-cdn`, the manifest includes
`canvaskit/**` files with `"urlHashed": false`; otherwise, it omits them.

<FragmentTarget id="opting-in-to-webassembly" />

## Deploy de WebAssembly em produção {:#deploy-webassembly-in-production}

Executar `flutter build web --wasm` gera um bundle de release com dois alvos
em `build/web`:

* **Alvo WebAssembly (`dart2wasm`)**:
  gera `main.dart.wasm` e seu módulo de runtime JS `main.dart.mjs`,
  usado junto com o motor de renderização WebAssembly do Flutter.
* **Fallback em JavaScript (`dart2js`)**:
  gera `main.dart.js` para navegadores sem os recursos Wasm necessários.

Na inicialização, o `flutter_bootstrap.js` verifica os recursos WebAssembly do navegador
e recorre automaticamente ao `main.dart.js` quando o navegador não suporta
os recursos Wasm necessários ou não está em `wasmAllowList`.
Para detalhes de compatibilidade dos navegadores,
consulte [Suporte para WebAssembly (Wasm)][Wasm support].
Para as flags do compilador `dart2wasm` e a configuração genérica de servidor Wasm,
consulte [Compilação Dart WebAssembly][dart-wasm] e
[Servir WebAssembly em produção][dart-wasm-serving].

### Configurar cabeçalhos do servidor e compressão {:#server-headers-and-compression}

Configure o servidor web de produção ou o CDN com os seguintes cabeçalhos e
configurações de compressão:

* **Isolamento de origem cruzada para renderização multithread**:
  retorne `Cross-Origin-Opener-Policy: same-origin` e
  `Cross-Origin-Embedder-Policy: credentialless` (ou `require-corp`) na
  resposta do documento para habilitar `SharedArrayBuffer` e a renderização WebAssembly
  multithread (consulte o [guia de COOP e COEP no web.dev][coep]).
* **Tipo MIME de Wasm e pré-compressão estática**:
  sirva todos os arquivos `.wasm` (tanto `main.dart.wasm` quanto os binários do motor
  hospedados por você) com `Content-Type: application/wasm`, e pré-comprima arquivos `.wasm`,
  `.mjs`, `.js`, fontes e manifestos em `assets/**` com Brotli (`br`) e Gzip
  (`gzip`).
  Para detalhes de compilação em streaming e compressão,
  consulte [Servir WebAssembly em produção][dart-wasm-serving].

### Reduzir a latência de inicialização a frio {:#reduce-cold-start-latency}

Para diminuir o tempo até o primeiro frame em carregamentos com cache frio:

* **Embutir o script de bootstrap**:
  substitua a tag externa `<script src="flutter_bootstrap.js" async></script>` em
  `web/index.html` por um `<script>` inline contendo
  `{% raw %}{{flutter_bootstrap_js}}{% endraw %}`.
  Embutir elimina uma viagem de ida e volta extra na rede antes de o navegador começar
  a baixar `main.dart.wasm` e o motor de renderização.
  Para detalhes da sintaxe de template,
  consulte [Inicializar o app][web-init-bootstrapping].
* **Reduzir o peso inicial do bundle**:
  builds de release habilitam tree shaking de ícones de fonte automaticamente
  (passe `--no-tree-shake-icons` apenas ao depurar code points de ícones dinâmicos).
  Para dividir recursos grandes em chunks `.wasm` carregados sob demanda,
  compile com a flag `--enable-wasm-deferred-loading`.
  Lembre que `--enable-wasm-deferred-loading` é incompatível com
  `--web-content-hash`.
* **Renderizar uma tela de carregamento em HTML**:
  inclua HTML e CSS leves dentro de `web/index.html`
  (ou atualize o estado do DOM pelo callback `onEntrypointLoaded` em
  `flutter_bootstrap.js`) para que os usuários recebam feedback visual enquanto o
  motor é inicializado.

### Servir assets de um CDN ou origem própria {:#multi-asset-and-cdn-loading}

Personalize de onde o motor, os módulos compilados e os assets são carregados para
combinar com a arquitetura de hospedagem e os requisitos de privacidade:

* **Hospedar os binários do motor e as fontes de fallback**:
  passe `--no-web-resources-cdn` para empacotar os arquivos do motor de renderização do Flutter
  diretamente em `build/web/canvaskit/`, em vez de buscá-los em
  `gstatic.com` em tempo de execução.
  Ao compilar com a flag `--web-content-hash`, os arquivos empacotados em
  `canvaskit/**` continuam sem hash e exigem cabeçalhos de cache com revalidação.
* **Servir assets estáticos de um CDN dedicado**:
  configure as opções `assetBase` e `entrypointBaseUrl` em
  `_flutter.loader.load({ config: ... })` (e `canvasKitBaseUrl` ao hospedar os
  binários do motor em um CDN).
  Para injetar um prefixo de CDN em `web/index.html` no momento do build, referencie o
  placeholder `$FLUTTER_STATIC_ASSETS_URL` em `web/index.html` e passe
  `--static-assets-url=https://cdn.example.com/` (incluindo a `/` final)
  para `flutter build web`.
  Ao compilar com a flag `--enable-wasm-deferred-loading` e roteamento de assets
  personalizado, forneça um callback `wasmDeferredModulesLoader` para buscar os
  módulos `.wasm` adiados no CDN.
  Para detalhes de configuração,
  consulte [Personalizar o loader do Flutter][web-init-loader].

## Deploy para a web {:#deploying-to-the-web}

Quando estiver pronto para fazer deploy do seu app,
faça upload do bundle release
para Firebase, nuvem, ou um serviço similar.
Aqui estão algumas possibilidades, mas há
muitas outras:

* [Firebase Hosting][]
* [GitHub Pages][]
* [Google Cloud Hosting][]

## Deploy para Firebase Hosting {:#deploying-to-firebase-hosting}

<<<<<<< pt
Você pode usar o Firebase CLI para compilar e lançar seu app Flutter com Firebase
Hosting.
||||||| base_en
You can use the Firebase CLI to build and release your Flutter app with Firebase
Hosting.
=======
You can use the Firebase CLI to deploy your Flutter web app to Firebase Hosting.
>>>>>>> new_en

### Antes de começar {:#before-you-begin}

Para começar, [instale ou atualize][install-firebase-cli] o Firebase CLI:

```console
npm install -g firebase-tools
```

<<<<<<< pt
### Inicializar Firebase {:#initialize-firebase}
||||||| base_en
### Initialize Firebase
=======
<FragmentTarget id="initialize-firebase" />
>>>>>>> new_en

<<<<<<< pt
1. Habilite o preview de web frameworks para o [CLI framework-aware do Firebase][Firebase framework-aware CLI]:
||||||| base_en
1. Enable the web frameworks preview to the [Firebase framework-aware CLI][]:
=======
### Initialize and deploy
>>>>>>> new_en

<<<<<<< pt
    ```console
    firebase experiments:enable webframeworks
    ```

2. Em um diretório vazio ou um projeto Flutter existente, execute o comando de
inicialização:
||||||| base_en
    ```console
    firebase experiments:enable webframeworks
    ```

2. In an empty directory or an existing Flutter project, run the initialization
command:
=======
1.  In the root directory of your Flutter project,
    run the initialization command:
>>>>>>> new_en

    ```console
    firebase init hosting
    ```

<<<<<<< pt
3. Responda `yes` quando perguntado se você quer usar um web framework.
||||||| base_en
3. Answer `yes` when asked if you want to use a web framework.
=======
1.  Follow the CLI prompts:
>>>>>>> new_en

<<<<<<< pt
4. Se você está em um diretório vazio,
    você será perguntado para escolher seu web framework. Escolha `Flutter Web`.
||||||| base_en
4. If you're in an empty directory,
    you'll be asked to choose your web framework. Choose `Flutter Web`.
=======
    * Select an existing Firebase project or create a new project.
    * When asked `What do you want to use as your public directory?`,
      enter `build/web`.
    * Choose whether to configure the site as a single-page application
      (rewriting all URLs to `/index.html`).
    * Choose whether to set up automatic builds and deploys with GitHub.
    * If `build/web/index.html` already exists,
      answer `no` when asked whether to overwrite it.
>>>>>>> new_en

<<<<<<< pt
5. Escolha seu diretório fonte de hospedagem; isso pode ser um app flutter existente.
||||||| base_en
5. Choose your hosting source directory; this could be an existing flutter app.
=======
1.  Build your web app for release:
>>>>>>> new_en

<<<<<<< pt
6. Selecione uma região para hospedar seus arquivos.
||||||| base_en
6. Select a region to host your files.
=======
    ```console
    flutter build web
    ```
>>>>>>> new_en

<<<<<<< pt
7. Escolha se deseja configurar builds e deploys automáticos com GitHub.
||||||| base_en
7. Choose whether to set up automatic builds and deploys with GitHub.
=======
    :::tip
    To run `flutter build web` automatically whenever you deploy,
    add `"predeploy": ["flutter build web"]` to the `"hosting"` section
    of your `firebase.json` file.
    For recommended `Cache-Control` headers and
    single-page application rewrite rules in `firebase.json`,
    check out the [Web FAQ][web-faq-caching].
    :::
>>>>>>> new_en

<<<<<<< pt
8. Faça deploy do app para Firebase Hosting:
||||||| base_en
8. Deploy the app to Firebase Hosting:
=======
1.  Deploy the release bundle to Firebase Hosting:
>>>>>>> new_en

    ```console
    firebase deploy
    ```

<<<<<<< pt
    Executar este comando automaticamente executa `flutter build web --release`,
    então você não precisa compilar seu app em um passo separado.

Para saber mais, visite a documentação oficial do [Firebase Hosting][] para
Flutter na web.
||||||| base_en
    Running this command automatically runs `flutter build web --release`,
    so you don't have to build your app in a separate step.

To learn more, visit the official [Firebase Hosting][] documentation for
Flutter on the web.
=======
To learn more, visit the official [Firebase Hosting][] documentation.
>>>>>>> new_en

## Manipulando imagens na web {:#handling-images-on-the-web}

A web suporta o widget `Image` padrão para exibir imagens.
Por design, navegadores web executam código não confiável sem prejudicar o computador host.
Isso limita o que você pode fazer com imagens comparado a plataformas mobile e desktop.

Para saber mais, consulte [Exibindo imagens na web][Displaying images on the web].

## Minificação {:#minification}

Para melhorar o start-up do app, o compilador reduz o tamanho do código compilado
removendo código não utilizado (conhecido como _tree shaking_), e renomeando símbolos de código para
strings mais curtas (por exemplo, renomeando `AlignmentGeometryTween` para algo como
`ab`). Quais dessas duas otimizações são aplicadas depende do modo de build:

| Tipo de build de app web | Código minificado? | Tree shaking executado? |
|-----------------------|----------------|-------------------------|
| debug                 | Não             | Não                      |
| profile               | Não             | Sim                     |
| release               | Sim            | Sim                     |

{:.table .table-striped}

## Viewport de navegadores móveis e SafeArea {:#mobile-browser-viewport-and-safearea}

:::version-note
O suporte a insets de `SafeArea` com `viewport-fit=cover` e a preservação da
codificação percentual de URLs no histórico do navegador requerem Flutter 3.50 ou posterior.
:::

Em navegadores móveis (como o Safari do iOS e o Chrome do Android) com recortes de tela,
notches ou indicadores de início, o Flutter web por padrão renderiza dentro da
área segura do navegador (`viewport-fit=contain`).

Para renderizar de borda a borda em toda a tela e preencher o [`SafeArea`][]
e o [`MediaQuery.paddingOf`][] do Flutter com os insets CSS
`env(safe-area-inset-*)` do navegador, adicione uma tag `<meta name="viewport">` com
`viewport-fit=cover` dentro do `<head>` do seu arquivo `web/index.html` (incluída
como uma opção comentada nos templates `web/index.html` recém-criados):

```html title="web/index.html"
<meta name="viewport" content="width=device-width, initial-scale=1.0, viewport-fit=cover">
```

:::note
Se um web app existente já inclui `viewport-fit=cover` em
`web/index.html`, o Flutter 3.50 e posterior o respeita em vez de removê-lo,
o que faz o app renderizar atrás dos recortes do dispositivo, a menos que os controles de UI estejam
envolvidos em um widget [`SafeArea`][]. Para detalhes de migração, consulte o
[guia da breaking change de SafeArea com `viewport-fit=cover`][viewport-fit-breaking].
:::

### Codificação percentual de URLs no histórico do navegador {:#url-percent-encoding-in-browser-history}

No Flutter 3.50 e posterior, strings de rota passadas para
`SystemNavigator.routeInformationUpdated` preservam sequências codificadas em porcentagem
(como `%2F`, `%26`, `%23` e `%20`) quando gravadas na barra de URL do navegador
e no histórico, em vez de decodificá-las usando `Uri.decodeComponent`.

## Incorporando um app Flutter em uma página HTML {:#embedding-a-flutter-app-into-an-html-page}

Para incorporar o Flutter em uma página web existente,
consulte [Adicione Flutter a qualquer aplicação web][Embedding Flutter web].

[Embedding Flutter web]: /platform-integration/web/embedding-flutter-web

[coep]: https://web.dev/articles/coop-coep
[dart-wasm]: {{site.dart-site}}/web/wasm
[dart-wasm-serving]: {{site.dart-site}}/web/deployment#serving-wasm-in-production
[dhttpd]: {{site.pub}}/packages/dhttpd
[Displaying images on the web]: /platform-integration/web/web-images
[Firebase Hosting]: {{site.firebase}}/docs/hosting
[install-firebase-cli]: {{site.firebase}}/docs/cli#install_the_firebase_cli
[GitHub Pages]: https://pages.github.com/
[give us feedback]: {{site.repo.flutter}}/issues/new?title=%5Bweb%5D:+%3Cdescribe+issue+here%3E&labels=%E2%98%B8+platform-web&body=Describe+your+issue+and+include+the+command+you%27re+running,+flutter_web%20version,+browser+version
[Google Cloud Hosting]: https://cloud.google.com/solutions/web-hosting
[`MediaQuery.paddingOf`]: {{site.api}}/flutter/widgets/MediaQuery/paddingOf.html
[`SafeArea`]: {{site.api}}/flutter/widgets/SafeArea-class.html
[viewport-fit-breaking]: /release/breaking-changes/viewport-fit-cover-safe-area
[Wasm support]: /platform-integration/web/wasm
[web-faq-caching]: /platform-integration/web/faq#how-do-i-configure-my-cache-headers
[web-init-bootstrapping]: /platform-integration/web/initialization#bootstrapping
[web-init-loader]: /platform-integration/web/initialization#customize-the-flutter-loader
[Workbox]: https://github.com/GoogleChrome/workbox
