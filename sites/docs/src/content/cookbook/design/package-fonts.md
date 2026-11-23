---
ia-translate: true
title: Exportar fonts de um package
description: Como exportar fonts de um package.
---

<?code-excerpt path-base="cookbook/design/package_fonts"?>

Em vez de declarar uma font como parte de um app,
você pode declarar uma font como parte de um package separado.
Esta é uma maneira conveniente de compartilhar a mesma font entre
vários projetos diferentes,
ou para programadores que publicam seus packages no [pub.dev][].
Esta receita usa os seguintes passos:

  1. Adicionar uma font a um package.
  2. Adicionar o package e a font ao app.
  3. Usar a font.

:::note
Confira o package [google_fonts][] para acesso direto
a quase 1000 famílias de fonts open-source.
:::

## 1. Adicionar uma font a um package {:#1-add-a-font-to-a-package}

Para exportar fonts, coloque os arquivos no diretório `lib` do seu package ou em um de seus subdiretórios.
Este exemplo usa `lib/fonts` em um package Flutter chamado `awesome_package`.
Coloque o diretório do package ao lado do diretório do seu app.

<FileTree>

- awesome_package/
  - lib/
    - awesome_package.dart
    - fonts/
      - Raleway-Regular.ttf
      - Raleway-Italic.ttf
  - pubspec.yaml

</FileTree>

## 2. Adicionar o package e fonts ao app {:#2-add-the-package-and-fonts-to-the-app}

Agora você pode usar as fonts no package
atualizando o `pubspec.yaml` no diretório raiz do *app*.

{% render "docs/cookbook/material-ui-dependency.md", site: site %}

### Adicionar o package ao app {:#add-the-package-to-the-app}

No diretório do seu app, adicione `awesome_package`
usando um [descritor de package de caminho][path package descriptor].

```console
$ flutter pub add "awesome_package:{path: ../awesome_package}"
```

[path package descriptor]: {{site.dart-site}}/tools/pub/dependencies#path-package-descriptor

### Declarar os assets de font {:#declare-the-font-assets}

No `pubspec.yaml` do seu app, declare os arquivos de font do `awesome_package`.

Para declarar fonts de package, prefixe o caminho para a font com
`packages/awesome_package`.
Isso diz ao Flutter para procurar na pasta `lib`
do package pela font.

```yaml
flutter:
  fonts:
    - family: Raleway
      fonts:
        - asset: packages/awesome_package/fonts/Raleway-Regular.ttf
        - asset: packages/awesome_package/fonts/Raleway-Italic.ttf
          style: italic
```

<FragmentTarget id="use" />

## 3. Usar a font {:#3-use-the-font}

Use um [`TextStyle`][] para mudar a aparência do texto.
Para usar fonts de package, declare qual font você gostaria de usar e
a qual package a font pertence.

<?code-excerpt "lib/main.dart (TextStyle)"?>
```dart
child: Text(
  'Using the Raleway font from the awesome_package',
  style: TextStyle(fontFamily: 'Raleway'),
),
```

## Exemplo completo {:#complete-example}

### Fonts

As fonts Raleway e RobotoMono foram baixadas do
[Google Fonts][].

### `pubspec.yaml`

```yaml
name: package_fonts
description: An example of how to use package fonts with Flutter

environment:
  sdk: ^3.13.0

dependencies:
  awesome_package:
    path: ../awesome_package
  flutter:
    sdk: flutter
  material_ui: ^1.11.0

dev_dependencies:
  flutter_test:
    sdk: flutter

flutter:
  fonts:
    - family: Raleway
      fonts:
        - asset: packages/awesome_package/fonts/Raleway-Regular.ttf
        - asset: packages/awesome_package/fonts/Raleway-Italic.ttf
          style: italic
  uses-material-design: true
```

### `main.dart`

<?code-excerpt "lib/main.dart"?>
```dart
import 'package:material_ui/material_ui.dart';

void main() => runApp(const MyApp());

class MyApp extends StatelessWidget {
  const MyApp({super.key});

  @override
  Widget build(BuildContext context) {
    return const MaterialApp(title: 'Package Fonts', home: MyHomePage());
  }
}

class MyHomePage extends StatelessWidget {
  const MyHomePage({super.key});

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      // The AppBar uses the app-default font.
      appBar: AppBar(title: const Text('Package Fonts')),
      body: const Center(
        // This Text widget uses the Raleway font.
        child: Text(
          'Using the Raleway font from the awesome_package',
          style: TextStyle(fontFamily: 'Raleway'),
        ),
      ),
    );
  }
}
```

![Package Fonts Demo](/assets/images/docs/cookbook/package-fonts.png){:.site-mobile-screenshot}

[Google Fonts]: https://fonts.google.com
[google_fonts]: {{site.pub-pkg}}/google_fonts
[pub.dev]: {{site.pub}}
[`TextStyle`]: {{site.api}}/flutter/painting/TextStyle-class.html
