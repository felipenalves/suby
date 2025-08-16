# suby — Deploy (GitHub + Vercel)

Este guia resume o mínimo para colocar o app no ar.

## 1) Pré-requisitos locais
- Node 20+, pnpm
- Docker (para Postgres local com `docker compose`)

## 2) Banco de dados local (opcional)
```bash
docker compose up -d
cp .env.example .env
# Ajuste NEXTAUTH_SECRET com uma chave segura
pnpm prisma:gen
pnpm prisma:migrate
pnpm seed
pnpm dev
```

## 3) Repositório no GitHub
```bash
git init
git add .
git commit -m "init suby"
git branch -M main
git remote add origin https://github.com/<seu-usuario>/suby.git
git push -u origin main
```

## 4) Banco de dados em Produção (Neon ou Supabase)
- Crie um Postgres gerenciado (Neon/Supabase).
- Copie a URL e defina como `DATABASE_URL` **tanto local quanto na Vercel**.
- Execute localmente:
```bash
# Usando a DATABASE_URL remota
pnpm prisma:gen
pnpm prisma:migrate
pnpm seed
```
Isso garante que o banco remoto esteja migrado/seedado antes do deploy.

## 5) Deploy na Vercel
- Vá em **Import Project** → GitHub → escolha o repositório `suby`.
- Em **Environment Variables**, adicione:
  - `DATABASE_URL` (a do Neon/Supabase)
  - `NEXTAUTH_URL` (ex.: https://suby.vercel.app)
  - `NEXTAUTH_SECRET` (uma chave segura)
  - `FX_PROVIDER_URL` (opcional; default: https://api.exchangerate.host)
  - `RESEND_API_KEY` (se for enviar e-mails)
  - `STORAGE_PATH` (pode deixar `./storage`)
- Build Command padrão do Next.js. Se preferir, assegure migrations no build:
  - Scripts `package.json`:
    ```json
    {
      "scripts": {
        "prisma:deploy": "prisma migrate deploy",
        "build": "pnpm prisma:deploy && next build"
      }
    }
    ```

## 6) Observações
- GitHub hospeda **código**; a execução do app fica na Vercel (recomendado para Next.js).
- Alternativas: Railway, Render, Fly.io (mude apenas a forma de setar env vars e o banco).
- Para atualizações de schema em produção:
```bash
# altere o Prisma, depois:
pnpm prisma migrate dev --name <mudanca>      # local
pnpm prisma migrate deploy                    # produção (ou no build)
```
