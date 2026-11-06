# GitHub Packages — `@projeta/contracts`

**Decisão:** OPEN-053 do plano de 2026-11-06.
**Objetivo:** permitir que `projeta-web` instale `@projeta/contracts` no build da Vercel e no CI, sem expor token no repositório.

---

## 1. O que já está no repositório

- `projeta-web/.npmrc` e `projeta-api/.npmrc`:

  ```
  @projeta:registry=https://npm.pkg.github.com
  ```

- Apenas o **escopo do registry** é versionado. O token nunca entra no repositório.

> **Importante (pnpm 11):** o pnpm ignora credenciais com variável de ambiente em `.npmrc` de projeto, para impedir que um `.npmrc` versionado aponte para um registry malicioso. Por isso a linha `_authToken=${NODE_AUTH_TOKEN}` **não** deve ficar no repositório. A credencial precisa ser gravada em `~/.npmrc` no passo de instalação (Vercel ou CI).

## 2. Token necessário

Criar um token de leitura no GitHub, com o menor escopo possível:

- **Classic PAT:** escopo `read:packages` (e `repo`, se o pacote estiver em repositório privado).
- **Fine-grained PAT:** permissão **Packages: Read** no repositório `andrelobo/projeta-api`.

> GitHub Packages exige autenticação para npm mesmo quando o pacote é público. Sem o token em `~/.npmrc`, o `pnpm install` falha com 401/404.

A criação do token é uma ação de conta: o GitHub não permite criar PAT por API. O responsável pela conta `andrelobo` deve criá-lo em **Settings → Developer settings → Personal access tokens**.

## 3. Onde configurar

### Vercel

1. Adicionar `NODE_AUTH_TOKEN` em **Project → Settings → Environment Variables**, nos três ambientes (Production, Preview e Development). Nunca usar `NEXT_PUBLIC_*` para esse valor.
2. Ajustar o **Install Command** do projeto para gravar a credencial em `~/.npmrc` antes do install:

   ```bash
   echo "//npm.pkg.github.com/:_authToken=$NODE_AUTH_TOKEN" >> "$HOME/.npmrc" && pnpm install --frozen-lockfile
   ```

   O `Build Command` continua sendo `pnpm build`.

### GitHub Actions

Adicionar o secret no repositório `andrelobo/projeta-web`:

```bash
gh secret set NODE_AUTH_TOKEN --repo andrelobo/projeta-web
```

O workflow deve gravar a credencial em `~/.npmrc` no passo de install:

```yaml
- name: Configure GitHub Packages
  run: echo "//npm.pkg.github.com/:_authToken=$NODE_AUTH_TOKEN" >> "$HOME/.npmrc"
  env:
    NODE_AUTH_TOKEN: ${{ secrets.NODE_AUTH_TOKEN }}
```

Para o `projeta-api` publicar o pacote, o workflow de release usa o `GITHUB_TOKEN` nativo com `packages: write`; não reutilizar o token de leitura da Vercel.

## 4. Verificação local

```bash
echo "//npm.pkg.github.com/:_authToken=$NODE_AUTH_TOKEN" >> "$HOME/.npmrc"
pnpm install
pnpm why @projeta/contracts
```

Sem o token, o install deve falhar de forma explícita. Isso é esperado e protege contra dependência não autenticada.

## 5. Segurança

- Token somente leitura (`read:packages`), sem `write:packages`.
- Rotação periódica e revogação imediata em caso de suspeita.
- Nunca commitar `.npmrc` com token literal nem `_authToken=${...}` no repositório.
- Não imprimir o token em log de CI.
- Previews de PR de fork não recebem secrets; o build deve usar mock ou falhar de forma controlada.
- O token da Vercel é diferente do `GHCR_READ_TOKEN` usado pela VPS para puxar a imagem privada.

## 6. Pendência operacional

A configuração de escopo está pronta. Faltam duas ações de conta:

1. **criar o PAT** com `read:packages`;
2. **cadastrá-lo** na Vercel (`NODE_AUTH_TOKEN`) e no GitHub Actions (`gh secret set`).

Enquanto isso, builds que dependam de `@projeta/contracts` devem ser considerados bloqueados por credencial, não por código. Nenhuma dependência atual usa o escopo `@projeta`, então M0.1 e M0.2 não são afetados.
