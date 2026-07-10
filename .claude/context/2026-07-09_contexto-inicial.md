# Contexto Inicial do Projeto

## Stack
- Linguagem: HTML + CSS (inline no `index.html`), sem JavaScript
- Framework: nenhum (página estática pura)
- Banco de dados: nenhum
- Gerenciador de pacotes: nenhum (sem `package.json`)
- Runtime / Versão: browser (arquivo estático)

## Arquitetura
- Tipo: monolito (página única)
- Estrutura de pastas principal:
  - `index.html` — página inteira (markup + CSS inline em `<style>`)
  - `assets/` — imagens
  - `README.md`

## Padrões Identificados
- Estilo de código: CSS com custom properties nomeadas em pt (`--cobre`, `--poeira`, `--estrela`, `--orbital`, `--linha`); comentários de seção `/* ---------- nome ---------- */`
- Linter / Formatter: nenhum configurado
- Padrão de nomeação: classes em inglês kebab-case (`.section-label`, `.about-grid`); tema visual de xadrez/espaço/átomos
- Padrão de imports: n/a (tudo inline)

## Testes
- Framework de teste: nenhum
- Diretório de testes: nenhum
- Comando para executar testes: n/a — verificação é feita via browser headless (Playwright/Chromium)
- Cobertura atual: n/a

## Observações
- Branch de trabalho: `portfolio-redesign` (redesign do portfólio em torno de "chess scoresheet")
- Página tem media query responsiva em `44rem` que **esconde o átomo decorativo** (`display: none`)
- Página respeita `prefers-reduced-motion: reduce` — desliga todas as animações
- Único momento animado da página é o átomo do hero (comentário no CSS: "atom — the one animated moment")
