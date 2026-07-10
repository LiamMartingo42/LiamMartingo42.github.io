# Contexto — Fix: átomo não animava no ambiente do usuário

**Data**: 2026-07-09
**Status**: ✅ Concluída

---

## Descrição da Tarefa

Após a primeira implementação das animações do átomo, usuário reportou: "Ainda não ocorre as animações do atomo, pode deixar ele numa div separada e fazer apenas com o css esse tipo de animação do atomo com os eletrons se movendo."

## O que foi implementado

**Causa raiz** (debugging sistemático, hipótese testada em headless antes do fix): o código anterior desligava TODAS as animações do átomo sob `prefers-reduced-motion: reduce` e escondia o átomo (`display: none`) em viewport ≤ 44rem. São as duas únicas condições que produzem o sintoma "nada se move" — as animações em si rodavam normalmente em Chromium.

**Fix — reimplementação do átomo:**

1. HTML: estrutura de div separada, elétrons como elementos reais:
   `.atom > .nucleus + .orbits > .ring.ring-N > .orbit-spin > .electron`
2. CSS: tilt estático no pai (`.ring-N { rotate(±Ndeg) scaleY(0.42) }`), rotação animada no filho (`.orbit-spin { animation: orbit-spin 14/21/29s linear infinite }`), keyframes triviais sem `var()`: `@keyframes orbit-spin { to { transform: rotate(360deg) } }`. Composição pai·filho = mesma matriz do transform composto antigo — visual idêntico.
3. Átomo removido do bloco `prefers-reduced-motion` — anima sempre (decisão do usuário; registrada como 🟡 no code review).
4. Media query 44rem: átomo vira fundo sutil (15rem, `opacity: 0.3`, right: -5rem) em vez de `display: none`.
5. CSS antigo removido: `.orbit`, `.orbit-1/2/3`, `--tilt`, `@keyframes spin`.

Mantidas da tarefa anterior: `nucleus-pulse` (4s), `atom-float` (9s), `precess` (60s em `.orbits`).

## Arquivos Modificados

| Arquivo      | Tipo de Mudança |
| ------------ | --------------- |
| `index.html` | Modificado (markup do átomo + bloco CSS do átomo + media queries) |

## Dependências Adicionadas

| Pacote    | Versão | Justificativa |
| --------- | ------ | ------------- |
| (nenhuma) | —      | — |

## Padrões Seguidos

- TDD: cenários que reproduzem o ambiente do usuário escritos primeiro (falharam 4/4 contra o código antigo), fix depois, 7/7 passando.
- CSS inline, keyframes kebab-case, sem JavaScript.

## Impacto

- **Breaking changes**: Não (visual idêntico em desktop; mobile agora mostra átomo sutil onde antes não havia nada)
- **Requer migration**: Não
- **Requer variável de ambiente nova**: Não
- **Requer rebuild/deploy**: Sim (republicar `index.html`; usuário deve dar hard refresh — Ctrl+F5)

## Aprendizados para Próximas Tarefas

- Quando usuário reporta "animação não roda" e o CSS é válido: procurar primeiro o que DESLIGA animações (`prefers-reduced-motion`, `display: none` em media queries, `animation-play-state`) antes de suspeitar da sintaxe.
- Padrão robusto para órbita elíptica em CSS puro: tilt+squash estáticos no pai, rotação animada no filho — evita `var()` em keyframes e transform composto animado.
- `.hero` tem `overflow: hidden` — elementos decorativos com offset negativo não geram scroll horizontal.

## Link com Code Review

→ `.claude/code-review/2026-07-09_fix-animacao-atomo.md`
