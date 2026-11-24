# Checklist de implantação — cliente

Copie este arquivo para `pacote-<slug-do-cliente>.md` e dê baixa ali.
Não edite este modelo com dados de um cliente.

Nunca cole chave de IA, senha ou token neste arquivo.

A ordem importa: cada bloco exige o anterior. O sistema recusa publicar agente sem número WORKING e sem credencial validada.

Nicho (marque um):

- [ ] Clínica / consultório / salão
- [ ] Imobiliária
- [ ] Serviços / agência / obra
- [ ] Curso / mentoria / infoproduto
- [ ] Loja
- [ ] Escritório de advocacia
- [ ] Outro: _______________

Cliente: _______________
Empresa no CRM: _______________
Implantador: _______________
Início: ____/____/________
Go-live: ____/____/________

---

## Como nasce a empresa (antes da triagem)

Não é Configurações → Organização. Aquilo **edita** a empresa em que você já está.

Caminho: **Administração → Tenants** (`/admin/tenants`) → **Novo tenant**
(`http://localhost:3000/admin/tenants/new`). Só o dono da instalação (platform admin).

- Nome de exibição, slug, razão social, CNPJ
- **E-mail do responsável** = e-mail **do cliente**, nunca o seu (`admin@local.test`).
  Se usar o seu, a empresa vira sua e o convite não sai.
- Plano Standard/Pro/Enterprise aqui é **rótulo**. Só vira cobrança de verdade quando
  Cobrança dos seus clientes estiver ligada.
- **Criar organização** → copiar o convite (24 h). Sem e-mail da VPS configurado,
  mande o link no WhatsApp. Se vencer: Equipe, na empresa nova.

Você entra como admin provisório, troca no seletor do topo (hoje: Deskcomm Local) e
aí sim preenche este checklist **dentro** da empresa nova.

Não apague Deskcomm Local: é o laboratório. Cliente real = empresa nova.

Quem pode se cadastrar **sozinho** em `/signup` é outra tela, da **instalação**:
**Administração → Cadastro** (`/admin/cadastro`). Três modos (vale para a VPS
inteira; quem já tem conta continua entrando):

- **Aberto** — qualquer um cria conta e empresa na hora (padrão).
- **Com aprovação** — cria conta, a empresa espera você aprovar nesta tela.
- **Só por convite** — `/signup` sem convite não entra. Convite (Tenants ou Equipe) passa sempre.

**Decisão Atlas (divulgação / Instagram):** os dois interruptores **desligados** =
modo **aberto**. Quem chega em `/signup` cria conta e a própria empresa na hora.

- Link público: `https://SEU_DOMINIO/signup` (e a home `/`).
- Convite (Tenants / Equipe) continua valendo para cliente que você montar à mão.
- **Com aprovação** só se quiser filtrar spam de anúncio (a pessoa espera você
  aprovar; não entra na hora).
- **Só por convite** trava o Instagram: `/signup` sem convite não entra.

Na VPS, **antes de divulgar:** Administração → E-mail (`/admin/email`). Sem
isso, quem se cadastra pelo Instagram não recebe confirmação / “esqueci a
senha”. SMTP da hospedagem **ou** Resend (um só). Depois `update.sh` para o
login herdar o SMTP. Depois, se for cobrar pelo sistema: plano com **Usar no
cadastro**.

Não rode `update.sh` no Windows. No localhost o aviso amarelo do GoTrue só aparece
se o convite estiver ligado.

---

## 0. Triagem (preencher antes de cadastrar)

### Negócio

- Como se chama e o que faz (uma frase):
- Cidade / fuso:
- Cliente típico e o que pede primeiro:

### Canal

- Número de WhatsApp (já conectado?):
- Horário em que a IA responde (dias, início, fim):
- Número novo? (sim = aquecimento / limite baixo no começo):

### IA

- Provedor (OpenRouter / Anthropic / OpenAI / Google):
- Tem chave da OpenAI à parte, para áudio e base de conhecimento? (sim/não)
- Teto mensal de gasto com IA:

### Funil

- Etapas (4 a 8; exatamente uma ganhou e uma perdeu):
- Como chamam cliente / negócio / ganhou / perdeu:
- Motivos de perda:

### Agente

- Um agente só, ou papéis separados:
- Nome e tom (caloroso / objetivo / formal):
- O que pode fazer sozinho (agendar, mover funil, proposta, desconto até X):
- O que nunca faz; quando chama uma pessoa:

### Conhecimento e regras da casa

- 10–30 perguntas frequentes com resposta oficial:
- Documentos (PDF/MD/TXT, até 20 MB cada):
- Regras da casa (horário, política, endereço):
- Piso de preço / desconto máximo / parcelas:

### Time

- Quem assume quando a IA passa (e-mail e papel):
- Distribuição: manual ou rodízio:

---

## 1. Canal

- [ ] **Canais → Conexões** — número por QR com status WORKING
- [ ] Número saiu do **modo de teste** (ou a lista de telefones autorizados cobre o teste)
- [ ] Proteção de envio conferida no próprio número (janela, ritmo, aquecimento)
- [ ] Teste: mensagem de um celular → aparece em **Atendimento → Inbox**
- [ ] Teste: resposta pela Inbox → chega no celular

Sem WORKING, não publique agente.

---

## 2. Credenciais de IA

- [ ] **Agente de IA → Credenciais → Adicionar**
- [ ] Provedor, nome (“Chave principal”) e chave cadastrados **na tela** (não neste arquivo)
- [ ] Credencial **validada** (só validada publica agente)
- [ ] Se o atendimento não for OpenAI: chave OpenAI cadastrada para áudio e base de conhecimento

Sem OpenAI, o agente não ouve áudio nem indexa documentos — a tela avisa.

---

## 3. Provedores / modelos

- [ ] **Agente de IA → Provedores**
- [ ] Atendimento: modelo que usa ferramentas
- [ ] Roteador / follow-up: modelo barato (ou Automático)

---

## 4. Funil

- [ ] **Configurações → Funis** — funil criado com o nome do pacote
- [ ] **Etapas do funil** na ordem
- [ ] Exatamente **uma** etapa ganhou e **uma** perdeu
- [ ] Cada etapa tem o **passo do agente** (novo, contatado, qualificando, qualificado, negociando, ganhou, perdeu)
- [ ] Vocabulário (cliente / negócio / ganhou / perdeu) preenchido
- [ ] Motivos de perda cadastrados
- [ ] Funil ativo (o primeiro ativo vira o padrão)

Etapa sem passo do agente o agente não usa.

---

## 5. Conhecimento

- [ ] **Agente de IA → Conhecimento → Adicionar material**
- [ ] FAQ (pares pergunta/resposta) — 10 a 30
- [ ] Documentos (PDF, MD ou TXT) — preço, política, catálogo, se houver
- [ ] Status **pronto** (indexação terminou)
- [ ] Sem chave OpenAI: anotado o que fica de fora (áudio + documentos)

Preço e política moram aqui, não no prompt. Duas fontes divergem.

---

## 6. Follow-ups (depois do funil; antes ou junto do agente)

Tela: **Agente de IA → Follow-ups** (`/app/ai/followups`). É da **empresa**.
Não é Webhooks (aquilo dispara na hora). Aqui é um **roteiro no tempo** por pessoa:
silêncio, etapa, falta, etc. → espera → mensagem → fim.

Duas abas: **Fluxos** (a receita) e **Fila** (quem está dentro agora; dá para cancelar).

Três portões, os três: 1) **Publicar** o fluxo  2) ligar no agente em *follow-ups que arma*
3) agente **publicado** no número. Rascunho ou desligado no agente = ninguém recebe.
A Central avisa fluxo publicado sem agente.

- [ ] **Agente de IA → Follow-ups**
- [ ] Clínica: **Começar de um modelo** (consulta / exame / cirurgia / falta — só os que o cliente usa)
- [ ] Outros nichos: fluxo criado (gatilho → espera → mensagem → fim)
- [ ] **Publicar** no construtor (rascunho não roda)
- [ ] Fluxo ligado no agente, campo *follow-ups que arma*

---

## 7. Agente

- [ ] **Agente de IA → Agentes → Novo agente**
- [ ] Nome, tom e prompt (identidade, o que o negócio faz, diagnóstico, limites, quando chama humano)
- [ ] Prompt **não** repete o que o motor já faz (STOP, horário, “não invente preço”)
- [ ] Prompt **não** cita ferramenta interna nem jargão (`lead_id`, etapa `qualified`)
- [ ] Credencial + modelo + **número WORKING**
- [ ] Funis que ele pode mover
- [ ] Materiais de conhecimento que consulta
- [ ] Follow-ups que arma
- [ ] Palavras de passagem para humano
- [ ] Horário de atendimento
- [ ] Capacidades: pacote *vender* se for o caso; críticas **uma a uma** (mensagem avulsa, cancelar agenda, fechar caso)
- [ ] Salvo como **rascunho**

---

## 8. Testar (antes de publicar)

- [ ] Botão **Testar** na versão — mensagem de primeira conversa
- [ ] Texto, ações tentadas e portões conferidos
- [ ] Roteiro do nicho (5 frases típicas) passou no que se espera
- [ ] Pessoa viu o teste

---

## 9. Publicar

- [ ] A **pessoa** clicou em **Publicar** (ou mandou publicar por escrito)
- [ ] Recusa do sistema, se houver, resolvida (credencial, WORKING, modelo, capacidade)
- [ ] Próximo atendimento já usa a versão nova (não precisa reiniciar)

---

## 10. Só se houver dois ou mais agentes no mesmo número

- [ ] **Agente de IA → Roteadores**
- [ ] Um roteador ativo por número
- [ ] Intenções: nome, agente, descrição na fala do cliente, 3 frases de exemplo
- [ ] Agente de fallback
- [ ] Todos os membros com versão **publicada**
- [ ] Ativo

---

## 11. Memória da organização

- [ ] **Agente de IA → Memória** — regras da casa em texto
- [ ] **Publicar versão** (só admin)

---

## 12. Time e operação

- [ ] **Configurações → Equipe** — convites, papéis (atendente / gerente / admin)
- [ ] Quem assume quando a IA passa
- [ ] Distribuição: manual ou rodízio
- [ ] **Uso e orçamento** — teto mensal de IA
- [ ] Número liberado do modo de teste, se o público real já pode falar com a IA

---

## 12.1 Notificações da equipe

Tela: **Configurações → Notificações** (`/app/settings/notifications`).
É preferência da **pessoa** (e uma regra da **empresa** na tela bloqueada). Não substitui agente nem WhatsApp.

O que a tela tem:

- Cartão do servidor: com as chaves de aviso (VAPID) + https (ou localhost), promete push **com a aba fechada**. Sem as chaves, diz a verdade: aviso só com o site aberto.
- Link **Receber no celular: instalar o app** → `/app/settings/aplicativo` (PWA, não loja).
- **O que os avisos mostram fora do app** (só admin): esconder tudo / só quem mandou / quem mandou e a mensagem.
- Tabela: Nova mensagem, Lead atribuído, Lead ganho, Lead perdido, Você foi mencionado, Ligação recebida.
  - Email: sempre desligado (ainda não existe).
  - In-app: toast com o site aberto.
  - Push: bandeja. O de **Nova mensagem** é o interruptor do aparelho; desligar corta os outros.
- Lembrete de tarefa (cron, site fechado) — cartão separado.
- Sons da Central (etapa que avisa / precisa de pessoa) — gerente+.

Na VPS:

- [ ] Cartão **não** diz “só com o site aberto” (chaves VAPID existem; kit gera no `update.sh`)
- [ ] Endereço é **https** (em `http://IP` o navegador não entrega aba fechada)
- [ ] Cada atendente ligou Push no **próprio** navegador/celular e aceitou a permissão
- [ ] Admin definiu o que aparece na tela bloqueada
- [ ] Quem atende no celular: instalou o app em **Configurações → App no celular**

Não cole chave VAPID neste arquivo. Não troque um par que já existe: desliga os aparelhos já inscritos.

---

## 12.2 Segurança

Tela: **Configurações → Segurança** (`/app/settings/security`).
Conta da **pessoa** + regra da **empresa**. Não instala WhatsApp nem IA.

### Verificação em duas etapas (MFA)

Padrão do produto: **não exigir**. Ativar o próprio fator é opcional.
Quem **já cadastrou** o autenticador prova o código em toda entrada, mesmo se a empresa não exigir.

- [ ] Decidido com o cliente: exigir da equipe ou não
- [ ] Se exigir: **Nível mínimo** (viewer / agent / manager / admin) + **Dias de carência** (0–30) + **Salvar**
- [ ] Cada pessoa obrigada cadastrou o autenticador (Google Authenticator, Authy, etc.)
- [ ] Códigos de recuperação gerados **depois** do MFA e guardados fora do celular (uso único)
- [ ] Sem MFA, o botão de códigos fica: “Habilite MFA antes de gerar códigos” — esperado

Não ligue MFA no primeiro acesso de teste: trava a tela se a pessoa perder o celular sem códigos.

### Chamada de voz pelo WhatsApp

Desligada por padrão. **Não é o QR da Inbox.** Liga um segundo aparelho no mesmo número, fora do caminho oficial. Risco de o WhatsApp **bloquear a conta inteira** (mensagens inclusive).

- [ ] Cliente leu o aviso da tela e aceitou o risco **por escrito**, ou ficou desligado
- [ ] O servidor tem a voz instalada (senão a tela diz: “Este servidor não tem a chamada de voz instalada”)
- [ ] Só então a empresa liga aqui — dá para desligar a qualquer momento; o aparelho cai na hora

Na instalação comum (sem voz no servidor): deixe desligado. Não entra na base.

### Sessões

Listagem de sessões ainda **não existe**. O botão **Sair de todos os dispositivos** encerra os logins na hora (útil se a senha vazou).

- [ ] A pessoa sabe onde desconectar todos os aparelhos

---

## 12.3 Organização (`/app/settings/tenant`) — ficha da empresa

É o cadastro **desta empresa** (o cliente). Cada empresa tem o seu. Não é a
marca da Atlas nem a lista de tenants da VPS.

Não confundir:

| Tela | De quem |
|---|---|
| **Admin → Tenants** | Nosso: criar / suspender / apagar empresas da instalação |
| **Admin → Marca** | Nosso: nome e logo da plataforma |
| **Configurações → Marca** | Cliente: logo/cor **desta** empresa |
| **Configurações → Perfil** | Pessoa logada (avatar, senha) |
| **Esta** | Identidade legal e operação **desta** empresa |

Preencher no cliente real (admin da empresa):

- [ ] Nome de exibição (o que aparece no topo)
- [ ] Razão social (entra no PDF de LGPD como controlador)
- [ ] CNPJ
- [ ] País (BR = LGPD; muda documento do contato e prazos)
- [ ] Fuso (agenda, relatórios, “hoje”)
- [ ] Idioma e moeda
- [ ] E-mail do DPO (encarregado LGPD)
- [ ] URL da política de privacidade, se tiverem
- [ ] Retenção de mídia: só ligar limpeza automática se o cliente pediu (apaga áudio/foto antigos)

Menu lateral e Inbox valem para a empresa inteira. Papel continua mandando no
acesso. Art. 15 (RGPD) só aparece se o país **não** for Brasil.

**Zona de perigo:** apaga mensagens, conversas, negócios, contatos, agenda e
pedidos **desta** empresa. Pede o nome para confirmar. Use para zerar teste
antes do go-live. **Nunca** em empresa já atendendo.

---

## 13. Atualização do sistema (VPS — não é cadastro do cliente)

Esta tela **não** configura o atendimento do cliente. Ela atualiza o **programa** no servidor
(Configurações → Atualização, ou o rodapé “Nova versão”). Fonte:
[github.com/melgarafael/DeskcommCRM](https://github.com/melgarafael/DeskcommCRM).

**Cuidado — o que a atualização NÃO apaga** (está no banco / volume / `.env` da VPS):

- conversas, contatos, funil, tenants, agentes, prompts
- WhatsApp pareado (volume do canal)
- marca (Admin → Marca, no banco — não no código)
- cadastro aberto/convite, SMTP, VAPID, chaves de IA

**Cuidado — o que SE perde** se estiver editado à mão **no clone da VPS**:

- qualquer arquivo do produto (ex.: patch em `lib/…`)
- `config.toml` com porta desta máquina
- `.env` desta máquina de desenvolvimento

O `update.sh` faz **backup antes** (banco + volume do WhatsApp). Se o clone da VPS
tiver arquivo divergente, o comando **para e não toca no banco**.

- [ ] A VPS foi instalada pelo **oficial**, não copiando a pasta do Windows
- [ ] Marca Atlas só pela tela (Admin → Marca), nunca editando código
- [ ] `implantacao/`, `config.toml` local e `.env.local` **não** estão no clone da VPS
- [ ] Estamos numa **VPS** com o kit (não no `localhost` / `pnpm dev`)
- [ ] O agente do servidor fala com o app — a tela mostra a versão e o botão
- [ ] Se a tela disser “indisponível”: no servidor, `bash hostgator-setup-kit/update.sh`
      (primeira vez: **duas vezes** — a primeira baixa o programa, a segunda liga o botão)
- [ ] Quem clica em **Atualizar agora** é o dono da instalação (`platform admin`)
- [ ] Leu o CHANGELOG da versão — só o bloco **⚠️ Requer atenção** pede ação

No computador de desenvolvimento esta tela **fica indisponível de propósito**.
Não rode `update.sh` no Windows. Não entra no pacote do cliente.

---

## 14. Depois (não é base)

- [ ] Skills (`agendamento`, `objecao-preco`) só se for personalizar texto
- [ ] Webhooks / formulário do site
- [ ] Importar planilha: B2B em Importações; lista de leads no Funil
- [ ] Prospecção (`/app/prospecting`) — só com número aquecido, base legal e chave de busca; **não** no WhatsApp de teste

### Prospecção (`/app/prospecting`) — disparo frio, da empresa

Não cria tenant nem usuário. Admin da **empresa** busca empresas (Google Maps via
chave Apify, ou planilha) e manda WhatsApp em campanha (teto 1–50/dia, intervalo
≥5 min). Pede agente, número WORKING, funil e **base legal** (LGPD).

Risco de o WhatsApp **bloquear o número**. Não use o número de teste / número novo.
Não é a base. Não cole a chave Apify neste arquivo.

### Importações (`/app/imports`) — B2B, da empresa

Só existe com o módulo **crm_b2b** ligado (Admin → Módulos). CSV/XLSX até 2 MB.
Colunas reconhecidas: empresa, razão social, nome fantasia, CNPJ, pessoa, cargo,
telefone, e-mail. Cria empresa + decisor + telefone **desta** empresa.

Não é a planilha do **Funil** (CRM → Funis → Importar planilha): aquela põe
lead na primeira etapa. Esta não. Lead de planilha **não** dispara follow-up
automático.

Gerente importa. Clique no arquivo para ver conflitos/falhas por linha.
Lista vazia = ainda não subiu nada. Sem carteira B2B, pule.

### Webhooks (`/app/webhooks`) — da empresa, não do WhatsApp

Não é o webhook do WAHA (esse é da conexão do número). Esta tela é **desta
empresa**: formulário/site entra, automação age, outro sistema é avisado.

| Aba | Faz |
|---|---|
| **Receber dados** | Cria uma fonte, escolhe o funil, copia `…/api/v1/webhooks/in/<token>` ou o HTML do form. HMAC na fonte. |
| **Leads recebidos** | Quem entrou, quem foi recusado e por quê. |
| **Automações** | Quando X (lead ganho, mensagem, agenda…) → fazer Y (WhatsApp, tag, avisar outro sistema…). Costuma nascer **pausada**: ligue o interruptor. |
| **Atividade** | Log do que disparou. |

Gerente ou acima. Cada empresa tem as fontes dela. No `localhost` um site na
internet não alcança o endereço — precisa da VPS (https) ou túnel.

Não cole o token da URL nem o segredo HMAC neste arquivo. Sem site/formulário,
pule.
- [ ] Google Agenda, Meta Ads, Conversões, Nuvemshop — se o cliente anuncia ou tem site
- [ ] Token de API / MCP — só se outro sistema for falar com o CRM
- [ ] Extensões (guias JSON) só se o cliente tiver um roteiro de trabalho para colar no hub do CRM

### Meta Ads (`/app/settings/meta-ads`) — é do cliente, não da instalação

Token de **leitura** da conta de anúncios **desta empresa**. Cada cliente cola o
token **dele** (permissão `ads_read`). Não é compartilhado entre empresas.

Não confundir:

| Tela | De quem | Faz |
|---|---|---|
| **Admin → Meta** (`/admin/meta`) | Nossa instalação (1 para a VPS) | App da Meta da plataforma (WhatsApp Cloud / webhook). Não é anúncio. |
| **Admin → Google** (`/admin/google`) | Nossa instalação | App OAuth do Google da VPS. Sem isto, o botão “Conectar com Google” da empresa falha. |
| **Configurações → Meta Ads** | Cliente | Lê campanhas e mostra em Análise → Meta Ads. Só leitura. |
| **Configurações → Conversões** | Cliente | Avisa Meta/Google quando vendeu (CAPI / Google Ads). Token diferente do de leitura. |

Conta padrão (opcional) é `act_123…`, **não** e-mail. Em branco, abre a primeira
conta que o token alcançar. O token some da tela depois de salvar — nunca cole
em checklist.

Só entra no pacote se o cliente já anuncia. Sem token real, não salve.

### Conversões (`/app/settings/conversoes`) — também do cliente

Uma tela, quatro abas. **Não é** a tela de leitura de campanha.

| Aba | URL | O que é |
|---|---|---|
| Configuração | `/app/settings/conversoes` | Dataset + token CAPI da Meta; regras de etapa; Google Ads da empresa; captura UTM. |
| Histórico de envios | `?aba=historico` | O que foi entregue, falhou ou ficou pendente (e cliques CTWA). |
| Diagnóstico | `?aba=diagnostico` | Saúde da integração com **Google Ads** (cartões OK / atenção / problema). |
| Links rastreáveis | `?aba=links` | Link de WhatsApp por campanha + script do site, com contagem de cliques. |

Gatilho: negócio marcado como **ganho** (e regras de etapa, se ligadas). Sem anúncio
e sem Pixel/dataset, a aba Configuração fica vazia de propósito — não preencha.

### Extensões (`/app/extensions`) — o que é e o que não é

Não é App Store. Não executa código de terceiro nem lê o banco. O JSON (até 512 KiB) publica
**cards de orientação** no hub do CRM, cada um com um botão para uma tela que **já existe**:
Tarefas, Conversas, Funil, Contatos, Agenda, Radar.

Quem admite o arquivo é o dono da **instalação**. Quem **ativa** é o admin da **empresa**.
Instalar ≠ autorizar: o clique reconfere o acesso na hora.

Não faz hoje: tela nova, menu novo, tabela, ferramenta de IA, preço no prompt, marketplace.

Os módulos **Honorários / Comandas / Financeiro** (Admin → Sistema) são outra coisa: oficiais
da plataforma, não esse JSON. Aparecem em “Atividade recente” quando a instalação os provisiona.

Para melhorar processo: escrever o roteiro do nicho (ex.: “quem tratou e não remarcou → Abrir
conversas”) num pacote JSON revisado, admitir, a empresa ativa. Sem roteiro escrito, não instale
arquivo de fonte desconhecida — o JSON não prova quem publicou.

---

## Entrega

Medido na tela, não na conversa:

- [ ] Agente publicado no número certo
- [ ] Roteador ativo (se houver 2+ agentes), membros publicados
- [ ] Follow-ups publicados **e** ligados no agente
- [ ] Base de conhecimento com status pronto
- [ ] Memória publicada
- [ ] Um teste de cada situação do roteiro ok
- [ ] A pessoa sabe onde muda prompt, FAQ, horário e teto de IA

O que ficou de fora (e o efeito enquanto falta):

-
-

---

## Não fazer

- Não cadastrar por SQL / banco — o motor lê a **versão publicada** na tela
- Não publicar sem a pessoa ver o teste
- Não pôr preço no prompt se o preço está no conhecimento
- Não ligar capacidade crítica sem explicar o que o agente passa a fazer sozinho
- Não colar segredo neste arquivo
