# Contexto — Faixa "atividade recente no GitHub"

**Data**: 2026-07-10
**Status**: ✅ Concluída

---

## Descrição da Tarefa

Após a feature de badges nos projetos curados, o usuário notou que os badges mostravam datas antigas ("há 9 meses", "há 2 anos"). Investigação confirmou que não era bug: são as últimas alterações reais dos 2 repos públicos curados, que genuinamente não mudam desde 2024/2025. O usuário então pediu uma faixa "atividade recente no GitHub" à parte.

## Decisão de escopo

Filtros escolhidos: **só código (sem forks/infra)** — exclui forks (`gpt-engineer`, `hackathon2022`) e denylist de infra (`LiamMartingo42.github.io`, `LiamMartingo42` perfil). Resultado: BioSpace, CR-SpaceWalker, `challenge-django-postgresql`, `Hackathon-Shift`. Usuário aceitou que BioSpace/CR-SpaceWalker dupliquem a scoresheet.

## O que foi implementado

Progressive enhancement aditivo em `index.html`, dentro de `<section id="projects">` após o `</ol>`:

1. Contêiner `<div class="activity" data-activity-user="LiamMartingo42" hidden>` com `<p class="section-label">` + `<ul class="activity-list">` vazio.
2. JS (mesmo IIFE): 1 fetch em `api.github.com/users/{user}/repos?sort=pushed&per_page=100` → filtra `fork===true` e `DENYLIST` → `slice(0, MAX_ITENS=5)` → renderiza `<li class="activity-item">` (ponto pulsante + nome linkado `html_url` com `target=_blank rel=noopener` + "atualizado há X" via `tempoRelativo` + linguagem) → `bloco.hidden = false`.
3. Falha (403/rede/lista vazia após filtro) → `catch` deixa o bloco `hidden`.
4. CSS `.activity`, `.activity-list`, `.activity-item`, `.act-when`, `.act-lang` — reusa `.live-dot` pulsante.

## Arquivos Modificados

| Arquivo      | Tipo de Mudança |
| ------------ | --------------- |
| `index.html` | Modificado (div `.activity` no HTML, bloco CSS `.activity*`, lógica no `<script>`) |

## Dependências Adicionadas

| Pacote    | Versão | Justificativa |
| --------- | ------ | ------------- |
| (nenhuma) | —      | — |

## Padrões Seguidos

- Progressive enhancement; faixa some se JS/API falhar (sem cabeçalho órfão).
- Reuso do helper `tempoRelativo` (uma fonte de verdade para tempo relativo).
- `textContent` para dados da API; links externos com `rel="noopener"`.
- Verificação por Chromium headless (mock + real).

## Impacto

- **Breaking changes**: Não
- **Requer migration**: Não
- **Requer variável de ambiente nova**: Não
- **Requer rebuild/deploy**: Sim (republicar `index.html`)

## Aprendizados para Próximas Tarefas

- Endpoint `/users/{user}/repos?sort=pushed` traz `fork`, `pushed_at`, `language`, `html_url`, `name`, `description` numa chamada — suficiente para a faixa inteira.
- Para blocos inerentemente dinâmicos (sem fallback estático), nascer `hidden` e revelar só em sucesso é o padrão limpo — evita UI quebrada quando a API falha.
- Ajustar a faixa: `DENYLIST` (nomes lowercase) e `MAX_ITENS` no script; novos repos entram sozinhos por ordem de push.
- A atividade pública "mais recente" da conta é infra (`.github.io` = o próprio site, 2026) e perfil — por isso foram excluídos por denylist; os projetos de vitrine de verdade são de 2024–2025.

## Link com Code Review

→ `.claude/code-review/2026-07-10_faixa-atividade-recente.md`
