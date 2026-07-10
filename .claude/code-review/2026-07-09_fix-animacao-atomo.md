# Code Review — Fix: átomo não animava no ambiente do usuário

**Data**: 2026-07-09
**Arquivos revisados**: `index.html`
**Autor da implementação**: Claude Code (assistido)

---

## Resumo

Segunda iteração da animação do átomo. Usuário reportou que nada se movia mesmo após a primeira implementação. Debugging sistemático provou (via Chromium headless) que o código anterior zerava todas as animações sob `prefers-reduced-motion: reduce` e escondia o átomo em viewport ≤ 44rem — as duas únicas condições capazes de produzir o sintoma relatado. O átomo foi reimplementado em estrutura de div separada com elétrons como elementos reais, keyframes sem `var()`, animação independente de `prefers-reduced-motion` e visível (sutil) em mobile. Suite de verificação: 7/7 cenários passando, incluindo os dois que reproduzem o ambiente do usuário (falhavam antes do fix).

## Achados

### 🔴 Críticos
- Nenhum.

### 🟠 Altos
- Nenhum.

### 🟡 Médios
- **Átomo agora ignora `prefers-reduced-motion`** — regressão consciente de acessibilidade (WCAG 2.3.3, nível AAA). Justificativa de pendência: pedido explícito e repetido do usuário ("fazer o átomo se mexer"), elemento puramente decorativo (`aria-hidden`), movimento lento (14–60s por ciclo) e em área periférica. Se houver feedback de usuários sensíveis a movimento, reintroduzir o bloco reduced-motion para o átomo.

### 🟢 Baixos
- Elétron continua levemente achatado pelo `scaleY(0.42)` do anel (comportamento idêntico ao design original com `::after`). Contra-escala estática produziria deformação cíclica durante a rotação; correção perfeita exigiria counter-rotation sincronizada — complexidade não justificada para um dot de 8px com glow.

### 🔵 Informativo
- Equivalência matemática preservada: antes `rotate(tilt) scaleY(0.42) rotate(θ)` num único elemento; agora pai `.ring` com `rotate(tilt) scaleY(0.42)` e filho `.orbit-spin` com `rotate(θ)` — a composição pai·filho gera a mesma matriz. Visual idêntico ao design aprovado.
- CSS antigo removido por completo (`.orbit`, `.orbit-1/2/3`, `--tilt`, `@keyframes spin`) — sem código morto.
- Em mobile o átomo fica com `opacity: 0.3` atrás do texto do hero (`overflow: hidden` do `.hero` evita scroll horizontal — verificado: 0px de overflow em 600px).

## Resultado dos Testes

Verificação end-to-end via Chromium headless (Playwright 1.61.1) — `scratchpad/verify-atom.js`:

- Total executados: 7 cenários
- Passando: 7
- Falhando: 0
- Cobertura: n/a

Cenários:
1. Motion normal: 6 animações `running` (`atom-float`, `nucleus-pulse`, `precess`, 3× `orbit-spin`). ✅
2. Motion normal: pixels da região do átomo mudam entre t0 e t0+2s. ✅
3. `prefers-reduced-motion: reduce`: átomo continua com 6 animações (**falhava antes: 0**). ✅
4. `prefers-reduced-motion: reduce`: pixels mudam (**falhava antes**). ✅
5. Viewport 600px: átomo visível (**falhava antes: `display: none`**). ✅
6. Viewport 600px: 6 animações rodando (**falhava antes: 0**). ✅
7. Viewport 600px: sem overflow horizontal. ✅

## Decisões Técnicas

- **TDD aplicado**: cenários 3–6 foram escritos primeiro reproduzindo o ambiente hipotético do usuário e falharam contra o código anterior, confirmando a causa raiz antes de qualquer fix.
- **Elétrons como elementos reais** (`.electron` dentro de `.orbit-spin` dentro de `.ring`) em vez de `::after` com transform composto animado: keyframes ficam triviais (`to { rotate(360deg) }`), sem `var()` — elimina qualquer dependência de custom properties em keyframes.
- **Mobile mostra átomo sutil** (15rem, `opacity: 0.3`, parcialmente fora da tela) em vez de `display: none`: garante que o efeito exista em qualquer largura de janela.

## Pendências

- 🟡 Reintroduzir respeito a `prefers-reduced-motion` no átomo caso o usuário mude de ideia (uma media query de ~6 linhas).
