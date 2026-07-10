# Code Review — Projetos com dados em tempo real do GitHub

**Data**: 2026-07-10
**Arquivos revisados**: `index.html`
**Autor da implementação**: Claude Code (assistido)

---

## Resumo

A section "projects" (scoresheet de xadrez, 6 movimentos) passou a exibir a última alteração ao vivo dos projetos que são repositórios públicos no GitHub. Implementação por progressive enhancement: o HTML curado continua sendo a fonte da verdade e um `<script>` vanilla injeta, ao carregar a página, um badge com tempo relativo do último commit + primeira linha da mensagem. Apenas os 2 projetos públicos (CR-SpaceWalker, BioSpace Explorer) recebem o badge — marcados com `data-repo`; os 4 privados ficam intocados. Qualidade boa, sem achados críticos/altos. XSS mitigado por construção via `textContent`.

## Achados

### 🔴 Críticos
- Nenhum.

### 🟠 Altos
- Nenhum.

### 🟡 Médios
- Nenhum.

### 🟢 Baixos
- O ponto "ao vivo" (`.live-dot`) pisca ininterruptamente e não respeita `prefers-reduced-motion` — consistente com a decisão anterior do átomo (usuário optou por animação sempre ativa). Registrado; sem ação.

### 🔵 Informativo
- **Segurança (XSS)**: a mensagem de commit vem da API e é atacável em tese. Toda injeção usa `textContent` / `createElement` — nunca `innerHTML`. Cenário de teste com `<img onerror>` confirmou que o markup é tratado como texto puro (`window.__xss` permaneceu `undefined`, 0 `<img>` criados).
- **Rate limit**: fetch client-side, 60 req/h por IP de visitante (não global). 2 requests por carga de página. Sem token, sem secrets hardcoded.
- **Degradação graciosa**: `try/catch` por repo + checagem `response.ok`. Em 403/rede/repo movido, o cartão mantém o conteúdo estático — verificado (cenário 403: 0 badges, 6 moves intactos).
- 1 request por repo (endpoint `/commits?per_page=1` traz data E mensagem juntas), em vez de 2 (repo + commits).

## Resultado dos Testes

Verificação end-to-end via Chromium headless (Playwright 1.61.1) — `scratchpad/verify-projects.js` (mock) + teste real contra a API:

- Total executados: 8 cenários mockados + 1 teste real
- Passando: 9
- Falhando: 0
- Cobertura: n/a

Cenários mockados:
1. API OK → 2 badges (1 por público). ✅
2. Tempo relativo em pt-BR ("atualizado anteontem"). ✅
3. Só a 1ª linha da mensagem de commit (corpo descartado). ✅
4. Nenhum projeto privado recebe badge. ✅
5. Ponto ao vivo pulsa (`animationName === live-blink`). ✅
6. Rate limit 403 → 0 badges injetados. ✅
7. 403 → scoresheet estática intacta (6 moves, título visível). ✅
8. XSS → markup do commit tratado como texto (sem execução, sem `<img>`). ✅

Teste real (API pública do GitHub, sem mock): 2 badges renderizados — "atualizado há 2 anos · Add README.md" (CR-SpaceWalker), "atualizado há 9 meses · Backend & new docs" (BioSpace). Zero erros de console. Screenshot conferido: hierarquia visual limpa.

## Decisões Técnicas

- **Progressive enhancement** em vez de renderização client-side da lista: preserva a narrativa curada de xadrez e os 4 projetos privados, que a API pública não retorna. Página funciona 100% sem JS.
- **`data-repo` como opt-in** por movimento: mapeia explicitamente quais cartões são enriquecidos; adicionar/remover um projeto do live é trivial (só o atributo).
- **`/commits?per_page=1`** escolhido sobre `/repos/{repo}`: uma única chamada entrega `committer.date` (última alteração) e `commit.message` (pedido do usuário), metade dos requests.
- **`Intl.RelativeTimeFormat('pt-BR')`**: tempo relativo localizado sem biblioteca.

## Pendências

- Nenhuma.
