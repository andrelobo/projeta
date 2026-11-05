# ADR-014 — Autenticação por cookie httpOnly + refresh rotativo + CSRF

- **Status:** Proposto — requer aprovação
- **Data:** 2026-11-05
- **Decisores:** System Architect
- **Relacionado:** RF-002, §11

## Contexto

O frontend é Next.js com SSR. Guardar token em `localStorage` expõe a XSS e dificulta SSR autenticado.

## Decisão

- Access token JWT curto (≈15 min) em cookie `httpOnly`, `Secure`, `SameSite=Lax`.
- Refresh token rotativo por família, armazenado por hash, revogável.
- CSRF token double-submit para métodos mutantes.
- Logout revoga a família; reutilização de refresh revoga toda a família.

## Justificativa

- Cookies `httpOnly` não são acessíveis a JavaScript, reduzindo impacto de XSS.
- Com web na Vercel e API na VPS, o cookie é escopado ao domínio pai (`Domain=.<dominio>`), conforme ADR-023.
- Rotação detecta roubo de refresh.
- `SameSite=Lax` + CSRF cobre navegação cross-site em mutações.

## Consequências

- API precisa de CORS restrito com `credentials`.
- Testes E2E e de segurança cobrem CSRF, fixação de sessão e revogação.
- Requisições server-side do Next repassam cookies.

## Ver também

ADR-023 (domínios e previews).

## Alternativas

- Bearer em memória: simples, mas quebra refresh de página e SSR.
- Sessão opaca em Redis: viável; JWT curto reduz consulta por request.
