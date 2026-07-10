# Code Review — Animação fluida do átomo do hero

**Data**: 2026-07-09
**Arquivos revisados**: `index.html`
**Autor da implementação**: Claude Code (assistido)

---

## Resumo

Foram adicionadas três animações CSS ao átomo decorativo do hero: pulso do núcleo (`nucleus-pulse`, 4s), flutuação vertical do átomo inteiro (`atom-float`, 9s) e precessão lenta do plano orbital (`precess`, 60s, via wrapper novo `.orbits`). As animações `spin` existentes dos elétrons foram mantidas intactas. O código segue os padrões do arquivo (CSS inline, keyframes nomeados, bloco reduced-motion). Qualidade geral boa; nenhum achado crítico ou alto.

## Achados

### 🔴 Críticos
- Nenhum.

### 🟠 Altos
- Nenhum.

### 🟡 Médios
- Nenhum.

### 🟢 Baixos
- `nucleus-pulse` anima `box-shadow`, que não é composited (gera repaint por frame). Para um elemento de 14px o custo é desprezível; alternativa (pseudo-elemento com `opacity`) não justifica a complexidade extra. Registrado, sem ação.

### 🔵 Informativo
- Animações infinitas rodam enquanto a aba está visível, mesmo com o hero fora do viewport ao rolar. Browsers modernos fazem throttle do compositor; impacto irrelevante para esta página.
- O wrapper `.orbits` (novo nó no DOM) foi a alternativa escolhida em vez de `@property --tilt` animado, para compatibilidade universal sem risco de "salto discreto" em browsers antigos.
- Investigação do relato "nada se move": o CSS `spin` original é válido e roda em Chromium headless. Causas prováveis no ambiente do usuário: (1) `prefers-reduced-motion: reduce` ativo no SO, (2) janela ≤ 44rem, onde o átomo tem `display: none`.

## Resultado dos Testes

Projeto sem suíte de testes (HTML estático puro) — verificação end-to-end via Chromium headless (Playwright 1.61.1):

- Total executados: 4 cenários
- Passando: 4
- Falhando: 0
- Cobertura: n/a

Cenários verificados:
1. Motion normal: `document.getAnimations()` → 6 animações `running` (`atom-float`, `nucleus-pulse`, `precess`, 3× `spin`).
2. Movimento real: screenshots da região do átomo em t0 e t0+2s diferem (pixels mudaram); confirmado visualmente (órbitas precessando, núcleo pulsando, átomo deslocado).
3. `prefers-reduced-motion: reduce`: 0 animações; átomo renderiza estático e visível.
4. Viewport 600px: átomo escondido (comportamento responsivo preservado).

## Decisões Técnicas

- **Precessão via wrapper `.orbits` girando**, em vez de animar `--tilt` com `@property`: evita dependência de registro de custom property (que degrada para interpolação discreta em browsers sem suporte) e mantém compatibilidade total.
- **`atom-float` anima `transform` com `calc(-50% ± 8px)`**: preserva a centralização vertical original (`translateY(-50%)`) como estado base, que continua valendo quando `animation: none` (reduced-motion).
- **Bloco `prefers-reduced-motion` estendido** com `.atom`, `.orbits`, `.nucleus` — todas as animações novas respeitam a preferência do usuário.

## Pendências

- Nenhuma.
