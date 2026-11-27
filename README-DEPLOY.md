# Papato Gestão Pro — Cloudflare Pages

Este pacote mantém o painel original em HTML/CSS/JS, aplica a identidade visual da Papato e adiciona autenticação real no servidor com Cloudflare Pages Functions + D1.

## O que foi alterado

- Painel em `public/painel/index.html`.
- Logo oficial em `public/assets/img/logo.webp`.
- Tela de login em `public/admin-login.html`.
- Login validado no servidor; senha nunca fica no HTML ou no `localStorage`.
- Cookie `HttpOnly`, `Secure`, `SameSite=Strict`, token aleatório e sessão com validade de 8 horas.
- Logout invalida a sessão no D1.
- Middleware protege `/painel/*`, inclusive acesso direto e atualização da página.
- Limite de tentativas por IP (8 tentativas em 15 minutos).
- Fumaça/neblina suave no fundo, respeitando o botão de ligar/desligar já existente.

## Importante sobre os dados e a sincronização

O painel original guarda seus dados em `localStorage` no navegador. Isso significa que os registros atuais continuam locais a cada dispositivo/navegador. O botão "Sincronizar Catálogo Local" do HTML original apenas registra uma verificação local; ele **não** sincroniza de verdade com o site.

Esta entrega protege o acesso e melhora a interface, mas não migra automaticamente os dados para nuvem nem conecta pedidos/estoque do site. Para sincronização real entre site e painel, será necessário projetar uma API e tabelas D1 para produtos, estoque, vendas e clientes, além de definir qual sistema é a fonte oficial de cada dado.

## 1. Criar o banco D1

No terminal, na pasta do projeto:

```bash
npx wrangler login
npx wrangler d1 create papato-painel
```

Copie o `database_id` retornado e substitua `COLOQUE-AQUI-O-ID-DO-D1` no `wrangler.toml`.

Aplique o schema:

```bash
npx wrangler d1 execute papato-painel --remote --file=./schema.sql
```

## 2. Criar o primeiro usuário

Use Python 3:

```bash
python scripts/generate-admin-hash.py
```

Digite `papato` como usuário (ou outro nome), crie uma senha nova com pelo menos 12 caracteres e confirme. O script imprime um `INSERT` SQL com salt e hash PBKDF2. Cole esse SQL no terminal:

```bash
npx wrangler d1 execute papato-painel --remote --command="COLE_AQUI_O_INSERT_GERADO"
```

Não coloque a senha em HTML, JavaScript, GitHub, `wrangler.toml` ou variável pública.

## 3. Publicar no Cloudflare Pages

Configure o projeto Pages para usar o repositório GitHub que contém esta estrutura:
- Build command: deixe vazio (não precisa compilar)
- Build output directory: `public`
- Root directory: `/`

Mantenha a pasta `functions/` na raiz do repositório, no mesmo nível de `public/`. O Cloudflare Pages detecta e publica as Functions automaticamente.

Se o projeto Pages já existe, envie os arquivos deste pacote ao repositório ligado ao projeto. Não substitua nem apague o site público principal; este pacote serve para o projeto do painel.

## 4. Teste

- Abra `https://SEU-DOMINIO/admin-login.html`.
- Entre com o usuário e a senha que você acabou de criar.
- Deve abrir `https://SEU-DOMINIO/painel/`.
- Atualizar a página mantém a sessão até expirar.
- Clique em **Sair com segurança**; a sessão é revogada.
- Tente abrir `/painel/` novamente: deve pedir login.

## 5. Antes de usar com dados reais

- Faça backup do JSON atual do painel e guarde fora do repositório.
- Revise os dados de exemplo pré-carregados no HTML e substitua-os pelos dados reais.
- Restrinja o acesso ao repositório e ative autenticação multifator no GitHub e Cloudflare.
- Configure domínio e HTTPS no Cloudflare.
- Teste login, logout e restauração de backup antes de operar.

## Estrutura

```text
functions/
  _auth.js
  _middleware.js
  api/auth/
    login.js
    logout.js
    me.js
public/
  admin-login.html
  assets/img/logo.webp
  painel/index.html
schema.sql
scripts/generate-admin-hash.py
wrangler.toml
```
