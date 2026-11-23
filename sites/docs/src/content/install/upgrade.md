---
title: Atualizar o Flutter
shortTitle: Atualizar
description: Aprenda como atualizar o Flutter e mudar para outro canal.
ia-translate: true
---

Não importa qual dos canais de lançamento do Flutter
você segue, você pode usar o comando `flutter` para atualizar seu
Flutter SDK ou os packages dos quais seu app depende.

<FragmentTarget id="upgrading-the-flutter-sdk" />

## Atualizar o Flutter SDK {:#upgrade-the-flutter-sdk}

Para atualizar o Flutter SDK use o comando `flutter upgrade`:

```console
$ flutter upgrade
```

Este comando obtém a versão mais recente do Flutter SDK
que está disponível no seu canal Flutter atual.

Se você está usando o canal **stable**
e quer uma versão ainda mais recente do Flutter SDK,
mude para o canal **beta** usando `flutter channel beta`,
e então execute `flutter upgrade`.

### Mantenha-se informado {:#keep-informed}

Nós publicamos [guias de migração][migration guides] para mudanças breaking conhecidas.

Nós enviamos anúncios sobre essas mudanças para a
[lista de e-mails de anúncios do Flutter][flutter-announce].

Para evitar ser quebrado por versões futuras do Flutter,
considere submeter seus testes ao nosso [registro de testes][test registry].


<a id="switching-flutter-channels" aria-hidden="true"></a>

## Mudando canais do Flutter {:#switching-flutter-channels}

O Flutter tem dois canais de lançamento:
**stable** e **beta**.

### O canal **stable** {:#the-stable-channel}

Nós recomendamos o canal **stable** para novos usuários
e para lançamentos de apps em produção.
A equipe atualiza este canal aproximadamente a cada três meses.
O canal pode receber hot fixes ocasionais
para problemas de alta severidade ou alto impacto.

A integração contínua para plugins e packages da equipe Flutter
inclui testes contra o lançamento **stable** mais recente.

A documentação mais recente para o branch **stable**
está em: <https://api.flutter.dev>

<a id="the-beta-channel" aria-hidden="true"></a>

### O canal **beta** {:#the-beta-channel}

O canal **beta** tem o lançamento estável mais recente.
Esta é a versão mais recente do Flutter que testamos intensamente.
Este canal passou por todos os nossos testes públicos,
foi verificado contra suítes de teste para produtos Google que usam Flutter,
e foi examinado contra [suítes de teste privadas contribuídas][test registry].
O canal **beta** recebe hot fixes regulares
para resolver problemas importantes recém-descobertos.

O canal **beta** é essencialmente o mesmo que o canal **stable**
mas atualizado mensalmente em vez de trimestralmente.
De fato, quando o canal **stable** é atualizado,
ele é atualizado para o lançamento **beta** mais recente.

### Outros canais {:#other-channels}

Atualmente temos um outro canal, **main** (anteriormente conhecido como **master**).
Pessoas que [contribuem para o Flutter][contribute to Flutter] usam este canal.

Este canal não é tão completamente testado quanto
os canais **beta** e **stable**.

Nós não recomendamos usar este canal já que
é mais provável que contenha regressões sérias.

A documentação mais recente para o branch **main**
está em: <https://main-api.flutter.dev>

<FragmentTarget id="changing-channels" />

### Mudar canais {:#change-channels}

Para visualizar seu canal atual, use o seguinte comando:

```console
$ flutter channel
```

Para mudar para outro canal, use `flutter channel <channel-name>`.
Uma vez que você mudou seu canal, use `flutter upgrade`
para baixar o Flutter SDK mais recente e packages dependentes para aquele canal.
Por exemplo:

```console
$ flutter channel beta
$ flutter upgrade
```

<FragmentTarget id="switching-to-a-specific-flutter-version" />

## Mudar para uma versão específica do Flutter {:#switch-to-a-specific-flutter-version}

Para mudar para uma versão específica do Flutter:

1. Encontre sua **versão Flutter** desejada no [arquivo do Flutter SDK][Flutter SDK archive].

1. Navegue até o Flutter SDK:

   ```console
   $ cd /path/to/flutter
   ```

   :::tip
   Você pode encontrar o caminho do Flutter SDK usando `flutter doctor --verbose`.
   :::

1. Use `git checkout` para mudar para sua **versão Flutter** desejada:

   ```console
   $ git checkout <Flutter version>
   ```

<FragmentTarget id="upgrading-packages" />

## Atualizar packages {:#upgrade-packages}

Se você modificou seu arquivo `pubspec.yaml`, ou você quer atualizar
apenas os packages dos quais seu app depende
(em vez de tanto os packages quanto o Flutter em si),
então use um dos comandos `flutter pub`.

Para atualizar para as _versões compatíveis mais recentes_ de
todas as dependências listadas no arquivo `pubspec.yaml`,
use o comando `upgrade`:

```console
$ flutter pub upgrade
```

Para atualizar para a _versão mais recente possível_ de
todas as dependências listadas no arquivo `pubspec.yaml`,
use o comando `upgrade --major-versions`:

```console
$ flutter pub upgrade --major-versions
```

Isso também atualiza automaticamente as constraints
no arquivo `pubspec.yaml`.

Para identificar dependências de packages desatualizadas e obter conselhos
sobre como atualizá-las, use o comando `outdated`. Para detalhes, veja
a [documentação `pub outdated` do Dart]({{site.dart-site}}/tools/pub/cmd/pub-outdated).

```console
$ flutter pub outdated
```

## Solução de problemas {:#troubleshooting}

### Windows: erro "Filename too long" {:#windows-filename-too-long-error}

Ao executar `flutter upgrade` no Windows,
você pode encontrar um erro como o seguinte:

```text
error: unable to create file ...: Filename too long
```

Isso acontece porque o caminho de um arquivo no Flutter SDK excede o
limite máximo padrão de tamanho de caminho no Windows.

Para resolver este problema, considere instalar o Flutter SDK
em um caminho mais curto. Por exemplo, instale o Flutter em
`C:\Flutter` em vez de algo mais longo como
`C:\Users\<user name>\Documents\flutter`.

Caso contrário, faça o seguinte:

1. Ative o suporte a caminhos longos no Git:

   ```console
   $ git config --system core.longpaths true
   ```

   Se o comando falhar com um erro de permissão,
   tente executar seu terminal como administrador.

1. Ative os caminhos longos no Windows:

   ```console
   New-ItemProperty -Path "HKLM:\SYSTEM\CurrentControlSet\Control\FileSystem" -Name "LongPathsEnabled" -Value 1 -PropertyType DWORD -Force
   ```

   Este comando exige privilégios de administrador.

[Flutter SDK archive]: /install/archive
[flutter-announce]: {{site.groups.flutter-announce}}
[pubspec.yaml]: {{site.dart-site}}/tools/pub/pubspec
[test registry]: {{site.repo.organization}}/tests
[contribute to Flutter]: {{site.repo.flutter}}/blob/main/CONTRIBUTING.md
[migration guides]: /release/breaking-changes
