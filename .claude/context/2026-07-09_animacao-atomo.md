# Contexto — Animação fluida do átomo do hero

**Data**: 2026-07-09
**Status**: ✅ Concluída

---

## Descrição da Tarefa

Fazer o átomo decorativo do hero "se mexer" com animação CSS mais fluida. Usuário pediu (via seleção múltipla): núcleo pulsando, átomo flutuando, precessão das órbitas, e investigar por que os elétrons não se moviam na tela dele.

## O que foi implementado

Três animações CSS novas no bloco "atom" do `index.html`, mantendo o `spin` existente dos elétrons:

1. `nucleus-pulse` (4s ease-in-out infinite) — núcleo respira: `scale(1 → 1.25)` + `box-shadow` mais intenso no pico.
2. `atom-float` (9s ease-in-out infinite) — átomo inteiro flutua verticalmente com `translateY(calc(-50% ± 8px))`.
3. `precess` (60s linear infinite) — wrapper novo `.orbits` (envolvendo as 3 órbitas no HTML) gira 360°, fazendo o plano orbital inteiro precessar.

Bloco `prefers-reduced-motion` estendido para desligar as três animações novas.

**Diagnóstico do "nada se move"**: o CSS original era válido e roda normalmente em Chromium headless. Causas prováveis no ambiente do usuário: `prefers-reduced-motion` ativo no SO (Windows: "Mostrar animações" desligado) ou janela ≤ 44rem (átomo tem `display: none`).

## Arquivos Modificados

| Arquivo      | Tipo de Mudança |
| ------------ | --------------- |
| `index.html` | Modificado (CSS do átomo + wrapper `.orbits` no HTML + bloco reduced-motion) |

## Dependências Adicionadas

| Pacote    | Versão | Justificativa |
| --------- | ------ | ------------- |
| (nenhuma) | —      | — (playwright usado só como ferramenta de verificação, fora do projeto) |

## Padrões Seguidos

- CSS inline no `<style>` do `index.html`, keyframes nomeados em kebab-case, comentários de seção existentes preservados.
- Todas as animações respeitam `prefers-reduced-motion` (padrão já existente na página).
- Nenhum JavaScript introduzido — página continua estática pura.

## Impacto

- **Breaking changes**: Não
- **Requer migration**: Não
- **Requer variável de ambiente nova**: Não
- **Requer rebuild/deploy**: Sim (republicar o `index.html`)

## Aprendizados para Próximas Tarefas

- Playwright element screenshot falha com "element is not stable" em elementos com animação de `transform` contínua — usar `page.screenshot({ clip })` com região fixa.
- O MCP do Playwright neste ambiente exige distribution `chrome` (`/opt/google/chrome/chrome`, precisa sudo); alternativa que funciona sem sudo: `npx playwright install chromium` + script node com a lib `playwright` no scratchpad.
- A media query de `44rem` esconde o átomo por completo — qualquer trabalho futuro no átomo deve ser testado em viewport largo.

## Link com Code Review

→ `.claude/code-review/2026-07-09_animacao-atomo.md`
