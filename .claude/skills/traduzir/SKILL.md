---
name: traduzir
description: >-
  Roda o ciclo completo de tradução do Flutter Brasil de ponta a ponta: sync com
  o upstream (flutter/website), retradução das páginas que mudaram e tradução das
  páginas novas em subagentes Haiku, verificação determinística, commit, PR,
  review independente em Sonnet, correção em Haiku e auto-merge quando tudo fica
  verde. Use quando pedirem "traduzir", "sincronizar com o upstream", "rodar o
  sync", "atualizar a tradução", "o inglês mudou", "tem página defasada", ou
  quando o workflow semanal traducao-automatica.yml chamar. Também serve para
  traduzir só um conjunto de páginas: passe os caminhos como argumento.
---

# Ciclo de tradução do Flutter Brasil

Você é o **orquestrador**. Quem traduz é sempre um subagente **Haiku**; quem
revisa é um subagente **Sonnet** de contexto limpo. Você não escreve tradução:
cuida de git, scripts, lotes, commits, PR e da decisão de merge.

Argumentos opcionais (`$ARGUMENTS`): caminhos de páginas para traduzir só
elas, ou `sem-merge` para parar no PR pronto, sem auto-merge.

Leia antes: `CLAUDE.md` (processo e scripts de `tool/traducao/`) e as
convenções em `.claude/agents/translate-pt-br.md`.

## 0. Ambiente

```console
bash tool/traducao/setup.sh          # só se não houver Dart/upstream (sessão nova, CI)
export PATH="$HOME/.dart-sdk/bin:$HOME/.pub-cache/bin:$PATH"
git fetch upstream main
```

Árvore limpa e `main` atualizada. Se já existir um PR aberto com o label
`traducao-automatica`, **não abra outro**: continue nele (faça checkout da
branch dele e pule para a etapa que faltar).

## 1. Detectar o que fazer

- `git rev-list --count $(cat tool/traducao/ultimo-sync-upstream.txt)..upstream/main -- sites/docs`
  diz quantos commits do upstream mexeram nas docs desde o último sync.
- `TODO-TRADUCAO.md` lista as páginas ainda não traduzidas.
- Se não há commit novo do upstream nem página pendente (e nenhum caminho foi
  passado), **pare aqui** e responda "nada a traduzir". Não abra PR vazio.

Crie a branch `traducao/AAAA-MM-DD` a partir da `main`.

## 2. Sync com o upstream

Siga a "Regra principal" do `CLAUDE.md`, passos 2 e 3: rode
`python3 tool/traducao/sincronizar_upstream.py` e faça o commit do merge
**incluindo** `.traducao-pendente/`.

Arquivos que não são prosa (código Dart, workflows, `firebase.json`, dados de
configuração) você resolve direto, sem agente, como diz o `CLAUDE.md`.

## 3. Traduzir em Haiku

Monte lotes equilibrados por tamanho, **no máximo 5 subagentes em paralelo**,
nunca dois no mesmo arquivo. Sempre com `model: "haiku"` na ferramenta Agent:

- Páginas em `.traducao-pendente/` → subagente com o prompt de
  `.claude/agents/atualizar-traducao.md`.
- Páginas novas (do `TODO-TRADUCAO.md` ou dos argumentos) → subagente com o
  prompt de `.claude/agents/translate-pt-br.md`.
- Receitas em que o inglês só ganhou uma linha de include ou um comentário
  dentro do código: resolva por script, sem tradução.

Depois: `python3 tool/traducao/ancorar_titulos.py <arquivos traduzidos>`,
`python3 tool/traducao/traduzir_release_notes.py --todas` e, com
`.traducao-pendente/` vazio, apague a pasta e rode
`python3 tool/traducao/gerar_todo.py`.

## 4. Verificar

```console
python3 tool/traducao/verificar.py
dart run dash_site --site=docs refresh-excerpts --fail-on-update --dry-run
```

Zero erros é obrigatório. Erro num arquivo traduzido volta para um subagente
Haiku com a saída do `verificar.py` no prompt (no máximo 2 tentativas por
arquivo; depois corrija o trecho você mesmo, sem trocar de modelo). Procure
também prosa em inglês que escapou do limite de 30% do `verificar.py`
(seções novas inteiras em inglês) e mande para o Haiku.

## 5. Commit e PR

- Commits por lote, mensagens em português no padrão do repositório
  (`chore: sync com upstream <sha>`, `translate: ...`).
- `git push -u origin traducao/AAAA-MM-DD`
- `gh pr create --base main --label traducao-automatica` com o corpo:
  commits do upstream (de → para), páginas retraduzidas, páginas novas,
  resolvidas por script, e o que ficou de fora.

## 6. Review em Sonnet (contexto limpo)

Dispare o subagente `revisar-traducao` (`.claude/agents/revisar-traducao.md`,
`model: "sonnet"`) passando **só** a lista de arquivos traduzidos ou alterados
neste ciclo e o commit do upstream. Não passe sua opinião nem o resumo do que
foi feito. Até 5 revisores em paralelo para listas grandes.

O revisor devolve, por arquivo, `OK` ou problemas classificados como
**grave** (sentido trocado, trecho faltando, código/link/âncora alterado,
inglês sem tradução) ou **leve** (estilo, escolha de palavra).

Publique o resultado como um comentário no PR (`gh pr comment`), uma linha
por arquivo com problema.

## 7. Corrigir em Haiku

Problemas **graves** voltam para subagentes Haiku com o apontamento do
revisor no prompt. Leves só se forem baratos. Depois, `verificar.py` de novo,
commit `fix: correções do review`, push, e um **novo review só dos arquivos
corrigidos**. No máximo 2 voltas de review + correção.

## 8. Aprovação e merge

Auto-merge **só** se as três condições valerem:

1. `verificar.py` com 0 erros;
2. o último review sem nenhum problema grave em aberto;
3. o argumento `sem-merge` não foi passado.

Então:

```console
gh pr edit <n> --add-label revisao-ok
gh pr merge <n> --auto --squash --delete-branch
```

O merge acontece sozinho quando os checks obrigatórios da `main`
(Verificar traduções, Build and check links, Code excerpts) ficarem verdes.

Se alguma condição falhar: `gh pr edit <n> --add-label precisa-humano`, um
comentário dizendo exatamente o que ficou pendente, e **sem** auto-merge.

## 9. Resposta final

Uma mensagem curta: link do PR, de qual para qual commit do upstream, quantas
páginas retraduzidas / novas / por script, resultado do review (graves e
leves), e se ficou em auto-merge ou com `precisa-humano`.
