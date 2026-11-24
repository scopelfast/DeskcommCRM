# 03 — Mapa de telas (todas as funções com porta)

Fonte: `lib/navigation/catalogo.ts` (o menu, o hub e o ⌘K saem daqui).
Conferido neste clone em 11/11/2026. Se a tela discordar, a tela manda.

Papel mínimo: se a coluna está vazia, **todos** da empresa vêem.
Módulo/capacidade: a porta **some** se estiver desligada na instalação
ou na empresa.

No lab: prefixo `http://localhost:3020`.

---

## Atendimento (dia a dia)

| Tela | Caminho | Serve para | Quem |
|---|---|---|---|
| Inbox | `/app/inbox` | Conversas WhatsApp, você e a IA lado a lado. Contador da Fila. | todos |
| Radar | `/app/radar` | Quem esfriou e ainda está aberto. | todos |
| Agenda | `/app/agenda` | O que está marcado, com quem. | todos |
| Respostas rápidas | `/app/templates` | Scripts do atendente (não é template da Meta). | todos |

## CRM

Hub: **Ver tudo em CRM** `/app/crm`.

| Tela | Caminho | Serve para | Quem | Some se |
|---|---|---|---|---|
| Funis | `/app/kanban` | Lista dos funis → abre o quadro. | todos | |
| Contatos | `/app/contacts` | Pessoas e histórico. | todos | |
| Tarefas | `/app/tasks` | O combinado, com prazo. Contador das **suas**. | todos | |
| Planos de tarefa | `/app/tasks/planos` | Sequência reutilizável. | todos | |
| Campanhas | `/app/campaigns` | Disparo para uma lista, no ritmo do número. | todos | |
| Produtos | `/app/products` | Catálogo que a IA responde preço. | todos | |
| Etapas do funil | `/app/settings/tenant/pipelines` | Colunas, vocabulário, motivos de perda. | gerente+ | |
| Prospecção | `/app/prospecting` | Busca empresas e abordagem gradual com IA. | admin | |
| Empresas | `/app/companies` | Cadastro B2B. | todos | módulo `crm_b2b` |
| Pessoas | `/app/people` | Decisores da empresa. | todos | `crm_b2b` |
| Importações | `/app/imports` | CSV/XLSX B2B. | gerente+ | `crm_b2b` |
| Propostas | `/app/proposals` | Orçamento → aceite. | todos | capacidade `propostas` |
| Comandas | `/app/comandas` | O que foi feito e quanto paga. | todos | módulo `financeiro` |
| Chamadas | `/app/calls` | Histórico de voz por IA. | gerente+ | telefonia ligada na org |
| Cold Call | `/app/cold-call` | Fila de ligações com roteiro. | atendente+ | módulo `cold_call` |

## Agente de IA

Hub: **Ver tudo em IA** `/app/ai`.

| Tela | Caminho | Serve para | Quem | Some se |
|---|---|---|---|---|
| Agentes | `/app/ai/agents` | Prompt, modelo, número, publicar. **Novo** é admin. | gerente+ | |
| Follow-ups | `/app/ai/followups` | Régua de silêncio / falta / etapa. | gerente+ | |
| Roteadores | `/app/ai/routers` | Qual agente pega a conversa. | gerente+ | |
| Credenciais | `/app/ai/credentials` | Chave do provedor, validar. | gerente+ | |
| Provedores | `/app/ai/providers` | Modelo de atendimento vs barato; Jev. | gerente+ | |
| Conhecimento | `/app/ai/knowledge/sources` | FAQ e PDF que a IA consulta. | gerente+ | |
| Memória | `/app/ai/memory` | Regras da casa. Publicar = admin. | gerente+ | |
| Skills | `/app/ai/skills` | Roteiros (`agendamento`, `objecao-preco`…). | gerente+ | |
| Casos | `/app/ai/cases` | Quando a IA passou para humano. | atendente+ | |
| Alertas | `/app/ai/inbox` | Central: o que precisa de decisão. | todos* | |
| Aviso no WhatsApp | `/app/ai/cases/avisos` | Caso novo no WhatsApp da equipe. | admin | |
| Propostas (IA) | `/app/ai/proposals` | A IA sugere melhoria nela mesma. | todos | |
| Execuções | `/app/ai/runs` | O que a IA fez / falhou. | gerente+ | |
| Uso e orçamento | `/app/ai/usage` | Gasto e teto do mês. | gerente+ | |
| Fluxos de atendimento | `/app/ai/atendimento` | Perguntas conduzidas no turno. | gerente+ | módulo `fluxos_atendimento` |

\* o papel mínimo da Central pode ser mais alto conforme o destino.

## Canais

| Tela | Caminho | Serve para | Quem |
|---|---|---|---|
| Conexões | `/app/connections` | Números: QR ou oficial, saúde, pausa, proteção, grupos, templates Meta. | admin |
| Webhooks | `/app/webhooks` | Form entra; automação avisa outro sistema. | gerente+ |
| Nuvemshop | `/app/integrations/nuvemshop` | Pedidos/clientes da loja. Só pela **busca**. | admin |

## Análise

Hub: **Ver tudo em Análise** `/app/analise`.

| Tela | Caminho | Serve para | Quem | Some se |
|---|---|---|---|---|
| Desempenho | `/app/metrics` | Funil e atendente, 30 dias. | todos | |
| Atividades | `/app/activities` | O que equipe e agentes fizeram. | todos | |
| Meta Ads (leitura) | `/app/ads/meta` | Custo do anúncio que trouxe gente. | gerente+ | |
| Faturamento | `/app/faturamento` | Quanto entrou. | todos | |
| Evolução da IA | `/app/ai/evolution` | Onde o agente erra. | gerente+ | |
| Audit Log | `/app/audit` | Quem fez o quê. | gerente+ | |
| Honorários | `/app/honorarios` | Parcelas do caso. | todos | módulo `honorarios` |

## Organização (Configurações)

Hub: `/app/settings`. Rodapé do menu.

### Sua conta

| Tela | Caminho | Serve para | Quem |
|---|---|---|---|
| Perfil | `/app/settings/profile` | Nome, idioma, fuso, avatar. | todos |
| Segurança | `/app/settings/security` | MFA, códigos, sessões, chamada de voz (não ligar no lab). | todos |
| Notificações | `/app/settings/notifications` | Push, o que avisar. | todos |
| App no celular | `/app/settings/aplicativo` | PWA na tela inicial. | todos |

### Sua empresa

| Tela | Caminho | Serve para | Quem | Some se |
|---|---|---|---|---|
| Equipe | `/app/team` | Convites e papéis. | todos | |
| Distribuição | `/app/settings/atendimento` | Quem recebe conversa nova. | gerente+ | |
| Automações (freio) | `/app/settings/automacoes` | Freio da escolha da IA nas regras. | gerente+ | |
| Tags | `/app/settings/tags` | Vocabulário de etiquetas. | gerente+ | |
| Recursos opcionais | `/app/settings/recursos` | O que a **empresa** liga. | gerente+ | |
| Organização | `/app/settings/tenant` | Razão, CNPJ, DPO, retenção. | admin | |
| Conversões | `/app/settings/conversoes` | Devolver venda ao anúncio. | admin | |
| Meta Ads (token) | `/app/settings/meta-ads` | Conta de anúncio **desta** empresa. | admin | |
| Marca | `/app/settings/marca` | Nome e cor **desta** empresa. | admin | |
| Plano e cobrança | `/app/settings/billing` | O que **esta** empresa paga a você. | admin | |
| Tipos de agendamento | `/app/settings/tenant/agenda` | O que se marca, duração, quem atende. | todos | |
| Aviso de agendamento | `/app/agenda/avisos` | WhatsApp da equipe a cada horário. | admin | |
| Meta de vendas | `/app/settings/metas` | Meta do mês para o Desempenho. | gerente+ | |
| Propostas (config) | `/app/settings/tenant/proposals` | Validade e condições. | gerente+ | propostas |
| Modelos de proposta | `/app/settings/tenant/proposals/modelos` | Modelos da empresa. | gerente+ | propostas |
| Financeiro | `/app/settings/tenant/financeiro` | Contas, formas, comissões. | todos | |
| Extensões | `/app/extensions` | Guias JSON de nicho. | todos | |

### Dados e acesso

| Tela | Caminho | Serve para | Quem | Some se |
|---|---|---|---|---|
| LGPD | `/app/lgpd/requests` | Exportar / apagar dados do titular. | admin | |
| API Tokens | `/app/settings/api-tokens` | `dsk_…` para outro sistema. | admin | |
| Trunk SIP | `/app/settings/voip-trunk` | Provedor de voz. | admin | |
| Servidor MCP externo | `/app/settings/servidor-mcp-externo` | ERP/consulta da IA. Escrever = dono da instalação. | admin | |
| Dados externos | `/app/integracao-dados` | Banco de outro sistema, leitura. | todos | `banco_externo` |
| Ajuda | `/app/ajuda` | Caminho de cada tarefa. | todos | |

## Painel do dono (`/admin`)

Não é menu da clínica. É a VPS.

| Tela | Serve para |
|---|---|
| `/admin/dashboard` | Saúde da instalação |
| `/admin/tenants` + `/new` | Empresas-clientes. **Novo tenant** = cliente novo |
| `/admin/users` | Contas da instalação |
| `/admin/cadastro` | Aberto / com aprovação / só convite |
| `/admin/marca` | Marca **Atlas** no login |
| `/admin/email` | SMTP de todas as empresas |
| `/admin/cobranca` | Planos, Stripe/Asaas (módulo cobrança) |
| `/admin/modulos` | Módulos de tabela (honorários, financeiro) |
| `/admin/sistema` | Recursos opcionais da **instalação** |
| `/admin/meta` | App da Meta da **plataforma** (não o ads do cliente) |
| `/admin/google` | Google Agenda da instalação |
| `/admin/extensoes` | Pacotes JSON instalados na VPS |
| `/admin/destinos-internos` | Liberar IP interno (vazio = não fala com 192.168) |
| `/admin/configuracao` | E-mail de suporte, DPO da instalação |
| `/admin/platform-admins` | Quem é dono da VPS |
| `/admin/usage` | Custo de IA por empresa |
| `/admin/audit` | Auditoria da plataforma |
| `/admin/incidents` | Incidentes |
| `/admin/inbox` | Supervisão só leitura |
| `/admin/lgpd` | LGPD entre empresas |

Atualização da VPS: `/app/settings/atualizacao` (só dono). No **localhost**
a tela diz “indisponível” — esperado. Não “consertar”.

## Módulos que nascem desligados

Ligam em **Administração → Sistema** (flag) ou **Modulos** (tabela):

`crm_b2b`, `propostas`, `fluxos_atendimento`, `banco_externo`, `cobranca`,
`cold_call`, `honorarios`, `financeiro`, `setores`, `avisos`,
`conferir_whatsapp`, `mcp_por_agente`, `login_codex`.

Núcleo útil **com zero módulos**. Não ligue no pitch o que a instalação
padrão não mostra.

## O que não tem tela (e o cliente pergunta)

- Worker de IA e cron: processos da VPS, não menu.
- Webhook do WAHA: a Conexão configura sozinha.
- Token interno do agente (TTL curto): não aparece em API Tokens.

Lista viva: skill `deskcomm-operacao` → `funcionalidades/so-api-ou-sem-tela.md`.
