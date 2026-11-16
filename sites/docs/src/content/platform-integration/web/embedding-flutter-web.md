---
ia-translate: true
title: Adicionando Flutter a qualquer aplicação web
shortTitle: Adicionar Flutter a qualquer app web
description: Aprenda as diferentes formas de incorporar views Flutter em conteúdo web.
---

Views Flutter e conteúdo web podem ser compostos para produzir uma aplicação web
de diferentes formas. Escolha uma das seguintes dependendo do seu caso de uso:

* Uma view Flutter controla a página inteira ([modo de página inteira][full page mode], incluindo
  [incorporações `iframe`](#iframe-embedding))
* Múltiplas views Flutter adicionadas dinamicamente a uma aplicação web existente
  ([modo incorporado][embedded mode])
* Uma única view Flutter renderizada dentro de um elemento HTML específico
  ([`hostElement` customizado][custom `hostElement`])

[full page mode]: #full-page-mode
[embedded mode]: #embedded-mode
[custom `hostElement`]: #custom-element-hostelement

## Modo de página inteira {:#full-page-mode}

No modo de página inteira, a aplicação web Flutter assume o controle de toda a
janela do navegador e cobre seu viewport completamente ao renderizar.

Este é o modo de incorporação padrão para novos projetos Flutter web, e nenhuma
configuração adicional é necessária.

```html highlightLines=6
<!DOCTYPE html>
<html>
  <head>
  </head>
  <body>
    <script src="flutter_bootstrap.js" defer></script>
  </body>
</html>
```

Quando Flutter web é lançado sem referenciar `multiViewEnabled` ou um
`hostElement`, ele usa o modo de página inteira.

Para aprender mais sobre o arquivo `flutter_bootstrap.js`,
consulte [Personalize a inicialização do app][Customize app initialization].

[Customize app initialization]: /platform-integration/web/initialization

### Incorporação com `iframe` {:#iframe-embedding}

O modo de página inteira é recomendado ao incorporar uma aplicação web Flutter através de um
`iframe`. A página que incorpora o `iframe` pode dimensioná-lo e posicioná-lo conforme necessário,
e Flutter irá preenchê-lo completamente.

```html
<iframe src="https://url-to-your-flutter/index.html"></iframe>
```

Para aprender mais sobre os prós e contras de um `iframe`,
consulte a documentação do [elemento `<iframe>`][Inline Frame element] no MDN.

[Inline Frame element]: https://developer.mozilla.org/en-US/docs/Web/HTML/Element/iframe

### Compare os modos `iframe` e DOM direto {: #compare-iframe-and-dom-embedding }

Ao integrar o Flutter em uma aplicação web existente,
escolha entre a incorporação via `iframe` e a incorporação direta no DOM
(`hostElement` ou multi-view)
com base nos seus requisitos de isolamento, desempenho e comunicação:

| Recurso | Incorporação via `iframe` | Incorporação direta no DOM (`hostElement` / multi-view) |
| --- | --- | --- |
| **Isolamento e sandboxing** | Sandboxing completo do navegador com escopo JS global, árvore DOM e estilos CSS separados | Compartilha a mesma árvore DOM e o mesmo contexto JavaScript |
| **Memória e instâncias da engine** | Cada `iframe` inicializa sua própria engine Flutter, runtime WebAssembly/JS e heap de memória | Uma única instância da engine Flutter e um único heap de memória podem gerenciar uma ou mais views (ao usar multi-view) |
| **Interoperabilidade com JavaScript** | A comunicação requer mensagens assíncronas (como `postMessage`) | Comunicação síncrona direta usando `package:web` e `dart:js_interop` |
| **Estilo e layout** | Frame independente, isolado do CSS da página host | Integração direta ao layout CSS da página host (como flexbox e grid) |
| **Compartilhamento de estado** | O estado precisa ser sincronizado entre os limites das janelas | Compartilhamento direto de estado em Dart entre todas as views anexadas |
| **Ideal para** | Widgets de terceiros, conteúdo não confiável, micro-frontends isolados ou incorporações simples prontas para uso | Componentes de UI incorporados em apps web existentes, dashboards multi-view e integração estreita com o app host |

{:.table .table-striped}

## Modo incorporado {:#embedded-mode}

Aplicações web Flutter também podem renderizar conteúdo em um número arbitrário de
elementos (comumente `div`s) de outra aplicação web; isso é chamado de "embedded
mode" (ou "multi-view").

Neste modo:

* Uma aplicação web Flutter pode ser lançada, mas não renderiza até que a primeira
  "view" seja adicionada, com `addView`.
* A aplicação host pode adicionar ou remover views da aplicação web Flutter
  incorporada.
* A aplicação Flutter é notificada quando views são adicionadas ou removidas,
  para que possa ajustar seus widgets adequadamente.

### Habilitar modo multi-view {:#enable-multi-view-mode}

Habilite o modo multi-view configurando `multiViewEnabled: true` no
método `initializeEngine` como mostrado:

```js highlightLines=8 title="flutter_bootstrap.js"
{% raw %}{{flutter_js}}{% endraw %}
{% raw %}{{flutter_build_config}}{% endraw %}

_flutter.loader.load({
  onEntrypointLoaded: async function onEntrypointLoaded(engineInitializer) {
    let engine = await engineInitializer.initializeEngine({
      multiViewEnabled: true, // Enables embedded mode.
    });
    let app = await engine.runApp();
    // Make this `app` object available to your JS app.
  }
});
```

### Gerenciar views Flutter a partir do JS {:#manage-flutter-views-from-js}

Para adicionar ou remover views, use o objeto `app` retornado pelo método `runApp`:

```js highlightLines=2-4,7
// Adding a view...
let viewId = app.addView({
  hostElement: document.querySelector('#some-element'),
});

// Removing viewId...
let viewConfig = app.removeView(viewId);
```

### Lidar com mudanças de view a partir do Dart {:#handling-view-changes-from-dart}

Adições e remoções de views são expostas ao Flutter através do
[método `didChangeMetrics`][`didChangeMetrics` method] da classe `WidgetsBinding`.

A lista completa de views anexadas ao seu app Flutter está disponível
através do iterável `WidgetsBinding.instance.platformDispatcher.views`.
Essas views são do [tipo `FlutterView`][type `FlutterView`].

Para renderizar conteúdo em cada `FlutterView`, seu app Flutter precisa criar um
[widget `View`][`View` widget]. Widgets `View` podem ser agrupados juntos sob um
[widget `ViewCollection`][`ViewCollection` widget].

O exemplo a seguir, do _Multi View Playground_, encapsula
o acima em um widget `MultiViewApp` que pode ser usado como o widget raiz para
seu app. Uma [função `WidgetBuilder`][`WidgetBuilder` function] executa para cada `FlutterView`:

```dart highlightLines=25,39,46-49,56-61,72 title="multi_view_app.dart"
import 'dart:ui' show FlutterView;
import 'package:flutter/widgets.dart';

/// Calls [viewBuilder] for every view added to the app to obtain the widget to
/// render into that view. The current view can be looked up with [View.of].
class MultiViewApp extends StatefulWidget {
  const MultiViewApp({super.key, required this.viewBuilder});

  final WidgetBuilder viewBuilder;

  @override
  State<MultiViewApp> createState() => _MultiViewAppState();
}

class _MultiViewAppState extends State<MultiViewApp> with WidgetsBindingObserver {
  @override
  void initState() {
    super.initState();
    WidgetsBinding.instance.addObserver(this);
    _updateViews();
  }

  @override
  void didUpdateWidget(MultiViewApp oldWidget) {
    super.didUpdateWidget(oldWidget);
    // Need to re-evaluate the viewBuilder callback for all views.
    _views.clear();
    _updateViews();
  }

  @override
  void didChangeMetrics() {
    _updateViews();
  }

  Map<Object, Widget> _views = <Object, Widget>{};

  void _updateViews() {
    final Map<Object, Widget> newViews = <Object, Widget>{};
    for (final FlutterView view in WidgetsBinding.instance.platformDispatcher.views) {
      final Widget viewWidget = _views[view.viewId] ?? _createViewWidget(view);
      newViews[view.viewId] = viewWidget;
    }
    setState(() {
      _views = newViews;
    });
  }

  Widget _createViewWidget(FlutterView view) {
    return View(
      view: view,
      child: Builder(
        builder: widget.viewBuilder,
      ),
    );
  }

  @override
  void dispose() {
    WidgetsBinding.instance.removeObserver(this);
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    return ViewCollection(views: _views.values.toList(growable: false));
  }
}
```

Para mais informações, consulte o
[mixin `WidgetsBinding`]({{site.api}}/flutter/widgets/WidgetsBinding-mixin.html)
na documentação da API.

[`didChangeMetrics` method]: {{site.api}}/flutter/widgets/WidgetsBindingObserver/didChangeMetrics.html
[type `FlutterView`]: {{site.api}}/flutter/dart-ui/FlutterView-class.html
[`View` widget]: {{site.api}}/flutter/widgets/View-class.html
[`ViewCollection` widget]: {{site.api}}/flutter/widgets/ViewCollection-class.html
[`WidgetsBinding` mixin]: {{site.api}}/flutter/widgets/WidgetsBinding-mixin.html
[`WidgetBuilder` function]: {{site.api}}/flutter/widgets/WidgetBuilder.html

### Substituir `runApp` por `runWidget` no Dart {:#replace-runapp-by-runwidget-in-dart}

A [função `runApp`][`runApp` function] do Flutter assume que há pelo menos uma view disponível
para renderizar (a `implicitView`), no entanto no modo multi-view do Flutter web,
a `implicitView` não existe mais, então `runApp` começará a falhar com
erros `Unexpected null value`.

No modo multi-view, seu `main.dart` deve chamar a [função `runWidget`][`runWidget` function]
em vez disso. Ela não requer uma `implicitView`, e só renderizará nas
views que foram explicitamente adicionadas ao seu app.

O exemplo a seguir usa o `MultiViewApp` descrito acima para renderizar
cópias do widget `MyApp()` em cada `FlutterView` disponível:

```dart highlightLines=3 title="main.dart"
void main() {
  runWidget(
    MultiViewApp(
      viewBuilder: (BuildContext context) => const MyApp(),
    ),
  );
}
```

[`runApp` function]: {{site.api}}/flutter/widgets/runApp.html
[`runWidget` function]: {{site.api}}/flutter/widgets/runWidget.html

### Identificar views {: #identifying-views }

Cada `FlutterView` tem um identificador atribuído pelo Flutter quando
anexada. Este `viewId` pode ser usado para identificar exclusivamente cada view,
recuperar sua configuração inicial, ou decidir o que renderizar nela.

O `viewId` da `FlutterView` renderizada pode ser recuperado de
seu `BuildContext` assim:

```dart highlightLines=4-5
class SomeWidget extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    // Retrieve the `viewId` where this Widget is being built:
    final int viewId = View.of(context).viewId;
    // ...
```

Similarmente, a partir do método `viewBuilder` do `MultiViewApp`,
o `viewId` pode ser recuperado assim:

```dart highlightLines=4
MultiViewApp(
  viewBuilder: (BuildContext context) {
    // Retrieve the `viewId` where this Widget is being built:
    final int viewId = View.of(context).viewId;
    // Decide what to render based on `viewId`...
  },
)
```

Leia mais sobre o [construtor `View.of`][`View.of` constructor].

[`View.of` constructor]: {{site.api}}/flutter/widgets/View/of.html

### Configuração inicial da view {:#initial-view-configuration}

Views Flutter podem receber qualquer dado de inicialização do JS ao iniciar.
Os valores são passados através da propriedade `initialData` do método `addView`,
como mostrado:

```js highlightLines=4-7
// Adding a view with initial data...
let viewId = app.addView({
  hostElement: someElement,
  initialData: {
    greeting: 'Hello, world!',
    randomValue: Math.floor(Math.random() * 100),
  }
});
```

No Dart, o `initialData` está disponível como um objeto `JSAny`, acessível através da
propriedade `views` de nível superior na biblioteca `dart:ui_web`. Os dados são
acessados através do `viewId` da view atual, como mostrado:

```dart
final initialData = ui_web.views.getInitialData(viewId) as YourJsInteropType;
```

Para aprender como definir a classe `YourJsInteropType` para mapear o objeto `initialData`
passado do JS para que seja type-safe no seu programa Dart, consulte:
[Interoperabilidade com JavaScript][JS Interoperability] no dartbrasil.dev.

[JS Interoperability]: {{site.dart-site}}/interop/js-interop

### Restrições de view {:#view-constraints}

Por padrão, uma view Flutter web incorporada considera o tamanho de seu `hostElement`
como uma propriedade imutável, e restringe fortemente seu layout ao espaço
disponível.

Na web, é comum que o tamanho intrínseco de um elemento afete o
layout da página (como tags `img` ou `p` que podem refluir conteúdo ao
redor delas).

Ao adicionar uma view ao Flutter web, você pode configurá-la com restrições que
informam ao Flutter como a view precisa ser disposta:

```js highlightLines=4-8
// Adding a view with initial data...
let viewId = app.addView({
  hostElement: someElement,
  viewConstraints: {
    maxWidth: 320,
    minHeight: 0,
    maxHeight: Infinity,
  }
});
```

As restrições de view passadas do JS precisam ser compatíveis com o estilo CSS
do `hostElement` onde Flutter está sendo incorporado. Por exemplo, Flutter
não tentará "corrigir" constantes contraditórias como passar `max-height: 100px`
no CSS, mas `maxHeight: Infinity` para o Flutter.

Para aprender mais, consulte a [classe `ViewConstraints`][`ViewConstraints` class],
e [Entendendo constraints][Understanding constraints].

[`ViewConstraints` class]: {{site.api}}/flutter/dart-ui/ViewConstraints-class.html
[Understanding constraints]: /ui/layout/constraints

## Elemento customizado (`hostElement`) {:#custom-element-hostelement}

Você pode incorporar um app Flutter web de visualização única
em qualquer elemento HTML da sua página web.

Para dizer ao Flutter web em qual elemento renderizar,
passe um objeto com um campo `config` para a função `_flutter.loader.load`
que especifica um `HTMLElement` como o `hostElement`.

```js highlightLines=3
_flutter.loader.load({
  config: {
    hostElement: document.getElementById('flutter_host'),
  }
});
```

:::note
Incorporação multi-view também funciona com uma única view.
Uma vantagem de incorporar uma visualização única usando suporte multi-view,
é que você pode criar e remover views dinamicamente. Além disso,
se o suporte de visualização única for descontinuado, não afetará seu app.
:::

Para aprender mais sobre outras opções de configuração,
consulte [Personalizando a inicialização do app web][Customizing web app initialization].

[Customizing web app initialization]: /platform-integration/web/initialization

## Requisitos de CSS do elemento host {: #host-element-css-requirements }

Ao incorporar o Flutter diretamente em um elemento DOM (usando `hostElement`
com visualização única ou `addView` no modo multi-view), o Flutter dimensiona seu
canvas de renderização com base nas dimensões calculadas do contêiner host.

Elementos contêiner HTML como `<div>` têm por padrão `height: auto` e
`position: static`. Sem estilização CSS explícita, um elemento host vazio
tem uma altura calculada de `0px`.
Isso faz com que o Flutter calcule a altura da view como zero,
o que colapsa a view Flutter e impede que o conteúdo seja renderizado.

### Propriedades CSS obrigatórias {: #mandatory-css-properties }

Para garantir que o Flutter renderize corretamente, aplique as seguintes propriedades CSS
ao seu elemento host:

* **`width` e `height`**:
  Defina dimensões explícitas ou relativas (por exemplo,
  `width: 100%; height: 500px;` ou `height: 100%` dentro de um pai dimensionado).
  O elemento deve resultar em largura e altura diferentes de zero.
* **`position: relative`** (ou `position: absolute` / `position: fixed`):
  Estabelece um contexto de posicionamento para o canvas e as camadas de sobreposição do DOM
  que o Flutter cria dentro do contêiner.
* **`overflow: hidden`**:
  Impede que o canvas e os elementos de interação do Flutter ultrapassem
  os limites do contêiner ou causem barras de rolagem inesperadas na página host.

### Exemplo de CSS {: #example-css }

```css title="styles.css"
#flutter_host {
  width: 100%;
  height: 600px;
  position: relative;
  overflow: hidden;
}
```

```html title="index.html"
<div id="flutter_host"></div>
```

## Otimize as páginas host e os scripts de terceiros {: #landing-pages-and-third-party-scripts }

Ao incorporar uma view Flutter em uma página web existente (como um dashboard,
uma página com muito conteúdo ou um app que carrega scripts de terceiros),
aplique os padrões a seguir para proteger o desempenho do carregamento inicial
da página e evitar conflitos de cabeçalhos cross-origin.

### Carregue sob demanda (lazy load) as views incorporadas abaixo da dobra {: #lazy-load-embedded-views }

Por padrão, o `flutter_bootstrap.js` chama `_flutter.loader.load()` imediatamente,
o que pode competir com o HTML, o CSS e as imagens críticas da página host
durante o carregamento inicial.

Quando uma view Flutter incorporada fica abaixo da dobra ou abre por interação,
adie o `_flutter.loader.load()` em um arquivo `web/flutter_bootstrap.js`
personalizado, usando um `IntersectionObserver` (ou um listener de `click` ou
`pointerenter`) para proteger o Largest Contentful Paint (LCP) inicial:

```js title="web/flutter_bootstrap.js"
{% raw %}{{flutter_js}}{% endraw %}
{% raw %}{{flutter_build_config}}{% endraw %}

function initLazyFlutter() {
  const hostElement = document.getElementById('flutter_host');
  if (!hostElement) return;

  const observer = new IntersectionObserver(
    (entries) => {
      if (!entries.some((entry) => entry.isIntersecting)) return;
      observer.disconnect();

      _flutter.loader.load({
        config: { hostElement },
      });
    },
    // Start loading before the container scrolls into view.
    { rootMargin: '200px' },
  );

  observer.observe(hostElement);
}

if (document.readyState === 'loading') {
  document.addEventListener('DOMContentLoaded', initLazyFlutter, {
    once: true,
  });
} else {
  initLazyFlutter();
}
```

### Evite o Cumulative Layout Shift (CLS) {: #prevent-cls }

Além das [propriedades CSS obrigatórias](#mandatory-css-properties)
necessárias para o Flutter calcular limites de view diferentes de zero,
montar uma view Flutter em um contêiner cuja altura muda depois do layout
dispara um Cumulative Layout Shift (CLS) na página host.

Para manter o conteúdo ao redor estável enquanto o Flutter inicializa:

* Defina o tamanho de `#flutter_host` com as dimensões finais do layout desde
  o início, usando valores explícitos de `width` e `height` ou a propriedade CSS
  `aspect-ratio` (por exemplo, `width: 100%; aspect-ratio: 16 / 9;`).
* Renderize um placeholder leve de esqueleto em HTML ou CSS dentro de
  `#flutter_host`, com as mesmas dimensões reservadas, até o Flutter pintar o
  primeiro frame.

### Conviva com scripts e iframes cross-origin de terceiros {: #coexist-with-third-party-scripts }

Ao compilar com a flag `--wasm`, a renderização WebAssembly multithread exige
isolamento cross-origin (os cabeçalhos de resposta HTTP `COOP` e `COEP`).
Em páginas host que carregam scripts de terceiros, analytics, imagens externas
ou embeds cross-origin, escolha a estratégia de cabeçalhos que corresponda à página:

* **Scripts, analytics e imagens de terceiros**:
  configure o servidor para enviar `Cross-Origin-Opener-Policy: same-origin` e
  `Cross-Origin-Embedder-Policy: credentialless`, em vez de `require-corp`.
  Com `credentialless`, o navegador carrega subrecursos cross-origin sem exigir
  que os servidores de terceiros retornem cabeçalhos
  `Cross-Origin-Resource-Policy`.
* **Embeds de `<iframe>` cross-origin de terceiros**:
  quando o documento host define o cabeçalho `Cross-Origin-Embedder-Policy`, o
  navegador bloqueia os elementos `<iframe>` cross-origin que não aderem ao COEP.
  Quando a página host incorpora iframes de terceiros incompatíveis, omita os
  cabeçalhos `COEP` e `COOP`.
  Sem isolamento cross-origin, o build WebAssembly do Flutter roda
  automaticamente em modo single-thread; defina `suppressMultithreadingWarning: true`
  para silenciar o aviso no console:

  ```js title="web/flutter_bootstrap.js"
  {% raw %}{{flutter_js}}{% endraw %}
  {% raw %}{{flutter_build_config}}{% endraw %}

  _flutter.loader.load({
    config: {
      hostElement: document.getElementById('flutter_host'),
      suppressMultithreadingWarning: true,
    },
  });
  ```

  Se você fornecer um callback `onEntrypointLoaded` personalizado (por exemplo,
  no [modo multi-view](#enable-multi-view-mode)), passe o objeto de configuração
  tanto para `load()` quanto para `initializeEngine()`.
  Essa configuração preserva a execução WebAssembly (`dart2wasm`) single-thread
  sem quebrar os iframes cross-origin de terceiros.
  Para mais detalhes sobre as flags do loader,
  consulte [Personalize o loader do Flutter][customize-flutter-loader].

[customize-flutter-loader]: /platform-integration/web/initialization#customize-the-flutter-loader
