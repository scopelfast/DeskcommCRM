# 01 — O que é

## Uma frase

Sistema operacional de vendas com **agentes de IA nativos** e **WhatsApp**,
instalado na **sua** VPS: atendimento, funil, agenda, follow-up e
integração — pessoas e IA na mesma mesa.

No mercado: alternativa **open source e self-hosted** a Kommo, Octadesk,
Intercom. Nós revendemos com a marca **Atlas CRM**.

O motor é o [DeskcommCRM](https://github.com/melgarafael/DeskcommCRM)
(MIT). A marca na tela sai do banco (`/admin/marca` e Configurações ›
Marca), não de um fork do código.

## Para quem serve

Qualquer negócio que **vende conversando** no WhatsApp:

- clínica / consultório / salão
- imobiliária
- serviços / agência / obra
- curso / mentoria
- loja
- escritório de advocacia

O funil troca o vocabulário (lead = paciente / interessado / cliente;
ganhou = agendado / fechou / pago). O núcleo é o mesmo.

## O que o cliente ganha (promessa honesta)

| Pilar | Na prática |
|---|---|
| WhatsApp de verdade | Número por QR (WAHA) ou API oficial. Ritmo, janela, aquecimento, STOP. |
| IA que opera | Lê histórico, consulta a base da empresa, agenda, move o card, chama humano. |
| CRM no mesmo lugar | Funil, contatos, tarefas, campanhas, propostas (se ligar). |
| Humano no comando | Fila, handoff, teto de gasto de IA, auditoria. |
| Dados na VPS dele (ou na sua) | Multi-empresa com isolamento. LGPD nativa. |
| Fora do CRM | Token `dsk_…` (o outro sistema puxa) e webhooks (o CRM empurra ou recebe). |

## O que **não** prometer

- Que a IA publica sozinha no primeiro dia. Publicar exige número WORKING,
  credencial validada, pessoa clicando.
- Que follow-up funciona só de desenhar o fluxo. Precisa **publicar** e
  **ligar no agente**.
- Chamada de voz / SIP neste lab local — o servidor não tem o serviço; ligar
  arrisca o WhatsApp.
- Prospecção em massa no número de teste.
- Que o token da lista de API Tokens dá para copiar depois. Plaintext **uma
  vez**.
- Que o site na internet alcança `localhost`. Webhook de entrada precisa da
  VPS (https) ou túnel.

## Atlas CRM × Atlas (loja)

| | Atlas CRM (este produto) | Atlas loja (`atlas-go`) |
|---|---|---|
| O que é | Atendimento + IA + funil | Fila, senha, agenda, caixa da loja |
| Lab | `localhost:3020` | `localhost:3010` |
| Ligação | — | Token `dsk_` + URL do CRM no Perfil da loja |

Vendem-se **juntos ou separados**. A loja usa o CRM como motor de WhatsApp;
o CRM não depende da loja.

## Como se ganha dinheiro (revenda)

O software é gratuito. Você cobra:

1. **Implantação** (montar empresa, agente, funil, treinar).
2. **Hospedagem** (VPS sua, várias empresas dentro).
3. **Mensalidade** — se ligar **Cobrança dos seus clientes** no painel
   (`/admin/cobranca`): planos, Stripe (cartão) ou Asaas (Pix/boleto).

Sem cobrança ligada, as empresas são **isentas**. Detalhe oficial:
`docs/saas/README.md`.

## Papéis dentro de uma empresa

`viewer` < `atendente` < `gerente` < `admin` da empresa.

Quem instalou a VPS é **admin da plataforma** (`/admin`) — transversal,
não é papel da clínica.

Criar agente (`/app/ai/agents/new`) é **admin da empresa**. Montar
follow-up e webhook é **gerente ou acima**. Inbox é de todos.

## Onde a pessoa entra

| URL | Quem |
|---|---|
| `/` e `/signup` | Cadastro público (se o modo da instalação permitir) |
| `/login` | Equipe da empresa |
| `/app` | O CRM |
| `/admin` | Dono da instalação |
| `/app/ajuda` | Mapa de tarefas **dentro** do produto |
