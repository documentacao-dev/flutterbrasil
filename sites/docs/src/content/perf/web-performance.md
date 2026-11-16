---
ia-translate: true
title: Depurar performance para apps web
description: >-
  Aprenda a fazer profiling de apps Flutter web e WebAssembly, medir Core Web
  Vitals e inspecionar o footprint do Wasm no Chrome DevTools.
---

O framework Flutter emite eventos de timeline conforme constrói frames,
desenha cenas e rastreia atividades de runtime, como coletas de lixo.
Esses eventos aparecem diretamente no
[painel de Performance do Chrome DevTools][Chrome DevTools Performance panel]
para profiling e depuração.

:::note
Para saber mais sobre como otimizar os tempos de carregamento inicial,
leia o post do blog do Flutter
[Melhores práticas para otimizar a velocidade de carregamento do Flutter web][article].

[article]: {{site.flutter-blog}}/best-practices-for-optimizing-flutter-web-loading-speed-7cc0df14ce5c
:::

Para emitir eventos de timeline personalizados no código Dart, use as APIs
[`Timeline`][] e [`TimelineTask`][] do `dart:developer`, ou registre marcações
no nível do navegador com a propriedade `window.performance` (consulte
[Medir marcos de inicialização e de frame][wasm-user-timing]).

![Screenshot of the Chrome DevTools performance panel](/assets/images/docs/tools/devtools/chrome-devtools-performance-panel.png)

## Compilar e executar em modo profile {:#build-and-run-for-profiling}

Sempre avalie a performance em runtime no [modo profile][profile mode] ou no
modo release, e não no modo debug.
Escolha o comando que corresponde à métrica que você está testando:

| O que medir | Comando | Observações |
| :--- | :--- | :--- |
| **CPU, renderização de frames e memória** | `flutter run -d chrome --profile` (adicione `--wasm` para WebAssembly) | Compila com otimizações de produção (`-O2` para Wasm), preservando símbolos e nomes de tipos WasmGC nos traces e nos heap snapshots. |
| **Bundle de profile implantável** | `flutter build web --profile` (adicione `--wasm` para WebAssembly) | Gera um bundle de profile estático em `build/web` para testar em hosts de staging ou em dispositivos-alvo. |
| **Tamanho do download e carregamento em cold start** | `flutter build web` ou `flutter build web --wasm` | Compila um bundle de release minificado e sem símbolos, com tamanhos de arquivo de produção. |

{:.table .table-striped}

## Configurar flags opcionais de timeline {:#optional-flags-to-enhance-tracing}

Para configurar quais eventos de timeline de widgets e de renderização são
rastreados, defina qualquer uma das seguintes propriedades de nível superior
como `true` em `main()` antes de fazer o profiling:

- [`debugProfileBuildsEnabled`][]:
  Adiciona eventos `Timeline` para cada `Widget` construído.
- [`debugProfileBuildsEnabledUserWidgets`][]:
  Adiciona eventos `Timeline` para cada `Widget` criado pelo usuário e construído.
- [`debugProfileLayoutsEnabled`][]:
  Adiciona eventos `Timeline` para cada layout de `RenderObject`.
- [`debugProfilePaintsEnabled`][]:
  Adiciona eventos `Timeline` para cada `RenderObject` pintado.

Para capturar um trace de frames:

1.  Opcional: defina como `true` as flags de rastreamento que você quer
    acompanhar em `main()`.
1.  Execute o app no modo profile (`flutter run -d chrome --profile`, adicionando
    `--wasm` para builds WebAssembly).
1.  Abra o [painel de Performance do Chrome DevTools][Chrome DevTools Performance panel]
    e [grave um trace de runtime][start recording] enquanto reproduz a interação.

## Fazer profiling dos Core Web Vitals (LCP, INP e CLS) {:#profile-core-web-vitals}

Grave um trace no [painel de Performance do Chrome DevTools][Chrome DevTools Performance panel]
para avaliar os [Core Web Vitals][] e inspecionar o comportamento de renderização
específico do Flutter:

- **Largest Contentful Paint (LCP)**:
  Em apps Flutter web de página inteira, o Chrome muitas vezes atribui o LCP
  ao elemento de carregamento do HTML hospedeiro (ou à superfície inicial do
  canvas), e não aos widgets internos do Flutter.
  - Registre uma marcação de User Timing no primeiro frame do Flutter para medir
    a prontidão real do app (consulte
    [Medir marcos de inicialização e de frame][wasm-user-timing]).
  - Verifique se o `flutter_bootstrap.js`, o motor de renderização do Flutter e
    o `main.dart.wasm` (ou `main.dart.js`) são baixados em paralelo, sem cascatas
    (consulte [Reduzir a latência de cold start][cold-start]).
- **Interaction to Next Paint (INP)**:
  Inspecione a thread **Main** (além das threads worker do WebAssembly, quando o
  cross-origin isolation está ativo) em torno de interações lentas, procurando
  tarefas longas, picos de layout síncrono ou de compilação de shaders, ou
  pausas de coleta de lixo do V8.
- **Cumulative Layout Shift (CLS)**:
  Verifique se remover o indicador de carregamento do HTML ou montar views
  Flutter embutidas com `addView()` não desloca o conteúdo DOM ao redor (consulte
  [Evitar o Cumulative Layout Shift (CLS)][prevent-cls]).

## Inspecionar o tamanho do módulo Wasm e o footprint de memória {:#inspect-wasm-footprint}

Para os passos gerais de profiling de tamanho do `dart2wasm` e de heap WasmGC no
Chrome DevTools, consulte [Fazer profiling de performance e footprint do Wasm][dart-wasm-profile].
Em um app Flutter Wasm, também confira os seguintes artefatos específicos do Flutter:

- **Payloads de bundle e de assets**:
  Gere um bundle de release sem símbolos (`flutter build web --wasm`) e inspecione
  `main.dart.wasm`, o binário `.wasm` do motor de renderização do Flutter e
  `assets/**` no [painel Network do Chrome DevTools][Chrome DevTools Network panel].
  Para reduzir o download inicial, consulte [Medindo o tamanho do app][app-size] e
  [Reduzir a latência de cold start][cold-start] (`--enable-wasm-deferred-loading`).
- **Memória do engine e do WasmGC**:
  Execute com `--profile` (ou `flutter build web --wasm --no-strip-wasm`) ao
  [gravar heap snapshots][recording heap snapshots] para inspecionar tanto os
  objetos Dart WasmGC no heap do V8 quanto a `WebAssembly.Memory` linear alocada
  pelo motor de renderização do Flutter.

[app-size]: /perf/app-size
[Chrome DevTools Network panel]: https://developer.chrome.com/docs/devtools/network
[Chrome DevTools Performance panel]: https://developer.chrome.com/docs/devtools/performance
[cold-start]: /deployment/web#reduce-cold-start-latency
[Core Web Vitals]: https://web.dev/articles/vitals
[dart-wasm-profile]: {{site.dart-site}}/web/wasm#profile-wasm
[`debugProfileBuildsEnabled`]: {{site.api}}/flutter/widgets/debugProfileBuildsEnabled.html
[`debugProfileBuildsEnabledUserWidgets`]: {{site.api}}/flutter/widgets/debugProfileBuildsEnabledUserWidgets.html
[`debugProfileLayoutsEnabled`]: {{site.api}}/flutter/rendering/debugProfileLayoutsEnabled.html
[`debugProfilePaintsEnabled`]: {{site.api}}/flutter/rendering/debugProfilePaintsEnabled.html
[prevent-cls]: /platform-integration/web/embedding-flutter-web#prevent-cls
[profile mode]: /testing/build-modes#profile
[recording heap snapshots]: https://developer.chrome.com/docs/devtools/memory-problems/heap-snapshots
[start recording]: https://developer.chrome.com/docs/devtools/performance/#record
[`Timeline`]: {{site.api}}/flutter/dart-developer/Timeline-class.html
[`TimelineTask`]: {{site.api}}/flutter/dart-developer/TimelineTask-class.html
[wasm-user-timing]: /platform-integration/web/wasm#record-startup-and-frame-milestones
