---
title: O que há de novo na documentação
description: >-
  Uma lista do que há de novo em docs.flutterbrasil.dev e sites de documentação relacionados.
ia-translate: true
---

Esta página contém anúncios atuais e recentes
do que há de novo no site e blog do Flutter.
Encontre informações antigas sobre novidades na
página [arquivo de novidades][what's new archive].
Você também pode conferir as
[notas de lançamento][release notes] do Flutter SDK.

Para se manter atualizado sobre anúncios do Flutter incluindo mudanças breaking,
junte-se ao grupo Google [flutter-announce][].

Para Dart, você pode se juntar ao grupo Google [Dart Announce][],
e revisar o [changelog do Dart][Dart changelog].

[Dart Announce]: {{site.groups.dart-announce}}
[Dart changelog]: {{site.repo.dart-sdk}}/blob/main/CHANGELOG.md
[flutter-announce]: {{site.groups.flutter-announce}}
[release notes]: /release/release-notes

## 11 de novembro de 2026: Lançamento 3.50 {:#11-november-2026-release-3-50}

Flutter 3.50 está no ar! Para mais detalhes,
confira o [post do blog do Flutter 3.50][3.50-blog-post]
e assista ao vídeo [O que há de novo no Flutter 3.50 e no Dart 3.14][3.50-video].

**Documentação atualizada ou adicionada desde o lançamento 3.47**

* Uma nova seção de [interoperabilidade nativa][native-interop] reúne
  guias para chamar código nativo em Android, iOS, macOS, Windows e Linux.
  Inclui guias atualizados para [Pigeon][pigeon],
  [interop com C e Objective-C][c-interop] com `package:ffigen`,
  [interop com Java e Kotlin][java-interop] com `package:jnigen`,
  e empacotamento de código nativo com [build hooks][bind-native-code].

* Com as bibliotecas Material e Cupertino incluídas no Flutter formalmente
  descontinuadas no Flutter 3.50, um novo [guia de migração][design-pkg-migration]
  orienta você a migrar para os pacotes independentes `material_ui` e
  `cupertino_ui` usando `dart fix` e `MaterialUiCompatibilityBridge`.

* A documentação de [instalação do Flutter][install] foi atualizada com
  fluxos consolidados de instalação rápida e de configuração manual.
  Novas instruções também orientam você a
  [executar apps Flutter em dispositivos iOS][ios-setup],
  contribuídas pelo membro da comunidade [poyito777][].

* A documentação web cobre a nova flag `--web-content-hash` de build para
  [invalidação de cache de assets web com hash][web-deploy],
  orientações atualizadas de compatibilidade de navegadores para
  [WebAssembly][wasm-350], a flag `--no-deprecated-js-interop` de build,
  e orientações revisadas de [acessibilidade web][web-a11y].

* A documentação web e de [WebAssembly (Wasm)][wasm] inclui orientações
  ampliadas para produção: [deploy de builds Wasm][web-deploy-wasm]
  (headers do servidor, pré-compressão, latência de inicialização a frio e
  carregamento de assets via CDN), [otimização de views incorporadas em
  páginas hospedeiras][web-embed-optimize] (lazy-loading, prevenção de
  deslocamentos de layout e convivência com scripts de terceiros),
  comunicação com páginas hospedeiras e medição de marcos de inicialização
  com [package:web][wasm-host-interop], e
  [perfil de Core Web Vitals e memória do Wasm][web-perf] no Chrome DevTools.

* A documentação de AI foi reorganizada em torno de um
  [guia de primeiros passos][ai-get-started] unificado, uma
  [visão geral das ferramentas de AI][ai-tools] que cobre regras de agentes,
  agentes personalizados (incluindo o agente de acessibilidade do Flutter,
  já integrado) e `package:skills_lint`, e um novo guia sobre
  [skills de pacotes][package-skills].

* A seção de dados e backend traz um novo guia para ajudar você a
  [escolher um banco de dados para seu app Flutter][choose-database] e
  documentação ampliada da integração com o [Firebase][firebase-docs].

* Novos guias explicam como configurar flavors de apps desktop para
  [Windows][flavors-windows] e [Linux][flavors-linux].

* A documentação de integração com plataformas inclui atualizações de
  prontidão para [iOS 27, macOS 27 e Xcode 27][supported-platforms], junto
  com orientações ampliadas de migração para o
  [Swift Package Manager][swiftpm-350] em apps e plugins.

* A documentação de contribuição e compatibilidade agora detalha o
  [processo público de RFCs do Flutter][rfc-process] para propor mudanças
  significativas no framework e no ecossistema.

Como sempre, confira a página de [mudanças breaking][bc-3-50] deste lançamento,
que inclui links para guias de migração e outras informações vitais.


[bc-3-50]: /release/breaking-changes#released-in-flutter-3-50
[ai-get-started]: /ai/get-started
[ai-tools]: /ai/tools
[bind-native-code]: /interop/bind-native-code
[3.50-blog-post]: {{site.main-url}}/blog/whats-new-in-flutter-3-50
[3.50-video]: {{site.yt.watch}}?v=K5Kk-C3IL94
[c-interop]: /interop/ffigen
[choose-database]: /data-and-backend/persistence/choose-a-database
[design-pkg-migration]: /release/breaking-changes/material-ui-and-cupertino-ui
[firebase-docs]: /data-and-backend/firebase
[flavors-windows]: /deployment/flavors-windows
[flavors-linux]: /deployment/flavors-linux
[install]: /install
[ios-setup]: /platform-integration/ios/setup
[java-interop]: /interop/jnigen
[native-interop]: /interop
[package-skills]: /ai/package-skills
[pigeon]: /interop/pigeon
[poyito777]: https://github.com/poyito777
[rfc-process]: /release/compatibility-policy
[supported-platforms]: /reference/supported-platforms
[swiftpm-350]: /packages-and-plugins/swift-package-manager/for-app-developers
[wasm]: /platform-integration/web/wasm
[wasm-350]: /platform-integration/web/wasm
[wasm-host-interop]: /platform-integration/web/wasm#host-scripts-and-measurement-apis
[web-a11y]: /ui/accessibility/web-accessibility
[web-deploy]: /deployment/web
[web-deploy-wasm]: /deployment/web#deploy-webassembly-in-production
[web-embed-optimize]: /platform-integration/web/embedding-flutter-web#landing-pages-and-third-party-scripts
[web-perf]: /perf/web-performance

---

## 12 de agosto de 2026: Lançamento 3.47 {:#12-august-2026-release-3-47}

Flutter 3.47 está no ar! Para mais detalhes,
confira o [post do blog do Flutter 3.47][3.47-blog-post].

**Documentação atualizada ou adicionada desde o lançamento 3.44**

* A ferramenta [Widget Previews][widget-previewer] do Flutter se tornou estável,
  introduzindo cache de build local para tempos de inicialização mais rápidos
  e busca e filtragem de prévias em tempo real.

* A documentação de AI foi reorganizada para melhorar a navegação
  e ajudar você a começar rapidamente com agent-plugins
  (anteriormente conhecidos como skills) usando a [Antigravity CLI][antigravity-cli],
  integrações com IDEs e outros assistentes de programação.
  Uma página experimental sobre [AI Evaluations][ai-evals] também foi adicionada.

* Um novo guia sobre [gerenciamento de dependências][dependency-mgmt] explica
  como configurar, gerenciar e atualizar dependências de pacotes.

* Os guias do [Swift Package Manager][swiftpm] para iOS e macOS foram expandidos,
  e a documentação de [Add-to-app][add-to-app] agora inclui
  instruções de integração com SwiftPM.

* A documentação web agora inclui orientações de solução de problemas para
  [erros de compilação Wasm][wasm] e instruções para usar a
  flag `--source-maps` durante builds web.

* O guia de assets agora explica como empacotar
  [assets específicos de plataforma][platform-assets] no `pubspec.yaml`.

* Um novo guia de migração ajuda você a migrar plugins e apps para
  os plugins Gradle de [Kotlin integrado][kotlin-migration].

Como sempre, confira a página de [mudanças breaking][bc-3-47] deste lançamento,
que inclui links para guias de migração e outras informações vitais.

[3.47-blog-post]: /blog/whats-new-in-flutter-3-47
[widget-previewer]: /tools/widget-previewer
[antigravity-cli]: /ai/antigravity-cli
[ai-evals]: /ai/evals
[dependency-mgmt]: /packages-and-plugins/dependency-management
[swiftpm]: /packages-and-plugins/swift-package-manager
[add-to-app]: /add-to-app
[wasm]: /platform-integration/web/wasm
[platform-assets]: /ui/assets/assets-and-images
[kotlin-migration]: /release/breaking-changes/migrate-to-built-in-kotlin
[bc-3-47]: /release/breaking-changes#released-in-flutter-3-47

---

## 18 de maio de 2026: Lançamento Google I/O 3.44 {:#18-may-2026-google-io-release-3-44}

Flutter 3.44 está no ar! Este lançamento está repleto de novidades.
Para mais detalhes, confira o [post do blog do Flutter 3.44][3.44-blog-post]
e [assista ao vídeo][3.44-whats-new]:

<div class="video-wrapper">
<span class="video-intro">Confira as novidades do Dart e do Flutter</span>
<YouTubeEmbed id="I1uIbGh1dGE" title="What's New in Dart and Flutter 3.44" fullWidth></YouTubeEmbed>
</div>

[3.44-blog-post]: {{site.flutter-blog}}/whats-new-in-flutter-3-44-b0cc1ad3c527
[3.44-whats-new]: https://www.youtube.com/watch?v=I1uIbGh1dGE&t=451s

Mais atualizações do site em breve.

## 11 de fevereiro de 2026: Lançamento "Year of the Fire Horse" 3.41 {:#11-february-2026-year-of-the-fire-horse-release-3-41}

Flutter 3.41 está no ar! Para mais detalhes,
confira o [post do blog do Flutter 3.41][3.41-blog-post].

[3.41-blog-post]: {{site.flutter-blog}}/whats-new-in-flutter-3-41-302ec140e632


**Documentação atualizada ou adicionada desde o lançamento 3.38**

Além de muitas atualizações menores, aqui estão algumas das mudanças
mais significativas:

* Introduzimos uma nova experiência de introdução para
  Flutter e Dart, conforme anunciado em um [post do blog][few] recente.
  Você pode encontrar o [caminho de aprendizado][learning pathway] do Flutter na aba **Learn**
  no topo de [docs.flutter.dev][]. Para voltar ao restante do site,
  selecione a aba **Guides**. Como sempre, a aba **Reference**
  leva você à [documentação da API do Flutter][Flutter API docs].

* Flutter e Dart agora têm um [glossário oficial][official glossary].
  Como parte da recente migração dos sites para o Jaspr,
  um glossário foi adicionado ao site do Flutter.
  Espere vê-lo crescer com o tempo.

* Como sempre, confira a página de [mudanças breaking][bc-3-41] deste lançamento,
  que inclui links para guias de migração e outras informações vitais.
  Um agradecimento especial a [navaronbracke][], que escreveu um guia
  de migração para a [descontinuação do callback `onReorder`][onReorder].

[bc-3-41]: /release/breaking-changes#released-in-flutter-3-41
[docs.flutter.dev]: /
[Flutter API docs]: {{site.api}}
[few]: {{site.flutter-blog}}/announcing-our-new-dart-and-flutter-getting-started-experience-b8c4b2be0984
[learning pathway]: /learn
[navaronbracke]: https://github.com/navaronbracke
[official glossary]: /resources/glossary
[onReorder]: /release/breaking-changes/separated-builder-find-child-index-callback

---

Para lançamentos passados, confira a
página [arquivo de novidades][What's new archive].

[What's new archive]: /release/archive-whats-new
