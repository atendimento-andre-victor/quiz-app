# quiz-funil

Skill para [Claude Code](https://claude.com/claude-code) que constrói, do zero, um **quiz de funil de vendas completo** — das telas de perguntas até o dashboard de métricas.

```
Perguntas progressivas → email-gate → resultado personalizado → oferta com countdown
                              ↓
                    tracking de funil (Supabase)
                              ↓
                   dashboard de métricas com auth
```

## O que a skill entrega

| Peça | Descrição |
|---|---|
| `index.html` | State machine de telas, tracking, email-gate, countdown, prova social |
| `create_quiz_sessions.sql` | Tabela de sessões + GRANTs para captura do funil |
| `metricas.html` | Dashboard de KPIs, funil e tabela de leads — auth via Supabase Auth |
| `webhook.php` | Recebe a compra (Kiwify/Hotmart), gera senha com bcrypt, envia e-mail de acesso |
| `.htaccess` | Proteção do log do webhook e regras de cache |

## Como a skill trabalha

A skill segue 8 fases e **não escreve código antes do briefing estar completo**:

1. **Fase 0 — Briefing** · coleta produto, nicho, checkout, perguntas, banco, design, domínio → gera `quiz.config.md`
2. **Fase 1 — Arquitetura** · state machine linear de telas (`S0 → Sn`), state object, navegação, CSS base
3. **Fase 2 — Tracking** · SQL da tabela `quiz_sessions` + JS fire-and-forget com `keepalive` no `beforeunload`
4. **Fase 3 — Email-gate** · captura obrigatória **antes** da tela de resultado
5. **Fase 4 — Telas especiais** · loading animado de resultado e countdown da oferta
6. **Fase 5 — Dashboard** · métricas com autenticação via Supabase Auth
7. **Fase 6 — Webhook** · acesso protegido com senha em bcrypt
8. **Fase 7 — Deploy** · upload na Hostinger, conexão do webhook, teste end-to-end

## Decisões de arquitetura embutidas

- **Email-gate antes do resultado** — o resultado é o incentivo; coletar antes garante captura total
- **Insert fire-and-forget** — a UX não espera a latência do banco
- **`keepalive` no `beforeunload`** — grava `duration_seconds` mesmo se a aba fechar de repente
- **Supabase Auth no dashboard** — comparação de senha em JS é trivialmente burlável
- **bcrypt nas senhas dos alunos** — nunca texto puro
- **`service_role` key só no PHP** — no cliente, qualquer DevTools revela acesso admin ao banco
- **SQL versionado no repositório** — sem ele é impossível recriar o banco numa migração

## Instalação

### Opção 1 — Plugin (recomendado)

Dentro do Claude Code, dois comandos:

```
/plugin marketplace add atendimento-andre-victor/quiz
/plugin install quiz-funil@andre-victor
```

Ou pelo terminal:

```bash
claude plugin marketplace add atendimento-andre-victor/quiz
claude plugin install quiz-funil@andre-victor
```

A skill fica disponível como `/quiz-funil:quiz-funil` — e o Claude também a ativa sozinho quando você descreve a necessidade.

### Opção 2 — Cópia manual

```bash
git clone https://github.com/atendimento-andre-victor/quiz.git
cp -r quiz/skills/quiz-funil ~/.claude/skills/
```

Reinicie o Claude Code e confirme com `/quiz-funil`.

## Uso

```
/quiz-funil
```

Ou descreva a necessidade — a skill ativa sozinha:

> "Quero um quiz de diagnóstico que leva para a venda do meu programa de nutrição"

A skill começa pelo briefing e só avança quando tiver as respostas.

## Skills complementares

A `quiz-funil` cuida da **arquitetura técnica**. Nas fronteiras, ela sinaliza handoff:

| Território | Skill |
|---|---|
| Identidade visual (cores, tipografia) | `/frontend-design` |
| Perguntas e lógica de resultado | `/brainstorming` |
| Textos, CTAs, tela de resultado | `/copywriting` |
| Revisão final dos textos | `/humanizer` |
| Landing page de entrada | `/landing-page-design` + `/landing-page-copywriter` |

## Segurança

Este repositório contém **apenas placeholders** (`[SUPABASE_URL]`, `[SERVICE_ROLE_KEY]`, `[RESEND_KEY]`). Ao usar a skill num projeto real:

- Mantenha chaves e senhas fora do código-fonte — use variáveis de ambiente
- A `service_role` key nunca vai para o cliente, só para código server-side
- Adicione `.env`, `*.config.md` e `webhook-log.txt` ao `.gitignore` do projeto gerado

## Stack alvo

Supabase (Postgres + Auth) · PHP 8 · MySQL/Hostinger · Kiwify · Hotmart · Resend · HTML/CSS/JS puro

## Licença

MIT — veja [LICENSE](LICENSE).
