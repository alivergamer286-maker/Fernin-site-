# Ferlin Estética e Saúde — Site + Agendamento

Só **2 arquivos** de verdade:

- `index.html` — o site completo (visual, animações, e o widget de agendamento)
- `admin.html` — painel privado da equipe pra ver e confirmar/cancelar os agendamentos

Nenhum dos dois precisa de instalação, build ou servidor próprio. O "backend" é o **Supabase** (gratuito), que guarda os agendamentos de verdade — evitando que duas pessoas marquem o mesmo horário.

---

## Passo 1 — Criar o banco (Supabase)

1. Crie uma conta grátis em [supabase.com](https://supabase.com) e clique em **New Project**.
2. Escolha um nome, uma senha de banco (guarde essa senha em local seguro) e a região mais próxima (ex: São Paulo).
3. Espere o projeto terminar de criar (~2 minutos).
4. No menu lateral, clique em **SQL Editor** → **New query**, cole o bloco abaixo inteiro e clique em **Run**:

```sql
-- tabela de agendamentos
create table public.bookings (
  id uuid primary key default gen_random_uuid(),
  created_at timestamptz default now(),
  service text not null,
  booking_date date not null,
  time_slot text not null,
  client_name text not null,
  client_phone text not null,
  notes text,
  status text not null default 'pendente'
);

-- impede dois agendamentos ativos no mesmo dia/horário
create unique index bookings_slot_unique
  on public.bookings (booking_date, time_slot)
  where status <> 'cancelado';

-- segurança: liga o RLS (row level security)
alter table public.bookings enable row level security;

-- qualquer visitante do site pode CRIAR um agendamento
create policy "publico pode agendar"
  on public.bookings for insert
  to anon
  with check (true);

-- só a equipe logada (autenticada) pode VER a lista completa
create policy "equipe pode ver tudo"
  on public.bookings for select
  to authenticated
  using (true);

-- só a equipe logada pode ATUALIZAR status (confirmar/cancelar)
create policy "equipe pode atualizar"
  on public.bookings for update
  to authenticated
  using (true);

-- função que devolve só os horários já ocupados de um dia
-- (o site usa isso pra saber quais horários esconder, sem expor nome/telefone de ninguém)
create or replace function public.get_booked_slots(p_date date)
returns table(time_slot text)
language sql
security definer
set search_path = public
as $$
  select time_slot from public.bookings
  where booking_date = p_date and status <> 'cancelado';
$$;

grant execute on function public.get_booked_slots(date) to anon;
```

5. Ainda no Supabase, vá em **Authentication → Users → Add user**, e crie um login (e-mail + senha) para a pessoa da clínica que vai usar o `admin.html`. Repita pra cada pessoa que precisar de acesso.

6. Vá em **Settings → API**. Copie dois valores:
   - **Project URL**
   - **anon public key**

   *(Essa chave "anon" é pública por natureza — ela vai aparecer no código do site, e isso é normal e seguro: quem protege os dados é o RLS que você acabou de criar no passo 4, não o segredo da chave.)*

## Passo 2 — Colar a configuração nos arquivos

Abra `index.html`, procure (bem no topo, logo depois do `<body>`) por:

```js
window.FERLIN_CONFIG = {
  SUPABASE_URL: "COLE_AQUI_A_SUPABASE_URL",
  SUPABASE_ANON_KEY: "COLE_AQUI_A_ANON_KEY",
  WHATSAPP_NUMBER: "55XXXXXXXXXXX",
  ...
```

Substitua pelos valores do Supabase e pelo número de WhatsApp real da clínica (só dígitos, com código do país — ex: `5541999998888`).

Abra `admin.html` e faça o mesmo com `SUPABASE_URL` / `SUPABASE_ANON_KEY` (bem no topo também).

Também ajuste, se precisar, o horário de funcionamento em `BUSINESS_HOURS` (dentro do mesmo bloco, em `index.html`) — está com um horário padrão de segunda a sábado que você deve confirmar.

## Passo 3 — Subir pro GitHub

1. Crie um repositório vazio em [github.com/new](https://github.com/new) (sem marcar "Add a README").
2. Na página do repositório, clique em **"uploading an existing file"**.
3. Arraste `index.html`, `admin.html` e `README.md`.
4. **Commit changes**.

## Passo 4 — Publicar no Vercel

1. Acesse [vercel.com/new](https://vercel.com/new), faça login com o GitHub.
2. **Import** o repositório.
3. Framework: **Other** (é site estático, não precisa configurar build).
4. **Deploy**.

O site fica em `seu-projeto.vercel.app`. O painel da equipe fica em `seu-projeto.vercel.app/admin.html` — não coloque esse link em nenhum menu público do site.

---

## Como funciona o agendamento

- O cliente escolhe o serviço, a data e um horário livre (os horários já ocupados somem da lista automaticamente).
- Ao confirmar, o site grava o agendamento no Supabase e mostra um código de confirmação, com um botão pra avisar a clínica pelo WhatsApp.
- Se duas pessoas tentarem o mesmo horário ao mesmo tempo, o banco de dados rejeita a segunda tentativa (graças ao índice único do passo 1) — a pessoa recebe um aviso pra escolher outro horário.
- A equipe entra em `admin.html` com o e-mail/senha criado no Supabase e vê todos os agendamentos, podendo marcar como **Confirmado** ou **Cancelado**.

## Pendências antes de publicar

Procure por `[inserir ...]` dentro do `index.html` (seção de contato e FAQ) e substitua por:

- Endereço completo
- Horário de funcionamento completo dos demais dias
- Formas de pagamento aceitas

Os dois depoimentos na seção "Depoimentos" são ilustrativos — vale trocar por avaliações reais quando tiver.

## Stack

- HTML5 + CSS3 + JavaScript vanilla (sem framework, sem build)
- [Supabase](https://supabase.com) — banco de dados Postgres + autenticação, usado direto do navegador
- [GSAP 3](https://gsap.com/) + ScrollTrigger (CDN, com fallback automático caso não carregue)
- Google Fonts: Fraunces, Cormorant Garamond, Manrope
- Fotos da própria clínica, embutidas em base64 no `index.html` (sem pasta de imagens separada)
