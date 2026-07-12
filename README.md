# Ferlin Estética e Saúde — Site

Site institucional em página única (`index.html`), sem build, sem dependências de servidor. HTML + CSS + JS puro, com [GSAP](https://gsap.com/) e ScrollTrigger carregados via CDN para as animações de scroll.

## Rodar localmente

Não precisa de instalação. Basta abrir o arquivo:

```bash
open index.html      # macOS
# ou dois cliques no arquivo
```

Ou, se preferir um servidor local simples:

```bash
npx serve .
```

## Publicar no Vercel

**Opção A — pelo GitHub (recomendado):**
1. Suba este repositório para o GitHub (veja abaixo).
2. Acesse [vercel.com/new](https://vercel.com/new) e faça login.
3. Clique em **Import Project**, selecione este repositório.
4. Framework: **Other** (site estático). Não precisa configurar build command nem output directory.
5. Clique em **Deploy**. Pronto — em ~30 segundos o site está no ar.

**Opção B — deploy direto, sem GitHub:**
1. Instale a CLI: `npm i -g vercel`
2. Dentro da pasta do projeto: `vercel`
3. Siga as instruções no terminal.

## Subir este repositório para o GitHub

```bash
# dentro da pasta do projeto (já inicializada com git)
git remote add origin https://github.com/SEU-USUARIO/ferlin-site.git
git push -u origin main
```

Se ainda não criou o repositório no GitHub: acesse [github.com/new](https://github.com/new), dê o nome `ferlin-site` (ou outro), **não** marque "Add a README" (já existe um aqui), crie, e use a URL gerada no comando acima.

## Pendências antes de publicar

O site já está pronto visualmente, mas alguns dados são placeholders — procure por `[inserir ...]` dentro do `index.html` (seção de contato) e substitua por:

- Endereço completo
- Telefone / WhatsApp
- Horário de funcionamento completo (hoje só consta "segunda a partir das 09:00")

Os dois depoimentos na seção "Depoimentos" também são ilustrativos — vale trocar por avaliações reais quando tiver.

## Stack

- HTML5 + CSS3 (variáveis nativas, grid, clamp) + JavaScript vanilla
- [GSAP 3](https://gsap.com/) + ScrollTrigger (CDN, com fallback para `IntersectionObserver` caso o CDN não carregue)
- Google Fonts: Fraunces, Cormorant Garamond, Manrope
- Fotos da própria clínica, embutidas em base64 no HTML (sem pasta de assets externa)
