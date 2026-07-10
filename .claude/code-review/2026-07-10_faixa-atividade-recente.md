# Code Review — Faixa "atividade recente no GitHub"

**Data**: 2026-07-10
**Arquivos revisados**: `index.html`
**Autor da implementação**: Claude Code (assistido)

---

## Resumo

Adicionada uma faixa "Live · recent pushes on GitHub" ao final da section `projects`, que lista ao vivo os repositórios públicos não-fork mais recentemente atualizados (top 5), ordenados por push. Exclui forks e uma denylist de infra (`LiamMartingo42.github.io`, repo de perfil). Implementação aditiva por progressive enhancement: contêiner `hidden` por padrão, revelado só quando o fetch tem sucesso. Reusa o helper `tempoRelativo` já existente. Sem achados críticos/altos.

## Achados

### 🔴 Críticos
- Nenhum.

### 🟠 Altos
- Nenhum.

### 🟡 Médios
- Nenhum.

### 🟢 Baixos
- Nomes de repo exibidos crus (ex: `neur.AI_BioSpaceExplorer_HACKATHON-NASA-2025`), enquanto a scoresheet usa "BioSpace Explorer". Inconsistência cosmética consciente — a faixa é "atividade no GitHub", então o nome real do repo é o correto. Sem ação.
- 2 dos 4 itens (BioSpace, CR-SpaceWalker) duplicam a scoresheet — trade-off aceito pelo usuário na escolha do escopo.

### 🔵 Informativo
- **Segurança**: nomes de repo via `textContent`; links com `rel="noopener"` + `target="_blank"` (previne `window.opener` hijacking). `html_url` vem da própria API do GitHub.
- **Degradação**: contêiner nasce `hidden`; só aparece em sucesso com ≥1 item. Falha de rede/403 ou lista vazia após filtro → faixa não aparece (sem heading órfão). Verificado nos 3 cenários.
- **Custo**: +1 request (`/users/{user}/repos?sort=pushed&per_page=100`) — 3 no total por carga de página. Rate limit 60/h por IP de visitante.
- **Manutenção**: para incluir/excluir repos, editar `DENYLIST` ou `MAX_ITENS` no script; novos repos públicos entram automaticamente por ordem de push.

## Resultado dos Testes

Chromium headless (Playwright 1.61.1) — `scratchpad/verify-activity.js` (mock) + teste real:

- Total executados: 11 cenários mockados + 1 teste real
- Passando: 12
- Falhando: 0
- Cobertura: n/a

Cenários mockados:
1. Repos OK → faixa visível. ✅
2. 4 itens (forks + infra filtrados). ✅
3. Sem forks (`gpt-engineer`, `hackathon2022` fora). ✅
4. Sem infra/perfil (denylist aplicada). ✅
5. Ordem por push (BioSpace 1º, Hackathon-Shift último). ✅
6. Tempo relativo pt-BR. ✅
7. Linguagem exibida quando presente. ✅
8. Links `target=_blank` + `rel=noopener`. ✅
9. 403 → faixa permanece `hidden`. ✅
10. 403 → nenhum item renderizado. ✅
11. Lista vazia após filtro (só forks) → `hidden`. ✅

Teste real (API pública, sem mock): 4 itens na ordem correta — BioSpace (há 9 meses · TypeScript), CR-SpaceWalker (há 2 anos · Python), challenge-django-postgresql (há 3 anos · Python), Hackathon-Shift (há 4 anos · PHP). Zero erros de console. Screenshot conferido.

## Decisões Técnicas

- **Contêiner `hidden` + reveal on success**: a faixa é inerentemente dinâmica (sem fallback estático possível); nascer oculta evita mostrar cabeçalho vazio quando não há dados ou a API falha.
- **Denylist por nome (lowercase)** + filtro `fork`: separa "projeto" de infra/forks sem depender de heurística frágil.
- **Reuso de `tempoRelativo`**: mesma formatação dos badges da scoresheet, uma fonte de verdade para tempo relativo.
- **1 fetch para a lista inteira** (`/users/{user}/repos`) em vez de 1 por repo.

## Pendências

- Nenhuma.
