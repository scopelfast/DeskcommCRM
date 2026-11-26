# 07 — O que veio de novo

Recorte **nosso** (Atlas CRM). O texto oficial é
[deskcomm.com.br/changelog](https://www.deskcomm.com.br/changelog).
Quando discordar, **a tela / o changelog oficial mandam**.

| | Versão | Quando |
|---|---|---|
| **Neste lab** (`C:\DeskcommCRM`) | **1.126.0** | overlay 25/11/2026 |
| **Última no GitHub / no site** | **1.126.0** | 25/11/2026 |

O código e as 9 migrations (1.125 + 1.126) já estão no disco e no Postgres
`55322`. **Reinicie o 3020** para a tela pegar a versão. Não foi `update.sh`.

Não ligue chamada de voz. Não prospecte no número de teste.

### ⚠️ Requer atenção neste salto (1.124 → 1.126)

- **1.125** — na **VPS**, o `update.sh` recusa se faltar disco; `BACKUP_KEEP`
  no `.env` (padrão 14). Não rode `docker image prune -a` nem
  `docker system prune --volumes` (leva sessão do WhatsApp e o banco).
  Neste lab Windows isso **não se aplica**.
- **1.126** — conversões de etapa do Google Ads: com **mais de uma conta**
  cadastrada, o clique precisa trazer `google_customer_id`. Com uma conta só,
  nada muda. Negócios já abertos sem o ID da conta continuam sem enviar.
- **1.126** — tipo de agendamento passa a usar o **fuso da empresa**. Se a
  empresa não for São Paulo, confira a grade de horários dos serviços.

O que já valia no salto 1.115 → 1.124 continua: campo do funil só vai à IA
se marcado “A IA pode ler este campo”; API de serviços recusa por **código**;
`SENTRY_DSN` vazio desliga telemetria.

---

## 1.126.0 — destaques (oficial 25/11)

Fonte: [changelog 1.126.0](https://www.deskcomm.com.br/changelog/1.126.0).

Dá para mostrar no lab (depois de **reiniciar o 3020**):

- **Sala/mesa na consulta** — com o módulo Reservas de lugar, a tela e o
  agente escolhem o lugar livre; lembrete com `{{lugar}}`.
- **Mensagens agendadas que se repetem** — diária até anual, com data de
  término. Envio único continua igual.
- **Contato no negócio do funil** — “Editar negócio” agora vincula um
  contato já existente (não cria ficha nova).
- **AI/ML API** — nova empresa de IA em IA › Credenciais. Só vale se
  alguém cadastrar a chave.
- **Painel de vendas** — período num campo só; coluna Vendedor.
- **Assumir / devolver** — a devolução automática não toma a conversa
  que um atendente acabou de assumir (migration 9472, já aplicada).

Ainda **travado / não ligar neste lab:**

- **Chat no site** — código e aba existem, mas o módulo **não liga** nesta
  versão (falta o teto de IA para visitante anônimo). Não prometa.
- Chamada de voz — continua fora.

Ferramentas de agente novas: `crm_count_conversations`. Na VPS, comando
`change-email.sh` troca o e-mail de login.

## 1.125.0 — destaques (oficial 24/11)

Fonte: [changelog 1.125.0](https://www.deskcomm.com.br/changelog/1.125.0).

- **Inbox:** anotações do cliente na lateral; Fila pode ordenar “Mais
  recentes”; painel do negócio com funil em cima e valor embaixo.
- **Funil:** ao arquivar/excluir, transferir negócios em andamento para
  outro funil (contagem na frente).
- **Agente:** opção “Continuar a conversa assim que a equipe devolver ao
  automático” (nasce **desligada**).
- **Tipos de link do contato** — Configurações escolhe quais aparecem
  (Instagram, site, etc.). Desmarcar não apaga o valor gravado.
- **Canal oficial Meta:** “Conectar com o Facebook” (Embedded Signup) —
  só se o app da Meta estiver preenchido em Admin › API Oficial.
- Agente: `crm_count_leads` com `por_etapa`.

## 1.124.0 — destaques (oficial 23/11)

Fonte: [changelog 1.124.0](https://www.deskcomm.com.br/changelog/1.124.0).
Sem “Requer atenção” nesta versão.

- Teto do dia **só na abordagem** (por número; padrão desligado). Follow-up
  a quem já foi abordado não gasta esse teto (teto próprio: 200/dia).
- Negócio **não comercial** (golpe, teste): não vira perda, sai do Radar e
  do follow-up; reversível.

No meio (1.115–1.123), o que mais importa para o Atlas: mais rotas com
token `dsk_`; funil de entrada por número; módulo conectado (opcional);
classificação “não comercial”; teto só-abordagem. Texto oficial no site.

## 1.114.0 — já estava no lab (16/11)

Fonte: [changelog 1.114.0](https://www.deskcomm.com.br/changelog/1.114.0).
16 itens novos, 9 correções. **Sem** “Requer atenção”.

### Dá para vender / mostrar (quando o lab/VPS estiver na 1.114)

| O quê | Onde | Liga sozinho? |
|---|---|---|
| Criar pessoa da equipe **com senha provisória** (sem e-mail) | Equipe → “Criar usuário com senha”. Senha aparece **uma vez**. Primeiro acesso obriga troca. | Só o **dono da instalação**, ou o admin da empresa se a VPS tiver **uma empresa só**. Com várias empresas, o admin continua no **convite**. |
| IA **envia PDF/material** na conversa (catálogo, tabela, ficha) | O agente manda quando o cliente pede. Não entra na base de conhecimento. MCP: `crm_send_material`. | Sim, se o material existir. **Ainda não há tela de cadastro** nesta versão. |
| Automação **“Quando um negócio for alterado”** | Automações → gatilho + condição “campo X foi alterado”. Edição por outra automação **não** dispara. | Não: você cria a regra. |
| Aviso na **troca automática de agente** | Roteador → “Mensagem de passagem ao cliente”. Uma vez, antes do agente novo. | Não: campo vazio = silêncio, como hoje. |
| Tarefa que **repete** (ex.: toda segunda às 9h) | Criar/editar tarefa. | Não. |
| **10 templates visuais** | Configurações → Aparência (Obsidiana, Aurora, Safira…). Marca (nome, cor, logo) continua em Marca. | Não: quem não escolhe, vê o visual de sempre. |
| Atendente cria **empresa + pessoa + contato** num passo | Página da empresa → “Adicionar pessoa com contato”. Apagar continua só Gerente. | Sim, a permissão do Atendente muda. |

### Atlas / agência — opcional, desligado de fábrica

- **Agências** administram empresas-cliente: Administração → Recursos
  opcionais, depois habilita cada agência. Sem isso, **nenhuma** tela de
  agência existe. Ainda **não** tem cobrança por % nem papéis finos.
- **Catálogo por feed** (JSON/CSV da revenda, a cada 2 h): mesmo interruptor
  de Recursos opcionais. Feed vazio não desativa produto.
- **Amigo** (agenda de clínica): Administração → Módulos. Sem instalar, não
  existe. Cada empresa cola o token do Amigo **na tela**, não no chat.

### Ainda incompleto nesta versão — **não prometer**

- Tela de cadastro de **materiais** (o envio já existe; o cadastro, não).
- Interruptor de **cobrança mensal** quando o negócio é ganho (o banco já
  sabe; a tela vem na próxima).
- Tela da **tabela de valores** (partida/limite por procedimento) e o agente
  negociando com ela.
- Tela para escolher o **modelo de reserva** do agente (o campo já existe no
  banco; a UI vem depois).

### Token `dsk_` (Atlas-go)

Mais rotas aceitam Bearer: anúncios, auditoria, importações, métricas,
relatórios, canais, voz, mensagens, módulos. Organização sai **do token**,
nunca do body. `mcp:read` para ler; `mcp:write` para importar planilha,
ocultar mensagem e régua de abandono. Métricas da org inteira pedem papel
**manager** no token.

Banco externo (imóvel/produto): a receita escolhe o **funil/etapa** do
interesse; o negócio guarda **qual item** o cliente escolheu. Sem receita
configurada, a IA passa para a equipe.

### Correções que o operador sente (1.114)

- IA **para** se a conversa virou pingue-pongue (20 respostas / 60 min) e
  abre aviso na Central. Follow-up e Testar não entram na conta.
- Instagram/Facebook: **some o microfone**; agente com nota de voz responde
  **em texto** (a Meta recusa o áudio do CRM). WhatsApp QR **não muda**.
- Origem **Google Ads** no contato, mesmo em ficha pessoal / módulo no meio.
- Agente pode **repetir a política** da base de conhecimento sem o guardrail
  tratar como “promessa inventada”.
- Anexo que vai para o card de outro funil **não duplica**; se a 1ª cópia
  falhou, a próxima reposição completa.

---

## Já neste lab — 1.107 + 1.108 (12/11/2026)

Nenhuma das duas tem **Requer atenção**. Cadastro aberto, WhatsApp QR,
portas `5532x` e tokens `dsk_` **não mudaram** nessas duas.

### Inbox e conversa

- **Exportar conversa em PDF** (1.107) — menu ⋯. Notas internas não entram.
- **Selo “Mencionado”** (1.107).
- **Vários anexos no +** (1.108).
- **IA com dono humano** (1.108) — nasce desligado.
- **Filtro por fila / setor** (1.108) — só com módulo Setores.

### Funil, agenda, agente

- Última mensagem **com hora** no card; previsão com **mensalidade** ao lado
  do setup (1.107).
- **Expandir** negócio; conversas de **todos os números**; Faturamento com
  etiquetas/clientes que mais vendem (1.108).
- Agenda **Quadro**; lembrete Meet; editar convidados (1.108).
- Follow-up com horário marcado (1.107); público no roteador; agente novo
  sabe que herdou; voz por agente (1.108).

### Contatos, captação, MCP

- Duplicados pela **conta WhatsApp** (1.107). Duplicados **só pelo nome**
  nascem desligados (1.108).
- Anonimizar **leva fichas fundidas** (1.108) — já rodou neste lab.
- Formulários Meta e Ouvidoria: desligados / módulo (1.107–1.108).
- MCP: `sent_by_user_id` / `sent_by_user_name` (1.108).

### Correções já no 3020

- Sem agente no ar → Fila, não “Automático atendendo”.
- Busca ignora acento.
- Consulta ao acervo: gestor abre a origem; erro diz o motivo.

### O que **não** prometer (vale no lab e na 1.114)

- Login da Meta para conectar o número oficial.
- Arrastar compromisso no Quadro.
- Chamada de voz neste lab.

---

## Onde ler o oficial

- Site: [v1.126.0](https://www.deskcomm.com.br/changelog/1.126.0)
- Nesta pasta: `CHANGELOG.md` (abre em `[1.126.0]`).
- Ajuda no CRM logado: `/app/ajuda`.
