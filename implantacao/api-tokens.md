# API Tokens — o que o servidor faz de verdade

Estudo de **11/11/2026**, conferido no código desta pasta (não na memória).
Tela: `/app/settings/api-tokens` (neste lab: `http://localhost:3020/app/settings/api-tokens`).
Nunca cole plaintext `dsk_…`, senha ou chave de IA neste arquivo.

**Contrato (campos, erros, tetos):** `implantacao/token-para-outros-sistemas.md`.  
**Exemplos de fora (um de cada sistema):** `implantacao/token-exemplos-fora.md`.

Fonte: `ApiTokensClient.tsx`, `lib/mcp/auth.ts`, `lib/api/auth-dual.ts`,
`lib/auth/public-paths.ts`, `app/api/mcp/route.ts`, `lib/mcp/tools/`,
rotas `app/api/v1/**` com `resolveAuthDual`, spec 11, aula `curso/licoes/api-tokens.json`.

O que é **CONFIRMADO** está no código. Onde a tela promete e o servidor **não cobra**, está escrito.

---

## 1. O que é

O token **não é login**. É chave de servidor: outro programa fala com **esta empresa**
sem abrir o navegador.

- A organização sai da **linha do token no banco**, nunca do body nem da URL.
- Prefixo `dsk_`. Formato: `dsk_<8-hex>_<segredo-base64url>`.
- O texto completo aparece **uma vez** na criação. O CRM guarda só o hash SHA256 (`token_hash`).
- Se perder: cria outro e **revoga** este. Não dá para reexibir.
- Quem cria precisa ser **administrador da empresa** (cookie de sessão). Token **não** cria token.

Caminho na tela: **Configurações → Dados e acesso → API Tokens**.

Quem não é admin da empresa cai em `/403`.

O agente de WhatsApp **não usa** o token da tela. O runtime emite um token efêmero interno
(`actor:ai_agent`, `agent_run:<uuid>`, TTL ~5 min) que não aparece nesta lista.

---

## 2. Ciclo de vida

| Ação | Efeito |
|---|---|
| Criar | POST `/api/v1/settings/api-tokens` (só sessão admin). Devolve `plaintext` + aviso. Audit `token.created`. |
| Listar | Prefixo, escopos, Ativo/Revogado, data de expiração. **Não** mostra `last_used_at` na tabela (a coluna existe; o Bearer atualiza; a tela não pinta). |
| Usar | `Authorization: Bearer dsk_…` (header, **nunca** query). |
| Expirar | `expires_in_days` 1–365, ou vazio = até revogar. Depois da data o servidor recusa; a coluna Status **continua “Ativo”** — conferir as duas. |
| Revogar | POST `…/api-tokens/{id}/revoke`. Imediato, **sem confirmação**, sem volta. Audit `token.revoked`. |
| Teto | **50 tokens ativos** por empresa (`fn_teto_de_tokens_ativos`). Revogado/vencido não conta. Estouro → 409 `api_token_teto_atingido`. |

`last_used_ip` existe no schema e **não é gravado** no TypeScript.

Empresa suspensa: REST recusa. MCP aceita o token marcado e só deixa ferramentas de privacidade/LGPD (decisão do produto).

---

## 3. As duas portas

Todo token válido destas duas portas. Sem entrada em `lib/auth/public-paths.ts`, o `proxy.ts`
devolve **401 antes** do handler — o Bearer nem chega.

| Porta | URL | Para quem |
|---|---|---|
| **MCP** | `POST`/`GET` `/api/mcp` | Claude Desktop, Cursor, n8n MCP, outro agente. Ferramentas `crm_*`. |
| **REST** | rotas `/api/v1/…` com auth dual | n8n, Make, worker próprio, site, ERP. |

```http
Authorization: Bearer dsk_<prefixo>_<segredo>
Content-Type: application/json
Idempotency-Key: <uuid>   # onde a rota/MCP aceita; TTL 24h
```

JSON `snake_case`. Resposta `{ data, meta? }` ou `{ error: { code, message } }`.
`X-Request-Id` em toda resposta.

Não existe namespace paralelo: a mesma rota serve cookie **ou** Bearer. Empresa do token =
tenant. Token da Atlas **não** lê empresa cliente.

---

## 4. Escopos: tela vs servidor

O papel do token **não é o seu cargo**. É o **maior** `role:*` marcado
(`scopesRole` em `lib/mcp/auth.ts`). Sem nenhum, nasce **atendente** (`agent`, rank 2).
Gerente + admin juntos valem **admin** (não depende da ordem do clique).

Ranks: `viewer` 1 < `agent` 2 < `ai_operator` 3 < `manager` 4 < `admin` 5.
A tela **não** oferece `role:ai_operator`. `role:manager` ou `role:admin` cobrem agenda.

| Caixa na tela | O servidor cobra? | Sem isto |
|---|---|---|
| `mcp:read` | **Sim** — lê MCP + `GET /api/v1/contacts` + GETs dual | Recusa: missing scope `mcp:read` |
| `mcp:write` | **Sim** — age no MCP + envio, lead PATCH, rascunho, mídia, agenda, follow-up, prospecção | Recusa em toda escrita |
| `role:manager` | **Sim** — conversa fria, etapas, webhooks, automações | `Role 'agent' insufficient` |
| `role:admin` | **Sim** — prospecção, criar agente; **obrigatório** junto de `config:*` | Config de IA: 403 `forbidden_role` |
| `config:read` | **Sim** — ler agente/roteador (**e** `role:admin`) | `mcp:read` sozinho **não** lê config do agente |
| `config:write` | **Sim** — editar, testar, publicar, pausar, arquivar agente; CRUD roteador (**e** `role:admin`) | Token antigo **não ganha** isto na atualização; cria token novo |
| `contacts:read` | **Só** foto: `GET /api/v1/contacts/{uuid}/avatar` | Listar contato usa `mcp:read` |
| `messages:on_behalf` | **Sim** — `POST /api/v1/messages` com `on_behalf_of_user_id` | Envio “em nome de” recusado; o token continua sendo quem envia |
| `messages:derived:read` | **Sim** — transcrição de áudio/mídia no histórico MCP | Histórico chega **sem** o texto extraído (LGPD) |
| `contacts:write` | **Não cobra** | Não cria contato por REST (POST contato é **só sessão**) |
| `leads:read` / `leads:write` | **Não cobra** | Lead usa `mcp:read` / `mcp:write` |
| `messages:read` / `messages:write` | **Não cobra** | Mensagem usa `mcp:read` / `mcp:write` |
| `audit:read` | **Não cobra** | Não há rota Bearer de audit log |

Uma chave **só** com `contacts:write` / `leads:write` / `audit:read` é recusada no que importa.
A aula do produto já avisa: “o CRM ainda não as confere”.

Escopos que **existem no código e a tela não oferece**:

| Escopo | Quem emite | Efeito |
|---|---|---|
| `actor:ai_agent` + `agent_run:<uuid>` | Runtime do agente (`mintEphemeralToken`) | Ator vira `ai_agent`, não integração |
| `role:ai_operator` | Mint interno / seeds | Agenda e alguns writes de retenção |
| `integration:<nome>` | Provisionar tenant (`rotateIntegrationApiKey`) | Só para rotacionar/revogar o token daquela integração; **não** é gate de rota |

---

## 5. Nome “Worker de import”

A caixa de nome é só apelido. **Importar planilha de prospecção não aceita token.**

`POST /api/v1/prospecting/importar` autentica, e se `via === "token"` devolve
403 *“Importar planilha exige a tela.”* De propósito: não está em `public-paths.ts`.

O que **aceita** Bearer em prospecção é `GET`/`POST /api/v1/prospecting`
(`mcp:read` / `mcp:write` + papel admin no token) — buscar/pausar campanha, não a planilha.

Planilha: **Prospecção na tela**, logado como admin.

---

## 6. Como usar ao máximo — um token por sistema

Não marque todas as caixas num token só. “Usar tudo” = **vários tokens**, cada um no menor
poder que o sistema precisa. Teto 50 ativos. Se um vazar, revoga **só** aquele.

### 6.1 Assistente / Claude / Cursor (MCP)

Nome: `Assistente MCP da operação`  
Escopos: `mcp:read` + `mcp:write` + `role:manager` + `messages:derived:read`  
Validade: 365 (ou menos).

Isto abre o catálogo MCP (ler CRM, funil, agenda, follow-up, enviar WhatsApp,
abrir conversa fria, etapas, webhooks). **Não** publica agente.

Endpoint: `http://localhost:3020/api/mcp` (no lab; na VPS, `https://SEU_DOMINIO/api/mcp`).

Cliente MCP (Streamable HTTP / SSE), header `Authorization: Bearer dsk_…`.
A spec 11 desenhou Claude Desktop / Cursor; o transporte é HTTP, não stdio.

### 6.2 Configurar agente e roteador por API (IaC)

Nome: `Configuração de IA`  
Escopos: `mcp:write` + `role:admin` + `config:read` + `config:write`  
(e `mcp:read` se também listar).

Sem `config:write` **e** `role:admin`, publicar/pausar/arquivar agente fecha 403.
`mcp:write` + `role:admin` **não basta** para config (issue #2052 / PR #2194).

Rotas: ver §8.

### 6.3 n8n / Make — enviar WhatsApp e abrir conversa

Nome: `n8n atendimento`  
Escopos: `mcp:write` + `role:manager`  
Extra: `messages:on_behalf` se o fluxo tiver que gravar **qual atendente** decidiu.

- Responder conversa existente: `POST /api/v1/messages` **ou** tool `crm_send_whatsapp_message`
- Abrir conversa nova num **número/canal escolhido**: tool `crm_start_conversation_and_send`
  **ou** `POST /api/v1/conversations/open-with-contact` depois `POST /api/v1/messages`
- `POST /api/v1/contacts` **não** aceita Bearer (só cookie). Contato novo pelo MCP
  (`phone_number` na tool de start) ou pela tela.

Corpo mínimo de envio:

```json
{
  "conversation_id": "<uuid>",
  "type": "text",
  "body": "Olá"
}
```

Opcional: `on_behalf_of_user_id` (uuid do membro, papel ≥ atendente, **e** escopo
`messages:on_behalf`). Mídia: primeiro `POST /api/v1/conversations/{id}/media`,
depois enviar com `media_url` / path. `Idempotency-Key` UUID, senão retry duplica.

### 6.4 n8n jurídico / monitoramento de processo

Nome: `n8n processos`  
Escopos: `mcp:write` (+ `mcp:read` se consultar).  
Comentário no código: PATCH de lead nasceu para Escavador/Jusbrasil/Codilo/Judit.

`PATCH /api/v1/leads/{uuid}` — Bearer `mcp:write`. Criar lead: tool `crm_create_lead`
(papel ≥ agent, então o default já serve) ou a tela.

### 6.5 Agenda do site / booking

Nome: `Agenda do site`  
Escopos: `mcp:write` + `role:manager` (cobre `ai_operator` exigido na rota).

Tools: `crm_list_event_types` → `crm_find_free_slots` → `crm_find_and_book_appointment`
ou `crm_book_appointment`. REST: `POST/PATCH/DELETE /api/v1/agenda/agendamentos`.
`GET` de listar agendamentos na REST é **só sessão**; listar pelo MCP (`crm_list_appointments`).

### 6.6 Rascunho sugerido no Inbox

Nome: `Sugestão de resposta`  
Escopos: `mcp:write`

`POST /api/v1/conversations/{id}/drafts` — o atendente vê o texto no compositor
e envia (ou a integração consome). Tool: `crm_create_conversation_draft`.

### 6.7 Foto do contato num painel próprio

Nome: `Avatar do contato`  
Escopos: `contacts:read` (e na prática `mcp:read` se também listar).

`GET /api/v1/contacts/{uuid}/avatar` → 307 para URL assinada do bucket.

### 6.8 Prospecção (buscar / pausar)

Nome: `Prospecção API`  
Escopos: `mcp:write` + `role:admin` (+ `mcp:read` para GET).

`GET`/`POST /api/v1/prospecting`. **Não** a planilha (§5).

### 6.9 Prontuário / ERP só leitura

Nome: `Prontuário (leitura)`  
Escopos: **só** `mcp:read`

Lista contatos (`GET /api/v1/contacts`), tools de leitura. Sem escrita, sem WhatsApp.

### 6.10 O que NÃO existe por token (não inventar)

- Criar/revogar token
- Importar planilha de prospecção
- `POST /api/v1/contacts` (criar contato REST)
- `GET /api/v1/leads` listagem (Bearer só no PATCH `{uuid}`; listar é MCP `crm_list_leads`)
- `GET /api/v1/messages/{id}` e irmãos
- Audit log REST
- Telas `/admin` (plataforma, tenants, SMTP, marca da VPS)
- Login, equipe, convite, LGPD de titular pela REST dual
- Métricas `/api/v1/metrics/funil` — **sem** Bearer no código
- Workers/crons (`INTERNAL_CRON_SECRET`, não `dsk_`)
- Cobrança Stripe/Asaas, SIP/WaCalls, Zernio
- Publicar follow-up por id (vários subpaths de follow-up **não** estão em public-paths;
  coleção `GET/POST /api/v1/ai/followup-flows` e `from-model` sim)

---

## 7. Catálogo MCP (o que o token alcança em `/api/mcp`)

Auth: `validateBearerToken`. Cada tool exige `mcp:read` **ou** `mcp:write` + papel mínimo.
Teto: 60 chamadas/min por token, 600/min por empresa, **30 escritas/min** por token
(protege o número de WhatsApp). Sem Redis o teto é por processo.

`apenasHumano: true` = o **agente publicado** do WhatsApp não chama; o token da tela
(integração / Claude) **chama**, se tiver o papel.

### Leitura (`mcp:read`, papel em geral `agent`)

| Tool | Serve para |
|---|---|
| `crm_search_contacts` / `crm_get_contact` | Achar e ler contato |
| `crm_propose_contact_field` | Sugerir campo do contato |
| `crm_list_conversations` / `crm_get_conversation` / `crm_get_conversation_history` | Fila e histórico (transcrição só com `messages:derived:read`) |
| `crm_get_queue_status` | Fila |
| `crm_list_leads` / `crm_get_lead` | Funil |
| `crm_list_pipelines` / `crm_get_pipeline_forecast` | Pipelines e previsão |
| `crm_search_knowledge` / `crm_list_knowledge_sources` | Base de conhecimento |
| `crm_list_improvement_proposals` | Propostas de melhoria |
| `crm_get_org_memory` | Memória da empresa |
| `crm_list_contact_orders` / `crm_search_products` | Pedidos e produtos |
| `crm_get_honorarios_contrato` / `crm_list_honorarios_parcelas` | Honorários (módulo) |
| `crm_preparar_proposta` | Preparar proposta |
| `crm_describe_external_data` / `crm_query_external_data` | Banco externo cadastrado |
| `crm_consultar_receita` | Receita externa |
| `crm_list_privacy_requests` | Pedidos LGPD |
| `crm_get_prospecting_site_audit` | Auditoria de site da prospecção |
| `crm_list_stages` / `crm_list_tags` | Etapas e tags |
| `crm_list_message_templates` / `crm_render_message_template` | Respostas prontas |
| `crm_list_webhook_sources` / `crm_list_webhook_source_events` | Origens de captura |
| `crm_list_automation_rules` / `crm_list_automation_runs` | Automações |
| `crm_list_team_members` | Equipe |
| `crm_list_followups` / `crm_list_at_risk_leads` | Follow-ups e risco |
| `crm_list_available_attendants` / `crm_list_human_cases` / `crm_get_human_case` | Casos humanos |
| `crm_list_event_types` / `crm_find_free_slots` / `crm_list_appointments` | Agenda |

### Escrita (`mcp:write`)

Papel **agent** (default do token já serve), salvo nota:

| Tool | Papel | Nota |
|---|---|---|
| `crm_create_lead` / `crm_update_lead` / `crm_move_lead_stage` / `crm_retomar_lead` | agent | Funil |
| `crm_propose_lead_fields_from_document` | agent | Campos a partir de documento |
| `crm_send_whatsapp_message` | agent | Conversa **já existente** |
| `crm_start_conversation_and_send` | **manager** | **apenasHumano** — abre conversa + 1ª msg num canal escolhido |
| `crm_create_conversation_draft` | agent | Rascunho no Inbox |
| `crm_assign_conversation` / `crm_manage_tags` | agent | Atribuir e tags |
| `crm_request_human_handoff` | agent | Passar para humano |
| `crm_find_and_book_appointment` / `crm_book_appointment` / `crm_reschedule_appointment` / `crm_cancel_appointment` / `crm_confirm_appointment` / `crm_set_appointment_outcome` | ai_operator | `role:manager` cobre |
| `crm_schedule_followup` / `crm_enroll_followup_flow` / `crm_cancel_followup` | ai_operator | |
| `crm_close_demand` / `crm_propose_reactivation` | agent | |
| `crm_add_case_note` / `crm_close_human_case` / `crm_resume_ai_attendance` | agent | Caso humano |
| `crm_draft_proposal` | agent | |
| `crm_save_org_memory` | ai_operator | |
| `crm_create_stage` / `crm_update_stage` / `crm_archive_stage` | **manager** | **apenasHumano** |
| `crm_create_webhook_source` / `crm_set_webhook_source_active` | **manager** | **apenasHumano** |
| `crm_set_automation_rule_active` | **manager** | **apenasHumano** |

Servidor MCP externo cadastrado na org (ainda sem tela rica): o agente interno pode
chamar tools de fora; isso **não** é o token da tela.

---

## 8. REST que aceita Bearer (inventário)

Sessão usa o papel da pessoa. Token usa escopo + papel dos `role:*`.
`tokenRole` = piso **só no Bearer**.

| Método | Caminho | Escopo Bearer | Papel token | Faz |
|---|---|---|---|---|
| GET | `/api/v1/contacts` | `mcp:read` | ≥ viewer | Listar contatos |
| POST | `/api/v1/contacts` | — | — | **Só sessão.** Bearer 401 no proxy |
| GET | `/api/v1/contacts/{uuid}/avatar` | `contacts:read` | ≥ viewer | Foto (307) |
| POST | `/api/v1/messages` | `mcp:write` | ≥ agent | Enviar; `on_behalf` extra |
| PATCH | `/api/v1/leads/{uuid}` | `mcp:write` | ≥ agent | Atualizar lead |
| POST | `/api/v1/conversations/open-with-contact` | `mcp:write` | ≥ agent | Abrir conversa |
| POST | `/api/v1/conversations/{id}/drafts` | `mcp:write` | ≥ agent | Rascunho sugerido |
| POST | `/api/v1/conversations/{id}/drafts/consume` | `mcp:write` | ≥ agent | Consome rascunho. **Fora** de public-paths → Bearer tende a 401 no proxy |
| POST | `/api/v1/conversations/{id}/media` | `mcp:write` | ≥ agent | Upload de mídia outbound |
| POST | `/api/v1/conversations/{id}/notes/media` | `mcp:write` | ≥ agent | Anexo de nota. **Fora** de public-paths |
| POST/PATCH/DELETE | `/api/v1/agenda/agendamentos` | `mcp:write` | ≥ **ai_operator** | Marcar/remarcar/cancelar. GET listar = sessão |
| GET/POST/PATCH/DELETE | `/api/v1/agenda/tipos` | `mcp:read` / `mcp:write` | mix | Tipos de compromisso |
| GET/POST | `/api/v1/agenda/servicos` | `mcp:write` no POST | manager no POST | Serviços |
| POST | `/api/v1/agenda/servicos/atualizar` | `mcp:write` | agent | **Fora** de public-paths |
| GET | `/api/v1/prospecting` | `mcp:read` | admin | Ler prospecção |
| POST | `/api/v1/prospecting` | `mcp:write` | admin | Escrever/pausar |
| POST | `/api/v1/prospecting/importar` | — | — | **Recusa token** (exige tela) |
| GET/POST | `/api/v1/ai/followup-flows` | `mcp:read` / `mcp:write` | viewer / manager | Listar/criar fluxo |
| POST | `/api/v1/ai/followup-flows/from-model` | `mcp:write` | manager | Criar a partir de modelo |
| GET/POST | `/api/v1/ai/agents` | `mcp:read` / `mcp:write` | manager / admin | Listar / **criar** agente (`mcp:*`, não `config:*`) |
| GET/PATCH/DELETE | `/api/v1/ai/agents/{uuid}` | `config:read` / `config:write` | **admin** | Ler/editar/pausar/arquivar |
| POST | `/api/v1/ai/agents/{uuid}/publish` | `config:write` | **admin** | Publicar |
| GET/POST | `/api/v1/ai/agents/{uuid}/versions` | `mcp:read` / `mcp:write` | manager / admin | Versões |
| GET/PATCH | `/api/v1/ai/agents/{uuid}/versions/{vid}` | `config:read` / `config:write` | **admin** | Ver/editar versão |
| POST | `…/versions/{vid}/test` | `config:write` | **admin** | Testar versão |
| GET/POST | `/api/v1/ai/routers` | `config:read` / `config:write` | **admin** no write | Roteadores |
| GET/PATCH/DELETE | `/api/v1/ai/routers/{uuid}` | idem | **admin** | CRUD roteador |
| PUT | `/api/v1/ai/routers/{uuid}/members` | `config:write` | **admin** | Membros |
| POST | `/api/v1/ai/routers/{uuid}/test` | `config:write` | **admin** | Testar roteador |

Rotas de **criar/revogar token** (`/api/v1/settings/api-tokens`) = **só cookie admin**.

Três rotas chamam `resolveAuthDual` mas **não** estão em `public-paths`
(`drafts/consume`, `notes/media`, `agenda/servicos/atualizar`): o proxy
responde 401 ao Bearer. Tratar como **só tela** até alguém abrir a porta.

---

## 9. Prova rápida (sem colar a chave no chat)

Depois de copiar o plaintext **uma vez**, no PowerShell (troque URL se a porta não for 3020):

```powershell
$token = "COLE_AQUI_E_NAO_MANDE_NO_CHAT"
$h = @{ Authorization = "Bearer $token" }

# deve listar contatos da EMPRESA do token
Invoke-RestMethod -Uri "http://localhost:3020/api/v1/contacts" -Headers $h
```

- 401 → chave errada, revogada, vencida, ou esqueceu `dsk_`
- 403 missing scope → faltou `mcp:read`
- 200 com `data` → porta REST ok
- corpo de **outra** empresa = bug (não deve acontecer; org sai da linha)

MCP: o cliente tem que falar JSON-RPC no `/api/mcp`; um GET solto no browser não prova ferramenta.

Envio: `Idempotency-Key` UUID. Ritmo anti-banimento também vale para token
(`ritmo-do-envio-por-token.ts`) — laço agressivo queima o **número**, não o servidor.

---

## 10. Tetos e auditoria

| Teto | Número |
|---|---|
| Chamadas MCP por token | 60 / 60s |
| Chamadas MCP por empresa (todos os tokens) | 600 / 60s |
| Escritas MCP por token | 30 / 60s |
| Escritas REST Bearer | o mesmo `tetoDeEscritaDoToken` |
| Tokens ativos | 50 / empresa |
| Guess de Bearer inválido | IP 30/5 min; valor 5/5 min |

Toda tool MCP audita `mcp.tool_called` com `actor_api_token_id`.
Mutação REST dual também audita. Ator do token é `api_token` (id = id da linha),
**não** a pessoa que criou (`created_by` é só proveniência).

---

## 11. Aplicar no Atlas (comercial)

Cada **empresa cliente** (tenant) tem a **própria** lista de tokens.
Admin da empresa A não emite chave da empresa B. Plataforma (`/admin`) não substitui isto.

Receitas que vendem:

1. **Site marca consulta** → token §6.5 (agenda).
2. **ERP/prontuário lê paciente** → token §6.9 (`mcp:read`).
3. **n8n dispara WhatsApp** → token §6.3 (`mcp:write` + `role:manager`). Prospecção fria em massa no QR de teste: **não**.
4. **Claude/Cursor opera o CRM** → token §6.1.
5. **Instalar agente como código** (repetir cliente) → token §6.2 (`config:*` + `role:admin`).
6. **Monitorar processo e mover funil** → token §6.4.

Não entregar um “token root” com todas as caixas. Contrato com o cliente: uma chave
por sistema, validade, revogar na hora que o sistema sair.

Esta pasta Windows **não** é a VPS. Token criado no localhost vale neste banco local
(portas 5532x). Na VPS, cria de novo na tela daquela instalação. Não copiar `.env.local`
nem plaintext para o GitHub oficial.

---

## 12. Receita de criação na tela (checagem)

1. Logado como admin da **empresa certa** (seletor do topo).
2. Configurações → API Tokens → Criar token.
3. Nome = sistema que vai usar (não “Worker de import” se for n8n/Claude).
4. Marcar **só** as caixas da receita (§6). Sem `mcp:read` ou `mcp:write` o MCP inteiro recusa.
5. Sem `role:manager`, abrir conversa fria e etapas recusam.
6. Sem `role:admin` + `config:write`, publicar agente recusa.
7. Expirar 180 ou 365, ou vazio.
8. Criar → **copiar agora** → colar no n8n/Claude/cofre. Fechar. Não cola no chat.
9. Prova §9. Se vazar: Revogar, emitir outro, trocar no sistema.

---

## 13. Arquivos para reabrir este estudo

| Arquivo | Por quê |
|---|---|
| `app/app/settings/api-tokens/_components/ApiTokensClient.tsx` | Lista de caixas da tela |
| `lib/mcp/auth.ts` | Hash, papel, ator |
| `lib/api/auth-dual.ts` | Cookie ou Bearer |
| `lib/auth/public-paths.ts` | Sem linha aqui, Bearer = 401 |
| `app/api/mcp/route.ts` | Porta MCP |
| `lib/mcp/tools/index.ts` + `lib/mcp/tools/catalogo/` | Catálogo |
| `lib/mcp/rate-limit.ts` | 60 / 600 / 30 |
| `app/api/v1/prospecting/importar/route.ts` | Planilha exige tela |
| `lib/mcp/tools/start-conversation.ts` | Conversa fria (manager, apenasHumano) |
| `lib/mcp/media-derived-read.ts` | Transcrição opt-in |
| `curso/licoes/api-tokens.json` | Aula (dica de escopos “decorativos” um pouco atrás: `contacts:read` hoje cobra na foto) |
| `docs/specs/11-spec-mcp-server-internal.md` | Contrato MCP (draft; o código manda) |
