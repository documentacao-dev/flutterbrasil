---
ia-translate: true
title: Inicialização de app web Flutter
description: Personalize como apps Flutter são inicializados na web.
---

Esta página detalha o processo de inicialização para apps web Flutter e
como ele pode ser personalizado.

## Bootstrap the app {:#bootstrapping}

O comando `flutter build web` produz
um script chamado `flutter_bootstrap.js` no
diretório de saída da build (`build/web`).
Este arquivo contém o código JavaScript necessário para inicializar e
executar seu app Flutter.
Você pode usar este script colocando uma tag async-script para ele em
seu arquivo `index.html` no subdiretório `web` do seu app Flutter:

```html highlightLines=3
<html>
  <body>
    <script src="flutter_bootstrap.js" async></script>
  </body>
</html>
```

Alternativamente, você pode incorporar todo o conteúdo do
arquivo `flutter_bootstrap.js` inserindo o
token de template `{% raw %}{{flutter_bootstrap_js}}{% endraw %}` no
seu arquivo `index.html`:

```html highlightLines=4
<html>
  <body>
    <script>
      {% raw %}{{flutter_bootstrap_js}}{% endraw %}
    </script>
  </body>
</html>
```

O token `{% raw %}{{flutter_bootstrap_js}}{% endraw %}` é
substituído pelo conteúdo do arquivo `flutter_bootstrap.js` quando
o arquivo `index.html` é copiado para o
diretório de saída (`build/web`) durante a etapa de build.
Incorporar o script de bootstrap elimina uma ida e volta extra à rede durante a
inicialização.
Para orientações de otimização em produção, consulte
[Reduzir a latência de cold start][cold-start].

[cold-start]: /deployment/web#reduce-cold-start-latency

<FragmentTarget id="customizing-initialization" />

## Personalizar a inicialização {:#customize-initialization}

Por padrão, `flutter build web` gera um arquivo `flutter_bootstrap.js` que
faz uma inicialização simples do seu app Flutter.
No entanto, em alguns cenários, você pode ter uma razão para
personalizar esse processo de inicialização, como:

* Definir uma configuração Flutter personalizada para seu app.
* Integrar um service worker personalizado para suporte offline ou cache.
* Escrever código JavaScript personalizado para
  executar em diferentes estágios do processo de inicialização.

Para escrever sua própria lógica de bootstrapping personalizada em vez de
usar o script padrão produzido pela etapa de build, você pode
colocar um arquivo `flutter_bootstrap.js` no subdiretório `web` do seu projeto,
que é copiado e usado em vez do
script padrão produzido pela build.
Este arquivo também é um template, e você pode inserir vários tokens especiais que
a etapa de build substitui em tempo de build ao copiar
o arquivo `flutter_bootstrap.js` para o diretório de saída.
A tabela a seguir lista os tokens que a etapa de build irá
substituir nos arquivos `flutter_bootstrap.js` ou `index.html`:

| Token | Substituído por |
|---|---|
| `{% raw %}{{flutter_js}}{% endraw %}` | O código JavaScript que torna o objeto `FlutterLoader` disponível na variável global `_flutter.loader` (consulte [Personalizar o Flutter loader](#customize-the-flutter-loader)). |
| `{% raw %}{{flutter_build_config}}{% endraw %}` | Uma instrução JavaScript que define metadados produzidos pelo processo de build que dá ao `FlutterLoader` informações necessárias para fazer bootstrap adequadamente da sua aplicação. |
| `{% raw %}{{flutter_service_worker_version}}{% endraw %}` | **Descontinuado.** Anteriormente substituído por um número de versão de build para o carregador do service worker legado do Flutter. O Flutter não gera nem registra mais um service worker de cache; consulte [Descontinuação do service worker](#service-worker-version-deprecation) e o [FAQ da Web](/platform-integration/web/faq#how-do-i-configure-a-service-worker). |
| `{% raw %}{{flutter_bootstrap_js}}{% endraw %}` | Como mencionado acima, isso incorpora o conteúdo do arquivo `flutter_bootstrap.js` diretamente no arquivo `index.html`. Observe que este token só pode ser usado no `index.html` e não no próprio arquivo `flutter_bootstrap.js`. |

{:.table .table-striped}

<FragmentTarget id="write-a-custom-flutter_bootstrap-js" />

## Escrever um script de bootstrap personalizado {:#custom-bootstrap-js}

Qualquer script `flutter_bootstrap.js` personalizado precisa ter três componentes para
iniciar com sucesso seu app Flutter:

* Um token `{% raw %}{{flutter_js}}{% endraw %}`,
  para tornar `_flutter.loader` disponível.
* Um token `{% raw %}{{flutter_build_config}}{% endraw %}`,
  que fornece informações sobre a build ao
  `FlutterLoader` necessárias para iniciar seu app.
* Uma chamada para `_flutter.loader.load()`, que realmente inicia o app.

O arquivo `flutter_bootstrap.js` mais básico seria algo assim:

```js
{% raw %}{{flutter_js}}{% endraw %}
{% raw %}{{flutter_build_config}}{% endraw %}

_flutter.loader.load();
```

<<<<<<< pt
:::tip Compatibilidade com `--web-content-hash`
Quando você compila com [`flutter build web --web-content-hash`][web-content-hashing],
o Flutter injeta os nomes de arquivo de entrypoint com hash de conteúdo (`mainJsPath`,
`mainWasmPath`, `jsSupportRuntimePath`) e os mapeamentos do manifesto de assets com hash
(`assetManifest`, `fontManifest`, `extraAssets`) em
`{% raw %}{{flutter_build_config}}{% endraw %}`. Arquivos `web/index.html` ou
`web/flutter_bootstrap.js` personalizados devem incluir
`{% raw %}{{flutter_build_config}}{% endraw %}` e chamar `_flutter.loader.load()`
em vez de fixar `"main.dart.js"` no código ou chamar a API descontinuada
`FlutterLoader.loadEntrypoint()`.
||||||| base_en
:::tip Compatibility with `--web-content-hash`
When you build with [`flutter build web --web-content-hash`][web-content-hashing],
Flutter injects the content-hashed entrypoint filenames (`mainJsPath`,
`mainWasmPath`, `jsSupportRuntimePath`) and hashed asset manifest mappings
(`assetManifest`, `fontManifest`, `extraAssets`) into
`{% raw %}{{flutter_build_config}}{% endraw %}`. Custom `web/index.html` or
`web/flutter_bootstrap.js` files must include
`{% raw %}{{flutter_build_config}}{% endraw %}` and call `_flutter.loader.load()`
rather than hardcoding `"main.dart.js"` or calling the deprecated
`FlutterLoader.loadEntrypoint()` API.
=======
:::tip Compatibility with `--web-content-hash`
When you build with [`flutter build web --web-content-hash`][web-content-hashing],
Flutter injects the content-hashed entrypoint filenames (`mainJsPath`,
`mainWasmPath`, `jsSupportRuntimePath`) and hashed asset manifest mappings
(`assetManifest`, `fontManifest`, `extraAssets`) into
`{% raw %}{{flutter_build_config}}{% endraw %}`.
If a custom `web/index.html` or `web/flutter_bootstrap.js` file hardcodes
`main.dart.js` or calls the deprecated `FlutterLoader.loadEntrypoint()` method,
the build fails.
A custom `web/index.html` file must load `flutter_bootstrap.js`, or inline
`{% raw %}{{flutter_bootstrap_js}}{% endraw %}` or
`{% raw %}{{flutter_build_config}}{% endraw %}`.
A custom `web/flutter_bootstrap.js` file must include
`{% raw %}{{flutter_build_config}}{% endraw %}` and call
`_flutter.loader.load()`.
>>>>>>> new_en
:::

[web-content-hashing]: /deployment/web#web-content-hashing

## Personalizar o Flutter loader {:#customize-the-flutter-loader}

A API JavaScript `_flutter.loader.load()` pode ser invocada com
argumentos opcionais para personalizar o comportamento de inicialização:

| Nome                    | Descrição                                                                                                                   | Tipo&nbsp;JS |
|-------------------------|-----------------------------------------------------------------------------------------------------------------------------|--------------|
| `config`                | A configuração Flutter do seu app.                                                                                          | `Object`     |
| `onEntrypointLoaded`    | A função chamada quando o engine está pronto para ser inicializado. Recebe um objeto `engineInitializer` como seu único parâmetro. | `Function`   |

{:.table .table-striped}

O argumento `config` é um objeto que aceita os seguintes campos opcionais:

### Incorporação do app, URLs e depuração {:#app-embedding-urls-and-debugging}

| Nome | Descrição | Tipo&nbsp;JS |
|---|---|---|
| `assetBase` | URL base do diretório `assets` do app quando ele é carregado de outro domínio, subdiretório ou CDN (consulte [Servir assets de um CDN ou origem própria][multi-asset-cdn]). | `string` |
| `debugShowSemanticsNodes` | Renderiza visivelmente a árvore de semântica na tela, para depuração de acessibilidade. | `boolean` |
| `entrypointBaseUrl` | URL base do entrypoint compilado do app Flutter. O padrão é `"/"`. | `string` |
| `fontFallbackBaseUrl` | URL base para fontes de fallback quando as fontes incluídas não têm os glifos necessários. O padrão é `"https://fonts.gstatic.com/s/"`. | `string` |
| `hostElement` | Elemento HTML no qual o Flutter renderiza um app de view única (consulte [Adicionar Flutter a qualquer aplicação web][embedding-flutter-web]). Quando omitido, o Flutter ocupa a página inteira. | `HTMLElement` |
| `multiViewEnabled` | Habilita o modo multi-view para adicionar e remover views incorporadas dinamicamente (consulte [Modo embedded][embedded-mode]). | `boolean` |
| `nonce` | [Nonce de Content Security Policy (CSP)][nonce-mdn] aplicado aos scripts e estilos inline criados pelo engine. | `string` |

{:.table .table-striped}

<FragmentTarget id="forcesinglethreadedskwasm" />

### WebAssembly e seleção de build {:#webassembly-and-build-selection}

| Nome | Descrição | Tipo&nbsp;JS |
|---|---|---|
| `forceSingleThreadedSkwasm` | Força a renderização WebAssembly de thread única quando [WebAssembly][wasm-support] é selecionado em tempo de execução (por exemplo, quando `SharedArrayBuffer` ou os headers `COOP` e `COEP` não estão disponíveis). O padrão é `false`. | `boolean` |
| `renderer` | Força o renderizador para `'canvaskit'` ou `'skwasm'`. | `string` |
| `suppressMultithreadingWarning` | Suprime o aviso do console registrado quando o WebAssembly executa em thread única porque os headers `COOP` e `COEP` estão ausentes. | `boolean` |
| `verboseBuildSelection` | Registra diagnósticos `console.warn` explicando por que cada build candidato em `_flutter.buildConfig.builds` foi selecionado ou ignorado. | `boolean` |
| `wasmAllowList` | Lista de permissão de engines de navegador (`{ blink, webkit, gecko, unknown }`) que controla quais engines executam o build WebAssembly (consulte [Suporte a WebAssembly][wasm-support]). O padrão é `{ blink: true, gecko: false, webkit: false, unknown: false }`. | `Object` |
| `wasmDeferredModulesLoader` | Callback de carregamento personalizado `(moduleNames, onModuleLoaded) => Promise<void>` para buscar módulos `.wasm` adiados quando `--enable-wasm-deferred-loading` está habilitado (consulte [Servir assets de um CDN ou origem própria][multi-asset-cdn]). `onModuleLoaded` aceita `(moduleName: string, module: Response \| Promise<Response> \| ArrayBuffer) => Promise<void>`. | `Function` |

{:.table .table-striped}

### Opções do CanvasKit {:#canvaskit-options}

| Nome | Descrição | Tipo&nbsp;JS |
|---|---|---|
| `canvasKitBaseUrl` | URL base da qual `canvaskit.wasm` e os arquivos do engine de renderização são baixados. | `string` |
| `canvasKitForceCpuOnly` | Força renderização somente por CPU no CanvasKit, sem WebGL. | `boolean` |
| `canvasKitMaximumSurfaces` | Número máximo de superfícies overlay que o renderizador CanvasKit pode alocar. | `number` |
| `canvasKitVariant` | Build do CanvasKit a baixar: `'auto'` (padrão, seleciona o build ideal), `'full'` (todos os navegadores) ou `'chromium'` (build menor, apenas para navegadores baseados em Chromium). | `string` |

{:.table .table-striped}

[embedding-flutter-web]: /platform-integration/web/embedding-flutter-web
[multi-asset-cdn]: /deployment/web#multi-asset-and-cdn-loading
[nonce-mdn]: https://developer.mozilla.org/docs/Web/HTML/Global_attributes/nonce
[wasm-support]: /platform-integration/web/wasm

## Exemplo de uso {:#example-usage}

```js
_flutter.loader.load({
  config: {
    forceSingleThreadedSkwasm: true,
    wasmAllowList: {
      blink: true,
      webkit: true,
      gecko: true,
    },
  },
});
```

## Exemplo: Personalizar a configuração Flutter com base em parâmetros de query da URL {:#example-customizing-flutter-configuration-based-on-url-query-parameters}

O exemplo a seguir mostra um `flutter_bootstrap.js` personalizado que habilita
o modo de thread única quando um parâmetro de query `force_st` está presente na URL:

```js
{% raw %}{{flutter_js}}{% endraw %}
{% raw %}{{flutter_build_config}}{% endraw %}

const searchParams = new URLSearchParams(window.location.search);
const forceSt = searchParams.has('force_st');
const userConfig = forceSt ? { forceSingleThreadedSkwasm: true } : {};
_flutter.loader.load({
  config: userConfig,
});
```

Este script avalia os `URLSearchParams` da página para determinar se
o usuário passou um parâmetro de query `force_st` e configura o app de acordo.

## O callback onEntrypointLoaded {:#the-onentrypointloaded-callback}

Você também pode passar um callback `onEntrypointLoaded` para a API `load` para
executar lógica personalizada em diferentes partes do processo de inicialização.
O processo de inicialização é dividido nos seguintes estágios:

**Carregando o script entrypoint**
: A função `load` chama o callback `onEntrypointLoaded` quando o
  script entrypoint principal foi baixado e executado pelo navegador.
  O Flutter também chama `onEntrypointLoaded` em
  cada hot restart durante o desenvolvimento.

**Inicializando o engine Flutter**
: O callback `onEntrypointLoaded` recebe um
  objeto **engine initializer** como seu único parâmetro.
  Use a função `initializeEngine()` do engine initializer para
  definir a configuração de tempo de execução, como `multiViewEnabled: true`,
  e iniciar o engine web Flutter.

**Executando o app**
: A função `initializeEngine()` retorna uma [`Promise`][js-promise]
  que resolve com um objeto **app runner**. O app runner tem um
  único método, `runApp()`, que executa o app Flutter.

**Adicionando views a (ou removendo views de) um app**
: O método `runApp()` retorna um objeto **flutter app**.
  No modo multi-view, os métodos `addView` e `removeView`
  podem ser usados para gerenciar views do app a partir do app host.
  Para saber mais, confira [Modo embedded][embedded-mode].

[embedded-mode]: /platform-integration/web/embedding-flutter-web#embedded-mode
[js-promise]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise

:::warning
Quando você fornece um callback `onEntrypointLoaded` personalizado,
`_flutter.loader.load()` consome as opções do estágio do loader (como
`wasmAllowList`, `renderer`, `entrypointBaseUrl`,
`forceSingleThreadedSkwasm`, `suppressMultithreadingWarning`,
`verboseBuildSelection` e `wasmDeferredModulesLoader`), mas não encaminha
automaticamente o objeto `config` para `engineInitializer.initializeEngine()`.

Passe o objeto de configuração tanto para `_flutter.loader.load()` quanto para
`initializeEngine()`, para que as configurações dos estágios do loader e do
engine (como `multiViewEnabled`, `hostElement`, `assetBase` e `nonce`) façam
efeito:

```js
const flutterConfig = {
  renderer: 'canvaskit',
  assetBase: 'https://cdn.example.com/app/',
};

_flutter.loader.load({
  config: flutterConfig,
  onEntrypointLoaded: async function(engineInitializer) {
    const appRunner = await engineInitializer.initializeEngine(flutterConfig);
    await appRunner.runApp();
  },
});
```

:::

## Exemplo: Exibir um indicador de progresso {:#example-display-a-progress-indicator}

Para dar feedback ao usuário do seu aplicativo
durante o processo de inicialização,
use os hooks fornecidos para cada estágio para atualizar o DOM:

```js
{% raw %}{{flutter_js}}{% endraw %}
{% raw %}{{flutter_build_config}}{% endraw %}

const loading = document.createElement('div');
document.body.appendChild(loading);
loading.textContent = "Loading Entrypoint...";
_flutter.loader.load({
  onEntrypointLoaded: async function(engineInitializer) {
    loading.textContent = "Initializing engine...";
    // If you have a `config`, pass it here. The config given to `load()`
    // is not forwarded when you supply your own `onEntrypointLoaded`.
    const appRunner = await engineInitializer.initializeEngine();

    loading.textContent = "Running app...";
    await appRunner.runApp();
  }
});
```

## Avisos comuns {:#common-warnings}

Esta seção descreve avisos comuns de build e inicialização web
e como resolvê-los.

### Descontinuação do service worker {:#service-worker-version-deprecation}

Se você encontrar um aviso semelhante ao seguinte durante `flutter build web`:

```text
Warning: In index.html:37: Flutter's service worker is deprecated and will be removed in a future Flutter release.
See https://github.com/flutter/flutter/issues/156910 for more details.
```

Remova as variáveis legadas do service worker, as chamadas de registro e
`serviceWorkerSettings` do seu arquivo `web/index.html` ou do arquivo
`web/flutter_bootstrap.js` personalizado:

```js
// Remove legacy serviceWorkerVersion declarations:
var serviceWorkerVersion = null;
const serviceWorkerVersion = null;

// Remove legacy flutter_service_worker.js registration calls:
navigator.serviceWorker.register('flutter_service_worker.js');

// Remove serviceWorkerSettings from _flutter.loader.load():
_flutter.loader.load({
  serviceWorkerSettings: {
    serviceWorkerVersion: {% raw %}{{flutter_service_worker_version}}{% endraw %},
  },
});
```

Se o seu app requer um service worker personalizado, consulte o
[FAQ da Web](/platform-integration/web/faq#how-do-i-configure-a-service-worker)
e [Hash de conteúdo web (`precache_manifest.json`)][web-content-hashing].

### Descontinuação de FlutterLoader.loadEntrypoint {:#flutterloader-loadentrypoint-deprecation}

Se você encontrar um aviso semelhante ao seguinte:

```text
Warning: In index.html:40: "FlutterLoader.loadEntrypoint" is deprecated.
Use "FlutterLoader.load" instead.
```

Este aviso ocorre quando um arquivo `web/index.html` mais antigo usa
`_flutter.loader.loadEntrypoint()` em vez de `_flutter.loader.load()`.
A mensagem da CLI faz referência à interface `FlutterLoader`,
que o código JavaScript acessa por meio de `_flutter.loader`.

Se o seu app usa a inicialização padrão e não requer
bootstrapping personalizado, substitua o bloco de script legado em `web/index.html`
pela tag de script padrão:

```html title="web/index.html"
<script src="flutter_bootstrap.js" async></script>
```

Se você personalizou sua lógica de inicialização,
atualize seu código de bootstrap em `web/index.html`
(ou em um arquivo `web/flutter_bootstrap.js` personalizado)
para chamar `_flutter.loader.load()` em vez disso:

```js
_flutter.loader.load({
  onEntrypointLoaded: async function(engineInitializer) {
    const appRunner = await engineInitializer.initializeEngine();
    await appRunner.runApp();
  }
});
```

Para ver um exemplo que atualiza elementos do DOM durante a inicialização,
consulte o [exemplo de indicador de progresso][progress indicator example].

[progress indicator example]: #example-display-a-progress-indicator
