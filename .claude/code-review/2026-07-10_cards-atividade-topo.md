# Code Review — Cards de atividade em destaque no topo de #projects

**Data**: 2026-07-10
**Arquivos revisados**: `index.html`
**Autor da implementação**: Claude Code (assistido)

---

## Resumo

A faixa "Live · recent pushes on GitHub" foi promovida de linhas mono discretas no rodapé da section para **cards em grid no topo** (entre a nota da scoresheet e a scoresheet). Cada card traz ponto pulsante, nome amigável linkado, tempo relativo da última alteração, descrição do repo (quando existe) e linguagem. Fetch, filtros e degradação (`hidden` até sucesso) inalterados — zero requests novos, a resposta da API já continha `description`. Um bug foi encontrado e corrigido durante a verificação visual.

## Achados

### 🔴 Críticos
- Nenhum.

### 🟠 Altos
- **Ponto pulsante invisível nos cards** — o CSS do dot estava escopado como `.live-meta .live-dot`; nos cards o dot vive em `.act-head`, ficando sem estilo (span de 0px). **Encontrado via screenshot na verificação e corrigido**: seletor generalizado para `.live-dot`. Re-testado: `{w: 7px, anim: live-blink}` nos cards e badges.

### 🟡 Médios
- **Grid quebrava em 2 colunas apertadas a 600px** (`minmax(15rem, 1fr)` cabia duplo nos 552px úteis). **Corrigido**: `minmax(17rem, 1fr)` → 1 coluna em mobile, 3 em desktop; + `overflow-wrap: anywhere` no nome para repos de nome longo. Detectado pela suite (cenário mobile), corrigido e re-testado.

### 🟢 Baixos
- Descrições dos repos vêm em pt-BR do GitHub ("Desafio -> Construir...") numa página em inglês — conteúdo do usuário no GitHub, não do site; corrigível editando a descrição no próprio repo. Sem ação no código.

### 🔵 Informativo
- Nome amigável via mapa `FRIENDLY` (chave lowercase): só `neur.AI_..._2025` → "BioSpace Explorer" hoje; demais usam fallback do nome cru. Extensível por uma linha.
- Segurança inalterada: `textContent` para nome/descrição (teste XSS com `<img onerror>` na descrição passou), links `target=_blank rel=noopener`.
- `.act-lang` com `margin-top: auto` alinha a linguagem ao pé do card em alturas desiguais.

## Resultado dos Testes

Chromium headless (Playwright 1.61.1) — `scratchpad/verify-activity.js` (mock) + teste real:

- Total executados: 17 cenários mockados + 1 teste real
- Passando: 18
- Falhando: 0
- Cobertura: n/a

Destaques: posição no DOM (activity antes da scoresheet), nome amigável + fallback, descrição condicional, filtros fork/denylist, ordem por push, 403 → `hidden`, vazio → `hidden`, XSS na descrição, grid 3 colunas desktop / 1 coluna mobile, sem overflow horizontal. Teste real: 4 cards com descrições reais da API, zero erros de console. Screenshots desktop e mobile conferidos.

## Decisões Técnicas

- **Zero requests novos**: `description` já vinha na resposta de `/users/{user}/repos` — só renderização mudou.
- **`minmax(17rem, 1fr)`** calibrado pelo container (`.wrap` 60rem − padding): 3 colunas desktop, 1 em ≤600px, sem media query dedicada.
- **Screenshot como parte da verificação**: o bug do dot invisível passaria em todos os asserts de DOM — só a inspeção visual pegou. Reforça o padrão de sempre conferir screenshot além dos asserts.

## Pendências

- Nenhuma.
