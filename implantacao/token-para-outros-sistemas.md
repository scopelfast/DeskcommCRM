# Token `dsk_` nos seus outros sistemas

Contrato de integração **11/11/2026**, conferido no código. Sem inventar campo.
Nunca cole o plaintext `dsk_…` neste arquivo nem no chat.

Mapa de escopos e catálogo: `implantacao/api-tokens.md`.
**Um exemplo de cada uso fora daqui** (Fleetbase, n8n, site, Claude): `implantacao/token-exemplos-fora.md`.
**Este arquivo é o contrato (campos, erros, tetos).**

No lab a base é `http://localhost:3020`. Na VPS: `https://SEU_DOMINIO`.
Toda chamada autenticada:

```
Authorization: Bearer dsk_<prefixo>_<segredo>
```

Nunca na query string (vaza em log de proxy). JSON `snake_case`. Datas ISO-8601 UTC.
Dinheiro em `_cents` + `currency`. IDs UUID v4.

Você já usa **`mcp:write` sozinho** para enviar mensagem. Abaixo está **tudo**
que essa chave abre, o que ela **recusa**, e o que falta marcar na tela para
os outros sistemas (Fleetbase, n8n, ERP, site, painel próprio).

---

## 0. Duas direções (não misturar)

| Direção | Peça | Quem autentica |
|---|---|---|
| **Seu sistema → CRM** | Token desta tela | `Authorization: Bearer dsk_…` |
| **CRM → seu sistema** | Webhooks (Canais → Webhooks) | HMAC do webhook, **não** o `dsk_` |

O token **não** faz o CRM avisar o outro sistema. Para receber `message.received`,
lead movido, etc., cadastre a URL no CRM (webhooks). São chaves diferentes.

O agente de WhatsApp **não** usa o seu `dsk_`. Ele tem token interno. O seu é
integração de servidor.

Uma chave = **uma empresa** (tenant). Token da Atlas no localhost não fala com
o tenant do cliente na VPS. Em produção: token criado **na tela daquela empresa**.

---

## 1. O que `mcp:write` sozinho já faz

Sem `role:*` o token nasce **atendente** (`agent`). Isso basta para envio,
lead PATCH, rascunho, mídia, funil via MCP, tags, atribuir conversa.

**Não** lista contato, **não** lê histórico, **não** abre conversa fria escolhendo
o canal (isso é gerente), **não** marca agenda (piso `ai_operator`), **não**
publica agente.

| Precisa além de `mcp:write` | Para |
|---|---|
| `mcp:read` | Listar/buscar contato, conversa, lead, fila, histórico |
| `role:manager` | Abrir conversa nova num canal escolhido; etapas; webhooks de captura; ligar automação |
| `role:admin` | Prospecção API; criar agente |
| `config:read` + `config:write` **e** `role:admin` | Editar/publicar/pausar agente e roteador |
| `messages:on_behalf` | Gravar *qual atendente* decidiu o envio |
| `messages:derived:read` | Transcrição de áudio no histórico MCP |
| `contacts:read` | Só a foto `GET …/contacts/{id}/avatar` |

Recomendação para o sistema que **já envia**:

1. Mantenha este token só de escrita (`mcp:write`) se ele só dispara WhatsApp.
2. Crie **outro** token `Leitura ERP` com só `mcp:read` para o sistema que consulta.
3. Se o mesmo worker precisa achar conversa **e** mandar: acrescente `mcp:read`
   neste (ainda sem admin). Menor que um token root.

---

## 2. Envelope HTTP (todas as rotas `/api/v1`)

Sucesso:

```json
{ "data": { }, "meta": { "cursor": null, "has_more": false, "total": null } }
```

`meta` só aparece quando a rota pagina.

Erro:

```json
{ "error": { "code": "forbidden", "message": "…", "details": {} } }
```

Header de correlação: `X-Request-Id` (guarde no log do outro sistema).

| HTTP | Significado típico |
|---|---|
| 200 / 201 | Ok |
| 400 | `Idempotency-Key` não é UUID |
| 401 | Token malformado, revogado, vencido, sem `dsk_` |
| 403 | Faltou escopo ou papel (`forbidden` / `forbidden_role`) |
| 404 | Recurso não existe **nesta** empresa (ou o token é de outra) |
| 409 | Idempotência em andamento; teto de 50 tokens (criação) |
| 413 | Arquivo > 50 MB |
| 415 | MIME de mídia não suportado |
| 422 | Body Zod / janela fechada / campo obrigatório |
| 429 | Teto de escrita (30/min) ou ritmo anti-ban do número |

Idempotência (quando a rota aceita): header `Idempotency-Key: <uuid v4>`.
Repetir a mesma chave em 24h devolve o **primeiro** resultado, não duplica.
Chave malformada = 400, não vira recibo.

Teto Bearer de escrita: 30/min por token, 600/min por empresa (janela 60s).
Envio WhatsApp ainda passa no **ritmo do número** (espaçamento / teto diário /
janela 7h–22h). 429 aqui é proteção do chip, não do servidor.

---

## 3. REST que o seu `mcp:write` já alcança

Base: `{BASE}/api/v1`. Header Bearer em todas.

### 3.1 Enviar mensagem — `POST /messages`

É o que você já usa. Papel ≥ agent, escopo `mcp:write`.

```json
{
  "conversation_id": "<uuid da conversa>",
  "type": "text",
  "body": "Olá, seu pedido saiu."
}
```

Campos (Zod `sendMessageSchema`):

| Campo | Obrigatório | Régua |
|---|---|---|
| `conversation_id` | sim | UUID da conversa **desta** empresa |
| `type` | não (default `text`) | `text` `image` `audio` `document` `sticker` `video` `location` `contact` `template` |
| `body` | texto/template | 1–4096. Sem body precisa de mídia (ver refine) |
| `media_url` | mídia por URL | URL absoluta |
| `media_storage_path` | mídia já no bucket | 1–500, o path que o upload devolveu |
| `media_mime` / `media_size_bytes` | com mídia | mime + tamanho > 0 |
| `metadata` | contato compartilhado etc. | objeto livre |
| `template_name` / `template_language` / `template_values` | `type=template` | nome **exato** Meta; `pt_BR` ≠ `pt`; values por `slotKey` |
| `reply_to_message_id` | responder bolha | UUID **da linha nossa**, não o wamid |
| `on_behalf_of_user_id` | autoria humana | UUID do membro; **exige** escopo `messages:on_behalf` + papel ≥ agent na org |

Pelo menos um: `body` **ou** `media_url` **ou** `media_storage_path`
(exceto `type=contact`, que exige `metadata.shared_contact_id` ou
`metadata.shared_contact.phone_number` ≥ 8).

Resposta típica em `data`: mensagem (`id`, `status`, `external_id`, `sent_at`, …).
Status começa `queued`/`sending` e caminha com o WAHA.

**Não existe** `GET /messages` com Bearer. Ler histórico = MCP `crm_get_conversation_history`
(`mcp:read`) ou a tela.

Janela 24h do canal oficial: texto fora = **422** `janela_fechada`. QR/WAHA local
não é Meta Cloud; a guarda de banimento (ritmo) continua.

PowerShell (não cole o token no chat):

```powershell
$base = "http://localhost:3020"
$h = @{
  Authorization = "Bearer $token"
  "Content-Type" = "application/json"
  "Idempotency-Key" = [guid]::NewGuid().ToString()
}
$body = '{"conversation_id":"UUID-DA-CONVERSA","type":"text","body":"teste do outro sistema"}'
Invoke-RestMethod -Method POST -Uri "$base/api/v1/messages" -Headers $h -Body $body
```

Node:

```js
await fetch(`${BASE}/api/v1/messages`, {
  method: "POST",
  headers: {
    Authorization: `Bearer ${token}`,
    "Content-Type": "application/json",
    "Idempotency-Key": crypto.randomUUID(),
  },
  body: JSON.stringify({
    conversation_id,
    type: "text",
    body: "Olá do ERP",
  }),
});
```

### 3.2 Abrir / achar conversa pelo telefone — `POST /conversations/open-with-contact`

É a porta que resolve o ovo-e-galinha: seu sistema tem um **telefone**, não um
`conversation_id`. `mcp:write` basta. Papel agent.

```json
{
  "phone_number": "5541999999999",
  "name": "Cliente do ERP",
  "channel_session_id": "<uuid opcional da sessão WAHA>"
}
```

| Campo | Régua |
|---|---|
| `contact_id` **ou** `phone_number` | um dos dois; telefone 8–32 chars |
| `name` | 1–200; só se for criar contato novo |
| `channel_session_id` | UUID da conexão. **Ausente** = o CRM escolhe sessão WORKING (`sessaoProntaParaEnvio`) |

Acha o contato pelas grafias do número ou cria (`fn_upsert_wa_contact`), abre/reabre
1:1. Token **não** assume a conversa (não vira “atendente”). Sessão de browser assume.

Resposta `data` traz `conversation_id` (e o que o helper devolver). Em seguida
chame `POST /messages` com esse id.

Fluxo canônico do outro sistema:

```
telefone do pedido
  → POST open-with-contact
  → conversation_id
  → POST messages
```

### 3.3 Subir mídia e depois enviar — `POST /conversations/{id}/media`

Multipart. Campo `file` obrigatório. Opcional `nota_de_voz=1` (converte AAC/webm → opus).
Teto 50 MB (`52428800`). Bearer `mcp:write`.

Resposta:

```json
{
  "data": {
    "storage_path": "<org>/<conversa>/out-<uuid>.<ext>",
    "media_mime": "audio/ogg",
    "media_size_bytes": 12345,
    "kind": "audio"
  }
}
```

Segundo passo: `POST /messages` com `conversation_id`, `type` (`image`/`audio`/`document`/…),
`media_storage_path`, `media_mime`, `media_size_bytes`. **Não** mande base64 inline.

No lab Windows o WAHA precisa ver o Storage por `host.docker.internal` (patch local).
Na VPS com https o Storage já é alcançável.

### 3.4 Rascunho no Inbox (humano envia) — `POST /conversations/{id}/drafts`

Quando a mensagem **tem** que sair de uma pessoa (bolha de atendente, IA cala),
mas o texto veio do ERP.

```json
{
  "texto": "Sua fatura venceu. Segue o boleto.",
  "origem": "erp-faturas",
  "expira_em_horas": 24
}
```

| Campo | Régua |
|---|---|
| `texto` | 1–4096 |
| `origem` | 1–64, default `integracao` |
| `expira_em_horas` | 1–720, default 24 |

**Não envia.** Devolve `draft_id` + URL `/app/inbox?id=<conversa>&rascunho=<draft_id>`.
O atendente abre, vê o texto no compositor, clica Enviar.

Aceita `Idempotency-Key` UUID.

### 3.5 Atualizar lead — `PATCH /leads/{uuid}`

Nasceu para n8n jurídico (Escavador etc.). `mcp:write`. **Não** muda etapa
(isso é MCP `crm_move_lead_stage`).

```json
{
  "title": "Processo 0001234-12.2024",
  "description": "Andamento atualizado",
  "value_cents": 150000,
  "recurring_value_cents": 50000,
  "recurring_period": "monthly",
  "recurring_months": 12,
  "currency": "BRL",
  "owner_user_id": null,
  "owner_agent_id": null,
  "expected_close_date": "2026-12-31",
  "tags": ["juridico"],
  "custom_fields": { "andamento_atual": "Aguardando perícia" },
  "contact_id": "<uuid ou null>"
}
```

Todos opcionais. `owner_user_id` e `owner_agent_id` os dois preenchidos = 422.
`recurring_period` só aceita `"monthly"`. Data `YYYY-MM-DD`.

**Não existe** `POST /leads` nem `GET /leads` com Bearer. Criar/listar/mover =
MCP (§5) com o mesmo `mcp:write` (criar/mover) ou + `mcp:read` (listar).

---

## 4. REST que o seu token atual **não** alcança

| Rota | Por quê | O que marcar |
|---|---|---|
| `GET /contacts` | exige `mcp:read` | acrescente `mcp:read` |
| `POST /contacts` | só cookie | MCP start ou tela |
| `GET /contacts/{id}/avatar` | `contacts:read` | essa caixa |
| `GET/POST /prospecting` | admin + mcp:* | `role:admin` |
| `POST /prospecting/importar` | **proibido** a token | tela |
| `POST/PATCH/DELETE /agenda/agendamentos` | `tokenRole: ai_operator` | `role:manager` (cobre) |
| Agentes `GET/PATCH …/ai/agents/{id}` | `config:*` + admin | §6.2 do outro md |
| Criar token | só sessão admin | — |

Três rotas têm código dual mas o **proxy 401** no Bearer (sem public-path):
`drafts/consume`, `notes/media`, `agenda/servicos/atualizar`. Trate como tela.

---

## 5. MCP — quando o outro sistema é um LLM (Claude/Cursor/n8n MCP)

Endpoint: `{BASE}/api/mcp` (GET/POST, Streamable HTTP / SSE).
Mesmo Bearer. Tools `crm_*`. Seu `mcp:write` **registra as tools de escrita**;
as de leitura recusam na hora (`ensureScope mcp:read`).

Para sistema **seu** (Node, Fleetbase, n8n HTTP Request) prefira **REST §3**.
MCP é protocolo JSON-RPC para modelo. REST é mais estável de programar à mão.

Tools de escrita que **já rodam** com `mcp:write` + papel agent:

| Tool | Input essencial |
|---|---|
| `crm_send_whatsapp_message` | `conversation_id`, `body` ou mídia, `idempotency_key` |
| `crm_create_conversation_draft` | `conversation_id`, `texto`, `origem?`, `expira_em_horas?` |
| `crm_create_lead` | `pipeline_id`, `stage_id`, `title` (2–200), contato/valor/tags/`custom_fields` |
| `crm_update_lead` | `lead_id` + mesmos campos do PATCH |
| `crm_move_lead_stage` | `lead_id`, `to_stage_id`, `lost_reason?` / `won_reason?` — **mesmo funil** |
| `crm_retomar_lead` | lead |
| `crm_propose_lead_fields_from_document` | documento → campos |
| `crm_assign_conversation` | `conversation_id`, `reason`: `transfer`+`to_user_id` **ou** `release`+`to_user_id` null |
| `crm_manage_tags` | `target_kind`: `conversation`\|`contact`\|`lead`, `target_id`, `add[]` / `remove[]` |
| `crm_request_human_handoff` | passar para humano |
| `crm_add_case_note` / `crm_close_human_case` / `crm_resume_ai_attendance` | caso humano |
| `crm_draft_proposal` / `crm_preparar_proposta` | proposta |
| `crm_close_demand` / `crm_propose_reactivation` | retenção |

Tools de escrita que o token **atual recusa** (papel baixo):

| Tool | Exige |
|---|---|
| `crm_start_conversation_and_send` | `role:manager` + `channel_session_id` **obrigatório** + telefone ou contato. **apenasHumano** (agente WhatsApp não chama; o seu sistema sim) |
| `crm_book_appointment` e irmãs | `role:manager` (cobre `ai_operator`) |
| `crm_schedule_followup` / `crm_enroll_followup_flow` / `crm_cancel_followup` | idem |
| `crm_save_org_memory` | idem |
| `crm_create_stage` / `crm_update_stage` / `crm_archive_stage` | `role:manager` |
| `crm_create_webhook_source` / `crm_set_webhook_source_active` | `role:manager` |
| `crm_set_automation_rule_active` | `role:manager` |

Leitura (precisa `mcp:read` no **mesmo** ou noutro token):
`crm_search_contacts`, `crm_get_contact`, `crm_list_conversations`,
`crm_get_conversation`, `crm_get_conversation_history`, `crm_list_leads`,
`crm_get_lead`, `crm_list_pipelines`, `crm_get_queue_status`,
`crm_list_event_types`, `crm_find_free_slots`, `crm_list_appointments`,
`crm_list_team_members`, `crm_list_message_templates`, `crm_search_knowledge`, …

Histórico com transcrição de áudio: ainda `messages:derived:read`.

Teto MCP: 60 chamadas/min por token, 600/min empresa, 30 escritas/min.

---

## 6. Como o outro sistema obtém os UUIDs

O CRM não aceita “mande no 419…” solto no `POST /messages`. Precisa de
`conversation_id`. Caminhos:

1. **Telefone → conversa** — `POST /conversations/open-with-contact` (§3.2). Já no seu token.
2. **Webhook inbound** — o CRM avisa seu sistema com ids. Cadastre URL em Webhooks.
3. **Tela** — copiar id da conversa / contato / lead (dev tools ou URL do Inbox).
4. **Listar** — `GET /contacts?search=` ou MCP `crm_search_contacts` (`mcp:read`).
5. **Conexão WhatsApp** — `channel_session_id` está em Conexões (id da sessão).
   Precisa dele para **escolher** de qual chip sair; sem ele o CRM pega um WORKING.

`pipeline_id` / `stage_id`: Funil na tela, ou `crm_list_pipelines` + `crm_list_stages`
(`mcp:read` / list stages é read agent).

Membro para `on_behalf_of_user_id` / `to_user_id`: Equipe na tela, ou
`crm_list_team_members` (`mcp:read`).

---

## 7. Receitas prontas para os seus sistemas

### A. ERP / Fleetbase: “pedido despachado → WhatsApp”

Token: o atual (`mcp:write`).

1. `POST /conversations/open-with-contact` `{ "phone_number": "<do pedido>" }`
2. `POST /messages` `{ "conversation_id", "body": "Pedido X saiu." }` + `Idempotency-Key` = id do pedido.

Se a bolha precisa ser de um atendente: novo token com `mcp:write` +
`messages:on_behalf`, e mande `on_behalf_of_user_id`.

Se o texto tem que ser revisado por gente: passo 2 vira `POST …/drafts`.

### B. Cobrança: “boleto venceu”

Igual A, ou rascunho se for conversa sensível.

Não faça laço de mil números no QR de teste. Ritmo + banimento.

### C. Jurídico: “andamento do processo no funil”

Token: `mcp:write` (+ `mcp:read` para achar o lead).

1. Achar lead (`crm_list_leads` / guardar `lead_id` no outro sistema na criação)
2. `PATCH /leads/{id}` `{ "custom_fields": { "andamento_atual": "…" } }`
3. Mover etapa: MCP `crm_move_lead_stage` (REST Bearer **não** move)

Criar o card: MCP `crm_create_lead` (REST `POST /leads` é **só tela**).

### D. Site: “cliente escolheu horário”

Token **novo**: `mcp:write` + `role:manager`.

MCP: `crm_list_event_types` (`mcp:read` também) → `crm_find_free_slots` →
`crm_find_and_book_appointment`. Ou REST `POST /agenda/agendamentos`.

### E. Assistente interno (Claude no escritório)

Token **novo**: `mcp:read` + `mcp:write` + `role:manager` (+ `messages:derived:read`).
Aponte o cliente MCP para `{BASE}/api/mcp`.

### F. Publicar agente por script (repetir cliente)

Token **novo**: `mcp:read` + `mcp:write` + `role:admin` + `config:read` + `config:write`.
Ver rotas `/api/v1/ai/agents` no outro md. Token antigo **não ganha** `config:*`
sozinho — cria outro.

### G. Planilha de prospecção

**Não use token.** Tela Prospecção. A rota recusa Bearer de propósito.

---

## 8. O que o outro sistema NUNCA deve fazer

- Colocar o `dsk_` no front (browser, app do cliente, GitHub, `.env` commitado).
- Um token só com todas as caixas para “todos os sistemas”.
- `GET /api/v1/messages?api_key=` — key em query é proibida no produto.
- Inventar `conversation_id` / `pipeline_id`.
- Disparo em massa no número de QR de laboratório.
- Copiar o token do localhost para a VPS (bancos diferentes; emita na VPS).
- Confiar em `contacts:write` / `leads:write` / `messages:write` / `audit:read`
  — a tela mostra, o servidor **não cobra**; a porta real é `mcp:*`.
- Achar que `mcp:write` lista contato. Não lista. É `mcp:read`.
- Achar que “Worker de import” importa planilha. Não importa.

---

## 9. Checklist para plugar um sistema novo

1. Qual empresa (seletor do topo) — o token nasce nela.
2. Nome na tela = nome do sistema (`Fleetbase pedidos`, `n8n cobrança`).
3. Escopos = o mínimo da receita §7. Sem `mcp:read` se for só disparar.
4. Validade 180 ou 365.
5. Copiar plaintext **uma vez** → cofre / secret do outro sistema.
6. Prova: `GET /contacts` (se tiver read) ou `POST open-with-contact` + `POST messages`
   para o **seu** número, não de cliente.
7. Guardar `X-Request-Id` no log do outro lado.
8. Se vazar: Revogar na tela, emitir outro, trocar no sistema. Sem confirmação no Revogar.

---

## 10. Arquivos no código (quando o contrato mudar)

| Contrato | Arquivo |
|---|---|
| Envio | `lib/schemas/messaging.ts` → `sendMessageSchema` |
| Abrir conversa | `openConversationWithContactSchema` + `lib/messaging/open-shared-contact-conversation.ts` |
| Lead PATCH | `lib/schemas/leads.ts` → `updateLeadSchema` |
| Rascunho | `lib/inbox/rascunho-sugerido.ts` |
| Upload | `app/api/v1/conversations/[id]/media/route.ts` |
| Quais paths aceitam Bearer | `lib/auth/public-paths.ts` |
| Dual cookie/token | `lib/api/auth-dual.ts` |
| Papel do token | `lib/mcp/auth.ts` → `scopesRole` |
| Tools MCP | `lib/mcp/tools/index.ts` |
| Envelope | `lib/api/wrappers.ts` |
