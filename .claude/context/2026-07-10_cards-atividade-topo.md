# Contexto — Cards de atividade em destaque no topo de #projects

**Data**: 2026-07-10
**Status**: ✅ Concluída

---

## Descrição da Tarefa

Usuário pediu (via /plan) que a section `#projects` mostrasse "de forma atualizada os repositórios que estão recebendo modificações". Clarificação por AskUserQuestion → opção escolhida: **"Faixa → cards em destaque"** — promover a faixa de atividade existente para cards ricos no topo da section, mantendo a scoresheet curada abaixo.

## O que foi implementado

1. **HTML**: bloco `.activity` movido de depois do `</ol>` para entre `.scoresheet-note` e `<ol class="scoresheet">`.
2. **JS** (mesma lógica de dados, zero requests novos): mapa `FRIENDLY` para nomes amigáveis (`neur.ai_..._2025` → "BioSpace Explorer", fallback nome cru); card com `.act-head` (dot + link + tempo relativo), `.act-desc` (descrição da API, omitida se null) e `.act-lang` (omitida se null).
3. **CSS**: `.activity-list` vira grid `repeat(auto-fill, minmax(17rem, 1fr))` (3 col desktop, 1 col mobile); `.activity-item` vira card (borda `--linha`, radius 8px, hover borda cobre); `.act-desc` com line-clamp 2; `.act-lang` com `margin-top: auto`.

## Bugs encontrados e corrigidos durante verificação

- 🟠 **Dot invisível nos cards**: CSS era `.live-meta .live-dot` (escopado ao badge da scoresheet); generalizado para `.live-dot`. Só o screenshot pegou — asserts de DOM passavam.
- 🟡 **2 colunas apertadas em 600px**: `minmax(15rem→17rem)` + `overflow-wrap: anywhere` no nome.

## Arquivos Modificados

| Arquivo      | Tipo de Mudança |
| ------------ | --------------- |
| `index.html` | Modificado (posição do bloco `.activity`, CSS `.activity*`/`.act-*`/`.live-dot`, renderização no `<script>`) |

## Dependências Adicionadas

| Pacote    | Versão | Justificativa |
| --------- | ------ | ------------- |
| (nenhuma) | —      | — |

## Padrões Seguidos

- Progressive enhancement mantido (`hidden` até sucesso do fetch).
- `textContent` para dados da API; `rel=noopener` em links externos.
- Verificação Playwright mock + real + screenshots (desktop e mobile).

## Impacto

- **Breaking changes**: Não
- **Requer migration**: Não
- **Requer variável de ambiente nova**: Não
- **Requer rebuild/deploy**: Sim (republicar `index.html`)

## Aprendizados para Próximas Tarefas

- **Sempre conferir screenshot além dos asserts** — o bug do dot invisível passava em todos os asserts de DOM (elemento existia, classe certa); só a inspeção visual revelou.
- Ao mover/reusar um componente estilizado, checar se os seletores CSS estão escopados ao contexto antigo (`.live-meta .live-dot` quebrou fora do `.live-meta`).
- `minmax` do grid deve ser calibrado pelo container real (`.wrap` = 60rem − 3rem padding): 17rem dá 3 colunas desktop e 1 em 600px sem media query.
- Nomes amigáveis: mapa `FRIENDLY` no script (chave = nome do repo lowercase); adicionar entrada nova é uma linha.

## Link com Code Review

→ `.claude/code-review/2026-07-10_cards-atividade-topo.md`
