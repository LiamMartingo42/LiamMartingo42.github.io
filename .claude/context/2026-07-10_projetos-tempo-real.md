# Contexto — Projetos com dados em tempo real do GitHub

**Data**: 2026-07-10
**Status**: ✅ Concluída

---

## Descrição da Tarefa

Usuário pediu que os repositórios da section "projects" fossem mostrados em tempo real, refletindo as últimas alterações, via JavaScript no HTML.

## Conflito descoberto e decisão

Dos 6 projetos da scoresheet, só 2 são repos públicos no GitHub (`CR-SpaceWalker`, `neur.AI_BioSpaceExplorer_HACKATHON-NASA-2025`). Os outros 4 são "private build" — a API pública não retorna. Um fetch puro "últimos alterados" traria infra (`.github.io`, repo de perfil) e forks antigos, destruindo a curadoria.

Brainstorming → usuário escolheu **enriquecer os curados**: manter os 6 movimentos, adicionar dados ao vivo só nos 2 públicos. Sinais escolhidos: última alteração (tempo relativo) + mensagem do último commit.

## O que foi implementado

Progressive enhancement em `index.html`:

1. `data-repo="owner/name"` nos 2 `<li class="move">` públicos.
2. `<script>` vanilla antes de `</body>`: para cada `.move[data-repo]`, `fetch` em `https://api.github.com/repos/{repo}/commits?per_page=1`; do commit `[0]` extrai `committer.date` → `Intl.RelativeTimeFormat('pt-BR')` ("atualizado há X") e `message` → primeira linha truncada em 60 chars; injeta `<p class="live-meta">` (ponto pulsante + tempo + mensagem) após o `.move-head`.
3. Segurança: DOM montado com `createElement`/`textContent` — nunca `innerHTML`. Mensagem de commit (atacável) tratada como texto.
4. Degradação: `try/catch` por repo + `response.ok`; em erro, cartão fica estático.
5. CSS `.live-meta` + `.live-dot` (ponto verde-água pulsante, `@keyframes live-blink`).

## Arquivos Modificados

| Arquivo      | Tipo de Mudança |
| ------------ | --------------- |
| `index.html` | Modificado (2× data-repo, bloco CSS `.live-meta`, `<script>` no fim do body) |

## Dependências Adicionadas

| Pacote    | Versão | Justificativa |
| --------- | ------ | ------------- |
| (nenhuma) | —      | — (fetch nativo, Intl nativo) |

## Padrões Seguidos

- Progressive enhancement: HTML curado é a fonte da verdade; página funciona sem JS.
- Sem bibliotecas, sem build step (mantém projeto como HTML estático puro).
- `data-repo` como opt-in explícito por cartão.
- Verificação por Chromium headless (mock + teste real), padrão já adotado no projeto.

## Impacto

- **Breaking changes**: Não (HTML estático permanece; badge é aditivo)
- **Requer migration**: Não
- **Requer variável de ambiente nova**: Não
- **Requer rebuild/deploy**: Sim (republicar `index.html`)

## Aprendizados para Próximas Tarefas

- API pública do GitHub: `/repos/{owner}/{repo}/commits?per_page=1` entrega data + mensagem do último commit numa só chamada — preferir a duas chamadas separadas.
- Rate limit da API do GitHub sem auth: 60 req/h **por IP**; como o fetch é client-side, o limite é por visitante, não global — viável para portfólio sem token.
- Playwright `page.route('**://api.github.com/**', ...)` para mockar sucesso/403 e testar degradação sem depender da rede.
- Para adicionar um novo projeto ao "ao vivo": basta pôr `data-repo="owner/nome"` no `<li>` — nenhum outro código muda.
- Repos públicos atuais de LiamMartingo42 (por push): `.github.io`, `LiamMartingo42` (perfil), `neur.AI_BioSpaceExplorer...` (TS), `CR-SpaceWalker` (Python), `challenge-django-postgresql`, forks (`gpt-engineer`, `hackathon2022`), `Hackathon-Shift` (PHP).

## Link com Code Review

→ `.claude/code-review/2026-07-10_projetos-tempo-real.md`
