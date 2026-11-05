# ADR-022 — Deploy: Vercel (web) + VPS própria (API e worker)

> **Atualização:** o *mecanismo* de entrega foi alterado por **ADR-024** — todo deploy é disparado pelo GitHub Actions. A topologia abaixo permanece.

- **Status:** Aceito (topologia definida pelo Product Owner)
- **Data:** 2026-11-05
- **Decisor:** Product Owner
- **Relacionado:** ADR-021, ADR-004, ADR-018

## Contexto

O frontend deve rodar na Vercel; a API e os workers devem rodar em VPS própria. MongoDB Atlas e object storage permanecem gerenciados externamente (ADR-003/018). É preciso definir rede, TLS, processos, deploy e recuperação.

## Decisão

### Frontend (Vercel)

- Projeto `projeta-web`, deploy automático por push na branch principal.
- Domínio de produção `app.<dominio>` com TLS gerenciado pela Vercel.
- Variáveis: `NEXT_PUBLIC_API_URL`, `NEXT_PUBLIC_ENV`, `NEXT_PUBLIC_USE_MOCKS`.
- Preview deployments por PR; **sem dados reais** (ver ADR-023).

### Backend (VPS)

- Docker Engine + Docker Compose em VPS Linux.
- Serviços: `api` (porta interna 3001), `worker`, `redis`, `caddy` (TLS automático).
- MongoDB Atlas e S3 são externos; portas de banco nunca expostas.
- `api.<dominio>` e `staging.api.<dominio>` com TLS via Caddy.
- API e worker usam a mesma imagem, comandos diferentes.
- Migrações executadas como job único antes do rollout.
- Healthcheck obrigatório antes de trocar tráfego.

### CI/CD

| Repositório | Pipeline |
|:--|:--|
| `projeta-web` | GitHub Actions → Vercel CLI (staging em `main`, produção em tag com aprovação) |
| `projeta-api` | GitHub Actions: test → build imagem → GHCR → deploy no VPS via SSH → healthcheck → rollback |

- Repositórios no GitHub sob o owner `andrelobo` (ADR-021).
- Usuário de deploy na VPS: **`lobojoe`**, com acesso ao Docker e ao diretório `/home/lobojoe/projeta`; login root desabilitado.
- Deploy da Vercel via CLI dentro do Actions; auto-deploy da integração Git **desabilitado** (ADR-024).
- Imagens da API publicadas no GHCR com tag semântica e digest.

- Deploy da API só a partir de tag; rollback para tag anterior.
- Segredos em GitHub Environments e `.env` no VPS com permissão 600.
- UFW liberando apenas 22/80/443; fail2ban; SSH por chave.
- Backup: Atlas (gerenciado) + versionamento do bucket; Redis é efêmero e jobs são reprocessáveis a partir do estado persistido.

## Consequências

- Latência Vercel ↔ VPS depende da região; escolher região próxima (ex.: `gru1`/`iad1` e VPS no Brasil/EUA).
- VPS é ponto único de falha para API/worker; exige monitoramento e runbook.
- Duas superfícies de deploy e dois rollbacks.
- Custo previsível de VPS + Vercel, sem orquestrador.

## Alternativas consideradas

- Backend serverless (Vercel Functions): inadequado para workers, filas e processos longos.
- Kubernetes: complexidade desproporcional ao estágio.
- VPS única para tudo (incluindo frontend): perde CDN e previews da Vercel.
