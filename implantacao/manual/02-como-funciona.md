# 02 — Como funciona

História curta. Sem jargão solto: se aparecer termo técnico, a frase
seguinte traduz.

## Uma mensagem chega

1. Cliente fala no WhatsApp.
2. O motor do número (WAHA no QR, ou a API oficial) avisa o CRM.
3. Nasce ou atualiza **contato** + **conversa** no Inbox.
4. Se a conversa está com a IA **e** o canal autoriza IA para aquela
   origem: o **agente publicado** daquele número pensa (modelo + prompt +
   base da empresa + ferramentas) e responde.
5. Portões mecânicos barram o que não pode sair: STOP, LGPD, ritmo,
   janela, inventar preço, confirmar agenda sem ferramenta, jargão interno.
6. Se o cliente pede gente, o sentimento estoura, ou o agente abre
   **caso**: a conversa vai para a **Fila** do Inbox. Humano assume.
7. Follow-up **não** é este turno. É a régua que, **depois**, puxa quem
   esfriou — se o fluxo estiver publicado e o agente o armar.

Sem agente publicado, o Inbox funciona na mão. Foi o estado do lab até
aqui: WhatsApp WORKING, IA ainda não atende.

## O que “agente” quer dizer

Não é um chatbot colado. É um **atendente de primeira classe**:

- tem prompt, modelo, número, funis, base, follow-ups, horário;
- **capacidades** (pacotes: atender, vender, passar para humano…);
- versão **rascunho** vs **publicada** (publicada não se edita; muda =
  versão nova);
- **Testar** roda o motor de verdade, sem mandar no WhatsApp do cliente.

Dois agentes no mesmo número brigam. Aí entra o **roteador**: “quer
agendar” → Recepção; “quanto custa” → Comercial; o resto → fallback.

## Três jeitos de o CRM falar com o mundo

| Jeito | Direção | Tela | Quando |
|---|---|---|---|
| **Inbox / agente** | conversa | Atendimento | O cliente falou agora |
| **Token `dsk_…`** | o outro sistema **puxa** | Configurações → API Tokens | Loja, n8n, ERP consulta ou grava |
| **Webhook** | o CRM **recebe** ou **empurra** | Canais → Webhooks | Form do site entra; “ganhou” avisa o ERP |

Token e webhook **não** são o webhook do WhatsApp (esse é da Conexão).

Detalhe dos tokens: [api-tokens.md](../api-tokens.md).
Webhooks: receber dados (form → funil) e automações (quando X → avisar
URL / mandar WhatsApp / start de follow-up).

## Funil

Quadro de etapas. Exatamente **uma** ganhou e **uma** perdeu. Cada etapa
tem o passo do agente (novo → … → ganhou/perdeu). Vocabulário do nicho
mora aqui.

O agente só move card nos funis que você marcou nele. Sem funil
marcado, ele só conversa.

## Agenda

Tipos de horário (Configurações → Tipos de agendamento) + tela Agenda.
Lembrete “amanhã às 14h” é **agenda**, não follow-up.
“Faltou” na agenda pode **disparar** o follow-up de falta, se o fluxo
existir e o agente o armar.

## Follow-up

Fluxo: gatilho → espera → mensagem → condição → fim.
Rascunho não manda. Publicado sem agente armando também não manda
(aviso na Central). Manual e disparo vindo de Webhooks dispensam agente.

## Relógio do sistema

Automações e follow-ups andam com o **agendador** da instalação (cron).
No lab, `pnpm dev` + crons locais. Na VPS, o `scheduler` do compose.
Sem relógio, o Inbox ainda funciona; a régua de silêncio não.

## Isolamento

Toda tabela de cliente tem a empresa. Uma clínica não lê a outra. Quem
usa **service role** no código tem de filtrar na mão — no produto, a tela
já está atrás de sessão + papel.

## Modo de teste do número

Número novo nasce em teste: só fala com a lista de confiança. Abrir para
o público é decisão consciente (banimento). No lab, **não** prospectar no
número de teste.
