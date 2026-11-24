# Exemplos — o outro sistema chamando o CRM

Cada bloco é **um uso fora daqui**: Fleetbase, n8n, ERP, site, Claude.
Nunca cole o `dsk_…` real neste arquivo.

Lab: `http://localhost:3020` · VPS: `https://SEU_DOMINIO`

No PowerShell, uma vez por sessão:

```powershell
$BASE = "http://localhost:3020"
$token = $env:DESKCOMM_TOKEN   # cofre / variável — não no chat
$h = @{ Authorization = "Bearer $token"; "Content-Type" = "application/json" }
```

`curl.exe` no Windows (o `curl` alias do PowerShell é outra coisa):

```text
Authorization: Bearer dsk_…
```

Resposta sempre `{ "data": … }` ou `{ "error": { "code", "message" } }`.
Guarde `X-Request-Id`.

---

## 1. ERP / Fleetbase — pedido saiu, avisar no WhatsApp

**Token:** `mcp:write` (o que você já tem)  
**Fora:** worker do Fleetbase, depois que o status vira “despachado”.

```powershell
# 1) telefone do pedido → conversa
$aberto = Invoke-RestMethod -Method POST -Uri "$BASE/api/v1/conversations/open-with-contact" -Headers $h -Body (@{
  phone_number = "5541999999999"
  name          = "Cliente pedido 8821"
} | ConvertTo-Json)
$cid = $aberto.data.conversation_id

# 2) envia (Idempotency-Key = id do pedido: retry não duplica)
$h2 = $h.Clone(); $h2["Idempotency-Key"] = "00000000-0000-0000-0000-000000008821"
Invoke-RestMethod -Method POST -Uri "$BASE/api/v1/messages" -Headers $h2 -Body (@{
  conversation_id = $cid
  type = "text"
  body = "Pedido 8821 saiu para entrega."
} | ConvertTo-Json)
```

**No CRM:** Inbox mostra a conversa e a bolha de saída. Sem `messages:on_behalf` a bolha é do sistema, não de um atendente.

---

## 2. Mesmo envio, bolha com nome do atendente

**Token:** `mcp:write` + `messages:on_behalf`  
**Fora:** o ERP sabe que a Joana autorizou o aviso.

```json
POST /api/v1/messages
{
  "conversation_id": "<uuid>",
  "type": "text",
  "body": "Sua fatura venceu.",
  "on_behalf_of_user_id": "<uuid da Joana em Equipe>"
}
```

Sem o escopo extra o campo é recusado. A Joana tem que ser membro **desta** empresa, papel ≥ atendente.

---

## 3. Cobrança — texto no Inbox para HUMANO enviar

**Token:** `mcp:write`  
**Fora:** Asaas/Stripe avisa “boleto vencido”. Você **não** dispara sozinho; o atendente revisa.

```powershell
Invoke-RestMethod -Method POST -Uri "$BASE/api/v1/conversations/$cid/drafts" -Headers $h -Body (@{
  texto = "Sua fatura de outubro venceu. Segue o boleto: https://…"
  origem = "asaas"
  expira_em_horas = 24
} | ConvertTo-Json)
```

**No CRM:** Inbox abre com o texto no compositor e aviso de origem. Nada vai ao WhatsApp até o clique.

---

## 4. Enviar PDF / áudio (canhoto, boleto, nota de voz)

**Token:** `mcp:write`  
**Fora:** o outro sistema tem o arquivo no disco.

```powershell
# upload (multipart) — não é JSON
$up = curl.exe -s -H "Authorization: Bearer $token" `
  -F "file=@C:\temp\canhoto.pdf" `
  "$BASE/api/v1/conversations/$cid/media"
# $up.data.storage_path / media_mime / media_size_bytes

Invoke-RestMethod -Method POST -Uri "$BASE/api/v1/messages" -Headers $h -Body (@{
  conversation_id     = $cid
  type                = "document"
  body                = "Canhoto da entrega 8821"
  media_storage_path  = "<storage_path devolvido>"
  media_mime          = "application/pdf"
  media_size_bytes    = 12345
} | ConvertTo-Json)
```

Áudio gravado: `type: "audio"`. Campo extra no upload `nota_de_voz=1` se for PTT do browser (converte para opus). Teto 50 MB. Sem base64 no body.

---

## 5. Prontuário / ERP — só consultar paciente

**Token:** só `mcp:read` (outro token, sem escrita)  
**Fora:** tela do prontuário, ao abrir o paciente.

```powershell
Invoke-RestMethod -Uri "$BASE/api/v1/contacts?search=1199999&limit=20" -Headers $h
```

Query: `search`, `tag` (repete `?tag=vip&tag=orcamento`), `modo=e|ou`, `source`, `blocked=yes|no`, `cursor`, `limit` 1–100, `order_by` (`last_activity_at` `created_at` `display_name` `email` `phone_number`), `order_dir=asc|desc`.

**No CRM:** nada muda. Só leitura desta empresa.

Foto (painel próprio segue o 307):

```text
GET /api/v1/contacts/<uuid>/avatar
Authorization: Bearer …   (escopo contacts:read)
```

---

## 6. Jurídico / n8n — andamento do processo no funil

**Token:** `mcp:write` (+ `mcp:read` para achar o lead)  
**Fora:** n8n consulta Escavador e atualiza o card.

```powershell
Invoke-RestMethod -Method PATCH -Uri "$BASE/api/v1/leads/$leadId" -Headers $h -Body (@{
  custom_fields = @{ andamento_atual = "Aguardando perícia" }
  tags          = @("juridico")
} | ConvertTo-Json -Depth 5)
```

Isto **não** muda a coluna do kanban. Mover etapa = MCP (exemplo 10).  
Criar o card também é MCP (exemplo 9) — `POST /api/v1/leads` é **só tela**.

---

## 7. Site da clínica — cliente escolheu horário

**Token:** `mcp:write` + `role:manager` (+ `mcp:read` para listar tipos)  
**Fora:** landing / Calendly próprio, depois do submit.

REST (tipo = UUID da tela Tipos de agendamento):

```powershell
Invoke-RestMethod -Method POST -Uri "$BASE/api/v1/agenda/agendamentos" -Headers $h -Body (@{
  event_type_id = "<uuid do tipo Consulta>"
  starts_at     = "2026-11-20T13:00:00-03:00"
  contact_id    = "<uuid>"
  title         = "Consulta — Maria"
  guest_email   = "maria@exemplo.com"
} | ConvertTo-Json)
```

Remarcar: `PATCH` mesmo path `{ "id", "starts_at" }`.  
Cancelar: `DELETE` body `{ "id", "reason": "cliente desmarcou" }` (`reason` 3–500, obrigatório).

**GET lista pela REST é só sessão** (tela). Listar de fora: MCP `crm_list_appointments` com `mcp:read`.

**No CRM:** Agenda ganha o bloco. Se o tipo exige aprovação, volta reservado — não diga “confirmado” no site.

---

## 8. Site — formulário vira conversa no chip certo

**Token:** `mcp:write` + `role:manager`  
**Fora:** form “Quero orçamento” precisa sair do número comercial, não do pessoal.

Isto **não** é REST `open-with-contact` (lá o canal é opcional e o CRM escolhe um WORKING).  
É MCP `crm_start_conversation_and_send` — `channel_session_id` **obrigatório**.

No n8n / código, o caminho estável de quem não quer JSON-RPC:  
`POST /conversations/open-with-contact` com `channel_session_id` preenchido + `POST /messages`.  
O UUID da sessão está em **Conexões** (a linha do WhatsApp).

```powershell
$aberto = Invoke-RestMethod -Method POST -Uri "$BASE/api/v1/conversations/open-with-contact" -Headers $h -Body (@{
  phone_number        = "5541888888888"
  name                = "Lead do site"
  channel_session_id  = "<uuid da conexão comercial>"
} | ConvertTo-Json)
```

Contato marcado **pessoal** recusa abrir.

---

## 9. Criar negócio no funil (MCP — o REST de criar é tela)

**Token:** `mcp:write`  
**Fora:** quando o formulário qualifica o lead.

Cliente MCP (Claude Desktop / Cursor) — cola no config do **outro** app:

```json
{
  "mcpServers": {
    "atlas-crm": {
      "url": "http://localhost:3020/api/mcp",
      "headers": { "Authorization": "Bearer dsk_COLOQUE_NO_COFRE" }
    }
  }
}
```

Aí o modelo chama `crm_create_lead`:

```json
{
  "pipeline_id": "<uuid do funil>",
  "stage_id": "<uuid da etapa Novo>",
  "title": "Obra rua X",
  "contact_id": "<uuid>",
  "value_cents": 150000,
  "currency": "BRL",
  "source": "site",
  "tags": ["site"],
  "custom_fields": { "bairro": "Centro" }
}
```

`pipeline_id` / `stage_id`: Funil na tela, ou tools `crm_list_pipelines` / `crm_list_stages` (`mcp:read`).

---

## 10. Mover o card de coluna

**Token:** `mcp:write`  
**Fora:** n8n “pagamento confirmado → etapa Pago”.

Tool `crm_move_lead_stage`:

```json
{
  "lead_id": "<uuid>",
  "to_stage_id": "<uuid da etapa Pago>",
  "won_reason": "pix"
}
```

Mesmo funil. Perda exige `lost_reason` do vocabulário do funil. Trocar de funil **não** existe por token — humano na tela.

---

## 11. Tag no contato quando o ERP marca “inadimplente”

**Token:** `mcp:write`  
**Fora:** evento “boleto 30 dias”.

Tool `crm_manage_tags`:

```json
{
  "target_kind": "contact",
  "target_id": "<uuid do contato>",
  "add": ["inadimplente"],
  "remove": ["em-dia"]
}
```

`target_kind`: `conversation` | `contact` | `lead`. Tags viram minúsculas, ≤40 chars, ≤20 no alvo.  
Isto **dispara** a automação “quando o contato ganhar uma tag” — igual a tag pela tela.

---

## 12. Atribuir conversa a um atendente

**Token:** `mcp:write`  
**Fora:** plantão / distribuição no seu sistema.

Tool `crm_assign_conversation`:

```json
{ "conversation_id": "<uuid>", "reason": "transfer", "to_user_id": "<uuid do atendente>" }
```

Devolver à fila:

```json
{ "conversation_id": "<uuid>", "reason": "release", "to_user_id": null }
```

`to_user_id` tem que ser membro desta empresa. Lista: `crm_list_team_members` (`mcp:read`).

---

## 13. Passar para humano (handoff)

**Token:** `mcp:write`  
**Fora:** seu bot percebeu que não resolve.

Tool `crm_request_human_handoff` — a conversa entra na fila humana; o Inbox mostra o caso.

---

## 14. Claude / Cursor no escritório — operar o CRM por chat

**Token:** `mcp:read` + `mcp:write` + `role:manager` (+ `messages:derived:read` se quiser transcrição)  
**Fora:** config MCP do exemplo 9, no **Claude Desktop** ou **Cursor**, não neste repo.

O modelo passa a ter as tools `crm_*` (buscar contato, histórico, funil, agenda, enviar).  
Não publique este token no site. É chave de escritório.

---

## 15. Publicar agente por script (repetir cliente)

**Token:** `mcp:write` + `role:admin` + `config:read` + `config:write`  
**Fora:** pipeline de implantação Atlas.

```powershell
# listar (mcp:read também ajuda; criar usa mcp:write + admin)
Invoke-RestMethod -Uri "$BASE/api/v1/ai/agents" -Headers $h

# publicar versão já testada
Invoke-RestMethod -Method POST -Uri "$BASE/api/v1/ai/agents/$agentId/publish" -Headers $h -Body (@{
  version_id = "<uuid da versão>"
} | ConvertTo-Json)
```

Token antigo **sem** `config:*` toma 403. Cria token novo.  
Pausar/arquivar: `PATCH /api/v1/ai/agents/{id}` com `config:write` + `role:admin`.

---

## 16. Prospecção — pausar campanha (não disparar)

**Token:** `mcp:write` + `role:admin`  
**Fora:** seu painel de plantão, botão “parar busca”.

```powershell
Invoke-RestMethod -Method POST -Uri "$BASE/api/v1/prospecting" -Headers $h -Body (@{
  action = "pause"
  id     = "<uuid da campanha>"
} | ConvertTo-Json)
```

Por token **só** `configure`, `search`, `pause`. **Iniciar/retomar campanha (envio frio) exige a tela** — 403 de propósito.

Planilha `.xlsx`: **tela**, sem token.

---

## 17. O CRM avisar o SEU sistema (direção inversa)

Isto **não** usa `dsk_`.

**Fora:** um HTTP listener no Fleetbase/n8n (`POST https://seu-sistema/hooks/deskcomm`).  
**Aqui:** Canais → Webhooks → URL + secret HMAC.

Quando chegar mensagem, seu sistema valida a assinatura e usa o token `dsk_` só se for **responder** (exemplos 1–4).

Duas chaves: HMAC (entrada) + `dsk_` (saída).

---

## 18. n8n — nó HTTP Request (molde)

| Campo | Valor |
|---|---|
| Method | POST |
| URL | `{{$env.DESKCOMM_URL}}/api/v1/messages` |
| Authentication | Generic → Header `Authorization` = `Bearer {{$env.DESKCOMM_TOKEN}}` |
| Header | `Idempotency-Key` = `{{$json.pedido_uuid}}` |
| Body JSON | `{ "conversation_id": "…", "type": "text", "body": "…" }` |

Não coloque o token no workflow exportado para Git.

---

## Qual exemplo é o seu token de agora (`mcp:write` só)

Serve **1, 3, 4, 6 (PATCH), 9–13**.  
Não serve **5** (falta `mcp:read`), **2** (falta `on_behalf`), **7–8** (falta `role:manager`), **15–16** (admin/config).

Contrato dos campos: `implantacao/token-para-outros-sistemas.md`.  
Escopos da tela: `implantacao/api-tokens.md`.
