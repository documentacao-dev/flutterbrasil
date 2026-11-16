---
title: Arquivo do Flutter SDK
shortTitle: Archive
description: "Todos os lançamentos atuais do Flutter SDK: stable, beta e main."
ia-translate: true
---

{% render "docs/china-notice.md" %}

## Visão geral {:#overview}

O arquivo do Flutter SDK é uma coleção de todas as versões anteriores do
Flutter SDK. Este arquivo é útil para desenvolvedores que precisam usar uma
versão mais antiga do Flutter por razões de compatibilidade ou para investigar bugs.

Depois de baixar uma versão, para configurar o Flutter,
siga as [instruções de instalação manual][manual-install].

O arquivo inclui Flutter SDKs para Windows, macOS e Linux nos
seguintes [channels][]:

*   **Canal estável (Stable channel)**: Este canal contém as builds mais estáveis do Flutter.
    Aproximadamente a cada terceira versão beta é promovida para a versão estável.
    O canal estável é o canal recomendado para
    novos usuários e para lançamentos de aplicativos em produção.

*   **Canal beta (Beta channel)**: Este canal é a versão mais recente do Flutter que está
    disponível, mas ainda não é estável. O branch beta é geralmente lançado
    na primeira quarta-feira do mês. Uma correção normalmente chega ao
    canal beta cerca de duas semanas após ser integrada ao canal main.
    Os lançamentos são distribuídos como [installation bundles][].

*   **Canal main (Main channel)**: Este canal tem os recursos mais novos, mas não foi totalmente
    testado e pode ter alguns bugs. Não recomendamos usá-lo, a menos que você esteja
    contribuindo para o próprio Flutter.

As seguintes informações estão disponíveis para cada lançamento do Flutter no
arquivo do SDK:

*   **Versão do Flutter (Flutter version)**: O número da versão do Flutter SDK
    (por exemplo, 3.35.0, 2.10.5) segue um esquema modificado de
    [versionamento de calendário][calendar versioning] chamado _CalVer_.
    Para mais informações, visite a página [Flutter SDK versioning][].
*   **Arquitetura (Architecture)**: A arquitetura do processador para a qual o SDK foi construído
    (por exemplo, x64, arm64). Isso especifica o tipo de processador com o qual o SDK é
    compatível.
*   **Ref**: O hash do commit git que identifica exclusivamente a base de código específica
    usada para aquele lançamento.
*   **Data de lançamento (Release Date)**: A data em que aquela versão específica do Flutter foi
    oficialmente lançada.
*   **Versão do Dart (Dart version)**: A versão correspondente do Dart SDK incluída no
    lançamento do Flutter SDK.
*   **Proveniência (Provenance)**: Fornece detalhes sobre o processo de build e a origem do
    SDK, potencialmente incluindo informações sobre atestações de segurança ou
    sistemas de build usados. Os resultados são retornados como JSON.

[calendar versioning]: https://calver.org/
[Flutter SDK versioning]: {{site.repo.flutter}}/blob/main/docs/releases/Release-versioning.md

## Janelas públicas de lançamento {:#public-release-windows}

Previsibilidade é fundamental para integrar recursos complexos com segurança.
Usamos janelas públicas de lançamento para dar à comunidade
a visibilidade necessária para planejar com antecedência.
Ao destacar explicitamente as datas de corte de branch e as metas de lançamento,
permitimos que todos alinhem seus ciclos de desenvolvimento e
coordenem a integração de recursos de forma eficaz.

<a id="what-is-a-branch-cutoff-date" aria-hidden="true"></a>

### O que é uma data de corte de branch? {:#branch-cutoff}

Esta data é o prazo final para que pull requests sejam
integrados nos branches padrão (`main` para Dart e `master` para Flutter) para
garantir a inclusão no próximo lançamento estável.

* **Antes do corte:** Seu PR será lançado na próxima versão estável.
* **Depois do corte:** Seu PR aguardará o ciclo seguinte.

<a id="2026-schedule" aria-hidden="true"></a>

### Cronograma de lançamentos de 2027 {:#release-schedule}

A tabela a seguir lista o mês de lançamento previsto de cada versão estável
do Dart e do Flutter em 2027 e a [data de corte de branch](#branch-cutoff)
de cada uma:

| Meta de lançamento estável | Data de corte de branch |
|----------------------------|-------------------------|
| Fevereiro de 2027          | 5 de janeiro de 2027    |
| Maio de 2027               | 6 de abril de 2027      |
| Agosto de 2027             | 6 de julho de 2027      |
| Novembro de 2027           | 5 de outubro de 2027    |

{:.table}

---

## Canal estável (Stable channel) {:#stable-channel}

<Tabs key="os-archive-tabs">
    <Tab name="Windows">
        <ArchiveTable os="Windows" channel="stable" />
    </Tab>
    <Tab name="macOS">
        <ArchiveTable os="macOS" channel="stable" />
    </Tab>
    <Tab name="Linux">
        <ArchiveTable os="Linux" channel="stable" />
    </Tab>
</Tabs>

## Canal beta (Beta channel) {:#beta-channel}

<Tabs key="os-archive-tabs">
    <Tab name="Windows">
        <ArchiveTable os="Windows" channel="beta" />
    </Tab>
    <Tab name="macOS">
        <ArchiveTable os="macOS" channel="beta" />
    </Tab>
    <Tab name="Linux">
        <ArchiveTable os="Linux" channel="beta" />
    </Tab>
</Tabs>

:::tip
Para ver o que mudou em uma versão beta, compare as tags de versão no GitHub.

1. Encontre o número da versão (tag) que você quer ver (por exemplo, `3.38.0-0.2.pre`).
2. Encontre o número da versão anterior (por exemplo, `3.38.0-0.1.pre`).
3. Acesse a [página de comparação do GitHub](https://github.com/flutter/flutter/compare).
4. Selecione a tag mais antiga no campo `base` e a tag mais recente no campo `compare`.

Por exemplo: [`flutter/flutter@3.38.0-0.1.pre...3.38.0-0.2.pre`](https://github.com/flutter/flutter/compare/3.38.0-0.1.pre...3.38.0-0.2.pre)
:::

<a id="master-channel" aria-hidden="true"></a>

## Canal main (Main channel) {:#main-channel}

[Pacotes de instalação][Installation bundles] não estão disponíveis para o canal `main`
(que anteriormente era conhecido como canal `master`).
No entanto, você pode obter o SDK diretamente do
[repositório no GitHub][GitHub repo] clonando o canal `main` e então
disparando um download das dependências do SDK:

```console
$ git clone -b main https://github.com/flutter/flutter.git
$ ./flutter/bin/flutter --version
```

<a id="install-from-an-archive" aria-hidden="true"></a>

## Mais informações {:#more-information}

- Para instruções sobre como instalar o Flutter a partir de um SDK baixado,
  veja [Instalar o Flutter manualmente][manual-install].

- Para saber o que há de novo nas principais builds do Flutter,
  confira a página de [notas de lançamento][release notes].

- Para detalhes sobre como nossos pacotes de instalação são estruturados,
  consulte [Installation bundles][].

[channels]: {{site.repo.flutter}}/blob/main/docs/releases/Flutter-build-release-channels.md
[manual-install]: /install/manual
[release notes]: /release/release-notes
[GitHub repo]: {{site.repo.flutter}}
[Installation bundles]: {{site.repo.flutter}}/blob/main/docs/infra/Flutter-Installation-Bundles.md
