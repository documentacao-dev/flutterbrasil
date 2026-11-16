---
ia-translate: true
title: Google APIs
description: Como usar Google APIs com Flutter.
---

<?code-excerpt path-base="googleapis/"?>

O [pacote Google APIs][Google APIs package] expõe dezenas de serviços do Google
que você pode usar em projetos Dart.

Esta página descreve como usar APIs que interagem com
dados do usuário final usando autenticação do Google.

Exemplos de APIs de dados de usuário incluem
[Calendar][], [Gmail][], [YouTube][] e Firebase.

:::note
As únicas APIs que você deve usar diretamente do seu projeto Flutter
são aquelas que acessam dados do usuário usando autenticação do Google.

APIs que requerem [contas de serviço][service accounts] **não devem**
ser usadas diretamente de uma aplicação Flutter.
Fazer isso requer enviar credenciais de serviço como parte
da sua aplicação, o que não é seguro.
Para usar essas APIs,
recomendamos criar um serviço intermediário.
:::

Para adicionar autenticação ao Firebase explicitamente, confira o
codelab [Adicionar um fluxo de autenticação de usuário a um app Flutter usando FirebaseUI][fb-lab]
e a documentação
[Começar com Firebase Authentication no Flutter][fb-auth].

[fb-lab]: {{site.firebase}}/codelabs/firebase-auth-in-flutter-apps
[Calendar]: https://developers.google.com/workspace/calendar/api/guides/overview
[fb-auth]: {{site.firebase}}/docs/auth/flutter/start
[Gmail]: https://developers.google.com/workspace/gmail/api/guides
[Google APIs package]: {{site.pub-pkg}}/googleapis
[service accounts]: https://cloud.google.com/iam/docs/service-account-overview
[YouTube]: https://developers.google.com/youtube/v3

## Visão geral {:#overview}

Para usar Google APIs, siga estas etapas:

1. Escolha a API desejada
1. Habilite a API
1. Autentique e determine o usuário atual
1. Obtenha um cliente HTTP autenticado
1. Crie e use a classe de API desejada

## 1. Escolha a API desejada {:#1-pick-the-desired-api}

A partir do diretório do seu projeto Flutter, adicione `googleapis`
e os pacotes de login e autorização.

```console
$ flutter pub add googleapis googleapis_auth
$ flutter pub add google_sign_in extension_google_sign_in_as_googleapis_auth
```

O [pacote Google APIs][Google APIs package] lista as APIs disponíveis e seus caminhos de import.
Cada biblioteca fornece uma classe cliente, como [`YouTubeApi`][],
que expõe os recursos da API e os escopos de autorização.
Para ler os dados do YouTube do usuário conectado, solicite [`youtubeReadonlyScope`][].

<?code-excerpt "lib/main.dart (youtube-import)"?>
```dart
// Provides the `YouTubeApi` class.
import 'package:googleapis/youtube/v3.dart';
```

[`YouTubeApi`]: {{site.pub-api}}/googleapis/latest/youtube_v3/YouTubeApi-class.html
[`youtubeReadonlyScope`]: {{site.pub-api}}/googleapis/latest/youtube_v3/YouTubeApi/youtubeReadonlyScope-constant.html

## 2. Habilite a API {:#2-enable-the-api}

Para usar Google APIs você deve ter uma conta Google
e um projeto Google. Você também
precisa habilitar a API desejada.

Este exemplo habilita [YouTube Data API v3][].
Para detalhes, veja as [instruções de primeiros passos][getting started instructions].

[getting started instructions]: https://cloud.google.com/apis/docs/getting-started
[YouTube Data API v3]: https://console.cloud.google.com/apis/library/youtube.googleapis.com

## 3. Autentique e determine o usuário atual {:#3-authenticate-and-determine-the-current-user}

Use o pacote [google_sign_in][gsi-pkg] para
autenticar usuários com sua identidade Google.
Configure o sign in para cada plataforma que você deseja suportar.

<?code-excerpt "lib/main.dart (google-import)"?>
```dart
// Provides the `GoogleSignIn` class.
import 'package:google_sign_in/google_sign_in.dart';
```

Guarde o future de inicialização de [`GoogleSignIn`][] para que o app inicialize o plugin apenas uma vez.
Use um `FutureBuilder` para mostrar o progresso enquanto o plugin inicializa e exibir um erro se a inicialização falhar.
Mostre os controles de login quando a inicialização tiver sucesso.

<?code-excerpt "lib/main.dart (init)"?>
```dart
class _LikedVideosPage extends StatelessWidget {
  const _LikedVideosPage();

  static final Future<void> _initialization = GoogleSignIn.instance
      .initialize();

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(title: const Text(_title)),
      body: Center(
        child: FutureBuilder<void>(
          future: _initialization,
          builder: (context, snapshot) {
            if (snapshot.error case final error?) {
              return Text(
                error is UnimplementedError
                    ? 'Google Sign-In is unavailable on this platform.'
                    : 'Check your Google Sign-In configuration.\n$error',
              );
            }
            if (snapshot.connectionState != ConnectionState.done) {
              return const CircularProgressIndicator();
            }
            return const _LikedVideosWidget();
          },
        ),
      ),
    );
  }
}
```

Subscribe to `authenticationEvents` before calling
`attemptLightweightAuthentication()` so your app receives a sign-in event
if the plugin restores a previous session.
Import `dart:async` for `StreamSubscription` and `unawaited`.
Cancel the subscription when the widget is disposed.

<?code-excerpt "lib/main.dart (post-init)" plaster="none"?>
```dart
final GoogleSignIn _googleSignIn = GoogleSignIn.instance;
late final StreamSubscription<GoogleSignInAuthenticationEvent> _subscription;

@override
void initState() {
  super.initState();
  _subscription = _googleSignIn.authenticationEvents.listen(
    _onAuthenticationEvent,
    onError: (Object error) => _snackbarError(error.toString()),
  );
  unawaited(_restoreSignIn());
}

Future<void> _restoreSignIn() async {
  try {
    await _googleSignIn.attemptLightweightAuthentication();
  } on Exception catch (error) {
    _snackbarError(error.toString());
  }
}

@override
void dispose() {
  unawaited(_subscription.cancel());
  super.dispose();
}
```

Ao fazer login, verifique se a conta já tem acesso ao YouTube.
Ao sair ou trocar de conta, limpe os vídeos da conta anterior.

<?code-excerpt "lib/main.dart (auth-event)"?>
```dart
void _onAuthenticationEvent(GoogleSignInAuthenticationEvent event) {
  final user = switch (event) {
    GoogleSignInAuthenticationEventSignIn() => event.user,
    GoogleSignInAuthenticationEventSignOut() => null,
  };
  setState(() {
    _currentUser = user;
    _favoriteVideos = null;
    _loadingVideos = false;
  });
  if (user != null) {
    unawaited(_loadLikedVideos(user));
  }
}
```

Para login interativo, chame `authenticate()` a partir de um toque em botão
nas plataformas em que `supportsAuthenticate()` retorna true.
Para apps web, execute `flutter pub add google_sign_in_web` e
[renderize o botão de login fornecido pelo Google][web-sign-in].
Ambos os fluxos informam a conta conectada por meio de `authenticationEvents`.

[web-sign-in]: {{site.pub-pkg}}/google_sign_in_web#authentication
[gsi-pkg]: {{site.pub-pkg}}/google_sign_in
[`GoogleSignIn`]: {{site.pub-api}}/google_sign_in/latest/google_sign_in/GoogleSignIn-class.html

## 4. Obtenha um cliente HTTP autenticado {:#4-obtain-an-authenticated-http-client}

Ter um usuário conectado identifica a conta.
Para ler os vídeos que ela curtiu, solicite o escopo somente leitura do YouTube.

<?code-excerpt "lib/main.dart (youtube-scopes)"?>
```dart
const List<String> _youtubeScopes = [YouTubeApi.youtubeReadonlyScope];
```

Quando um usuário faz login, chame [`authorizationForScopes`][] para verificar
uma permissão existente sem abrir um pedido de consentimento.
Se ele retornar `null`, deixe que o usuário escolha quando conceder o acesso.

<?code-excerpt "lib/main.dart (request-access)"?>
```dart
ElevatedButton(
  onPressed: () => _loadLikedVideos(currentUser, requestAccess: true),
  child: const Text('Load liked videos'),
),
```

Somente o manipulador do botão define `requestAccess` como `true`.
Ele chama [`authorizeScopes`][] como parte dessa interação do usuário.
Se o usuário recusar, mantenha o botão disponível para que ele possa tentar novamente.

<?code-excerpt "lib/main.dart (scope-authorize)"?>
```dart
final authorizationClient = user.authorizationClient;
final authorization = requestAccess
    ? await authorizationClient.authorizeScopes(_youtubeScopes)
    : await authorizationClient.authorizationForScopes(_youtubeScopes);
if (!mounted || !identical(user, _currentUser)) return;
if (authorization != null) {
  videos = await _downloadLikedList(authorizationClient, authorization);
}
```

Depois de ter os tokens de autorização relevantes,
use a extensão [`authClient`][] de
[`package:extension_google_sign_in_as_googleapis_auth`][] para
configurar um cliente HTTP autenticado com as credenciais relevantes aplicadas.

<?code-excerpt "lib/main.dart (auth-import)"?>
```dart
import 'package:extension_google_sign_in_as_googleapis_auth/extension_google_sign_in_as_googleapis_auth.dart';
```

<?code-excerpt "lib/main.dart (auth-client)"?>
```dart
final authenticatedClient = authorization.authClient(
  scopes: _youtubeScopes,
);
```

[`authorizationForScopes`]: {{site.pub-api}}/google_sign_in/latest/google_sign_in/GoogleSignInAuthorizationClient/authorizationForScopes.html
[`authorizeScopes`]: {{site.pub-api}}/google_sign_in/latest/google_sign_in/GoogleSignInAuthorizationClient/authorizeScopes.html
[`authClient`]: {{site.pub-api}}/extension_google_sign_in_as_googleapis_auth/latest/extension_google_sign_in_as_googleapis_auth/GoogleApisGoogleSignInAuth/authClient.html
[`package:extension_google_sign_in_as_googleapis_auth`]: {{site.pub-pkg}}/extension_google_sign_in_as_googleapis_auth

## 5. Crie e use a classe de API desejada {:#5-create-and-use-the-desired-api-class}

Passe o cliente autenticado para `YouTubeApi` e solicite os vídeos
curtidos do usuário conectado.
Se o YouTube rejeitar o token de acesso, remova-o do cache de login
para que o próximo toque no botão possa solicitar um token novo.
Para tratar um desafio de autenticação do cliente HTTP,
importe e verifique `AccessDeniedException` de
`package:googleapis_auth/googleapis_auth.dart`.
Feche o cliente quando a requisição terminar, mesmo que ela falhe.

<?code-excerpt "lib/main.dart (playlist)"?>
```dart
try {
  final youTubeApi = YouTubeApi(authenticatedClient);
  final favorites = await youTubeApi.playlistItems.list(
    ['snippet'],
    playlistId: 'LL', // Liked List
  );
  return [
    for (final item in favorites.items ?? <PlaylistItem>[]) ?item.snippet,
  ];
} on Exception catch (error) {
  if (error is AccessDeniedException ||
      (error is DetailedApiRequestError && error.status == 401)) {
    await authorizationClient.clearAuthorizationToken(
      accessToken: authorization.accessToken,
    );
  }
  rethrow;
} finally {
  authenticatedClient.close();
}
```

## Mais informações {:#more-information}

* [Exemplo completo de login no YouTube e vídeos curtidos][youtube-example]
* [Exemplo da Google People API usando o mesmo fluxo de login][auth-ex]

[youtube-example]: {{site.repo.this}}/tree/main/examples/googleapis
[auth-ex]: {{site.pub-pkg}}/extension_google_sign_in_as_googleapis_auth/example
