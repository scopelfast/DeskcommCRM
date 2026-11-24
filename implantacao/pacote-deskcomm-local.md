# Pacote — Deskcomm Local (teste nesta máquina)

Cópia de trabalho. O modelo reutilizável é `checklist-cliente.md`.
Para o próximo cliente: copie o modelo, não este arquivo.

Nunca cole chave de IA, senha ou token aqui.

Nicho (marque um):

- [ ] Clínica / consultório / salão
- [ ] Imobiliária
- [ ] Serviços / agência / obra
- [ ] Curso / mentoria / infoproduto
- [ ] Loja
- [ ] Escritório de advocacia
- [ ] Outro: _______________

Cliente: Atlas CRM (teste local)
Empresa no CRM: Deskcomm Local
Implantador: Scope
Início: 05/11/2026
Go-live: ____/____/________

Este arquivo é o laboratório **Deskcomm Local**. Cliente novo **não** se cria aqui:
Administração → Tenants → Novo tenant (`/admin/tenants/new`). E-mail do responsável
= do cliente, nunca `admin@local.test`. Depois troca no seletor do topo e copia o
checklist em `pacote-<slug>.md`.

## Cadastro da instalação (05/11/2026)

Tela: `http://localhost:3000/admin/cadastro` — é **nossa**, vale para a VPS inteira.

**Como deixar (Atlas + divulgação Instagram):** **aberto**
- [x] **Cadastro apenas por convite = Desligado**
- [x] **Cadastro com aprovação = Desligado**

Quem acessa `/signup` cria conta e empresa na hora. É o que a tela mostra agora.
Link na VPS: `https://SEU_DOMINIO/signup`.

Aprovação = filtro de spam (a pessoa espera). Convite = trava o anúncio.
Cliente que você implantar à mão continua podendo nascer em Tenants → Novo tenant.

Na VPS: Administração → E-mail (SMTP) antes de divulgar, senão a confirmação
de conta não chega. Cobrança in-app é opcional (plano com “Usar no cadastro”).
Não rodar `update.sh` neste Windows.

## Cuidado: atualização do GitHub oficial (05/11/2026)

Esta pasta (`c:\DeskcommCRM`) é **laboratório**. Não é git. Não é a VPS.
Atualização do [oficial](https://github.com/melgarafael/DeskcommCRM) **não
entra sozinha aqui** e **não apaga** o que está no banco da VPS.

**Lab atualizado em 24/11/2026 para v1.124.0** (código oficial + 40
migrations novas no Postgres `55322`). Conservado: `.env.local`, `config.toml`
(portas 5532x), `implantacao/`, patch de mídia + `ALLOWED_DEV_ORIGINS`.
**Não** foi `update.sh`. Spare do clone: `C:\DeskcommCRM-oficial-1.124`
(pode apagar; 1.114 e anteriores também). Reinicie o `pnpm dev` nesta pasta para o 3020 pegar a versão.

Instalar produção = clone oficial na VPS + marca pela tela. **Nunca** copiar
esta pasta do Windows por cima, nem mandar para o original:

- `implantacao/` (notas Atlas)
- `supabase/config.toml` (portas `5532x` por causa do Fleetbase)
- `.env.local` / qualquer chave
- patch local de mídia (`url-vista-do-container`, `waha.ts`, `client.ts`) —
  na VPS com https o Storage já é público; o lab é que precisava do patch

O que **fica** numa atualização da VPS: conversas, tenants, WhatsApp pareado,
marca no banco, SMTP, cadastro, agentes. O que a atualização troca é o
**programa**. Se o clone da VPS tiver arquivo mexido à mão, o `update.sh`
**recusa e não toca no banco**.

Tela `/app/settings/atualizacao` neste localhost = **Atualização automática
indisponível** / versão “Ainda não confirmada”. Esperado. Não “consertar”.

## Destinos internos (05/11/2026)

Tela: `/admin/destinos-internos` — **nossa** (rede da VPS). Lista vazia = o
sistema **não** fala com IP interno (10.x, 192.168.x). Só preencha se a
instalação apontar transcrição/IA para uma máquina na mesma rede. Empresa
cliente **não** consegue usar isso para apontar para dentro. Pular na base e
no Instagram. Não salvar `0.0.0.0/0`.

## E-mail da instalação (05/11/2026)

Tela: `/admin/email` — **nossa**. Um servidor manda o e-mail de **todas** as
empresas desta VPS (convite, confirmação de conta, esqueci a senha, LGPD,
aviso de prazo). Cliente **não** tem SMTP próprio.

**Laboratório (Gmail, 05/11/2026):** remetente de teste `scopelcursor@gmail.com`.
Senha **não** vai no chat nem neste arquivo — cola **na tela**. No Gmail é
**senha de app** (conta Google → Segurança → Verificação em duas etapas ligada
→ Senhas de app), não a senha normal.

Campos:
- Endereço do servidor: `smtp.gmail.com`
- Porta: `587`
- Segurança: STARTTLS
- Usuário: `scopelcursor@gmail.com`
- Senha: a de app
- Remetente: `scopelcursor@gmail.com`
- Nome: `Atlas CRM`
- **Salvar** primeiro, depois **Testar conexão**. Bloco Resend vazio.

No localhost isso manda convite / LGPD / aviso do **app**. Confirmação de
cadastro e “esqueci a senha” **locais** continuam no Supabase da máquina
(não herda este SMTP sem `update.sh` — e `update.sh` não roda neste Windows).

**VPS (antes do Instagram):** trocar por SMTP do domínio Atlas (ou Resend com
domínio verificado). Gmail estoura limite e cai em spam se o anúncio gerar
muito cadastro. Depois `update.sh` na VPS.

## Platform Admins (05/11/2026)

Tela: `/admin/platform-admins` — **nossa**, só leitura. Quem entra em
`/admin` (Tenants, Cadastro, E-mail, Marca). **Não** é o admin da empresa
cliente.

Linha atual correta: `admin@local.test` / C. Scopel, scope `full`, MFA Não,
Ativo, bootstrap. Quem se cadastra pelo Instagram **não** entra nesta lista.

Não há botão de incluir/tirar (por desenho). Só SQL no banco, se um dia
precisar de um segundo dono da VPS. MFA da política da instalação continua
em Configurações → Segurança; “Não” aqui = a linha não **obriga** MFA.

## Usuários da instalação (05/11/2026)

Tela: `/admin/users` — **nossa**, só lista. Todas as pessoas de **todas** as
empresas. Filtro por empresa/papel. **Não cadastra.** Clique na linha = ficha
(em quais empresas está).

Cadastrar gente da **empresa aberta no seletor**:
`http://localhost:3000/app/team` → **Convidar membros** → e-mails + papel
(Somente leitura / Atendente / Gerente / Administrador). Sem SMTP: **Copiar
link**. Precisa ser admin daquela empresa.

Empresa cliente nova: Tenants → Novo tenant (e-mail do responsável).
Instagram: a pessoa cria a própria conta em `/signup` e aparece nesta lista
depois.

## LGPD da instalação (05/11/2026)

Tela: `/admin/lgpd` — **nossa**, relógio. Pedidos de **todas** as empresas
(exportar dados D+7, anonimizar contato D+15, anonimizar empresa). Faixa
vermelha = prazo < 24h ou vencido. Clique na linha = prazo e auditoria.
**Não aprova** daqui.

Quem aprova/entrega: a empresa, em `/app/lgpd/requests`.
Controladora = a empresa-cliente (razão social + DPO em Organização).
Atlas = operador. PDF **não** leva marca Atlas.

Lista vazia agora = certo. DPO da instalação (fallback) em Administração →
Configuração. DPO de cada cliente no cadastro da empresa — senão o titular
escreve para vocês.

## Incidentes (05/11/2026)

Tela: `/admin/incidents` — **nossa**. Quadro de panes da VPS (WhatsApp
caiu, fila travada, webhook falhando). Filtro começa em **Abertos**. Lista
vazia **é o normal**: o produto hoje **não grava** linha aqui. O aviso real
(número desconectou, IA sem saldo, mensagem presa) vai para a empresa em
**IA → Alertas** (`/app/ai/inbox`).

Não invente incidente. Se um dia aparecer: abrir → **Resolver** (nota ≥ 10
letras). Não dá para desfazer. Filtro “Reconhecidos” não tem botão.

## Mídia no Inbox (05/11/2026)

Texto, áudio e anexo **saem** neste lab (provado na tela). A falha anterior
era o WAHA (Docker) baixando o arquivo em `127.0.0.1:55321` — lá isso é o
próprio WAHA, não o Storage. O envio troca loopback por
`host.docker.internal`.

WhatsApp não envia `.ico` como foto: use PNG ou JPG. Áudio do microfone
(webm) o WAHA converte.

Na VPS isso já funcionava: o Storage é https público. Não misturar com
**Conexões → Telefone** (tronco SIP) nem **Chamada de voz** (ligação).

---

## 0. Triagem (preencher antes de cadastrar)

### Negócio

- Como se chama e o que faz (uma frase):
- Cidade / fuso:
- Cliente típico e o que pede primeiro:

### Canal

- Número de WhatsApp (já conectado?): sim — QR, WORKING (05/11/2026)
- Horário em que a IA responde (dias, início, fim):
- Número novo? (sim = aquecimento / limite baixo no começo): sim (teste)

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

- Quem assume quando a IA passa (e-mail e papel): dono local (admin@local.test)
- Distribuição: manual ou rodízio: manual (por enquanto)

---

## 1. Canal

- [x] **Canais → Conexões** — número por QR com status WORKING
- [ ] Número saiu do **modo de teste** (ou a lista de telefones autorizados cobre o teste)
- [ ] Proteção de envio conferida no próprio número (janela, ritmo, aquecimento)
- [x] Teste: mensagem de um celular → aparece em **Atendimento → Inbox**
- [x] Teste: resposta pela Inbox → chega no celular

Sem WORKING, não publique agente.

Notas: Inbox na mão ok (Cleverson). Conversa assume na Fila. IA ainda não atende.

---

## 2. Credenciais de IA

- [ ] **Agente de IA → Credenciais → Adicionar**
- [ ] Provedor, nome (“Chave principal”) e chave cadastrados **na tela** (não neste arquivo)
- [ ] Credencial **validada** (só validada publica agente)
- [ ] Se o atendimento não for OpenAI: chave OpenAI cadastrada para áudio e base de conhecimento

**Próximo passo desta instalação.**

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

---

## 5. Conhecimento

- [ ] **Agente de IA → Conhecimento → Adicionar material**
- [ ] FAQ (pares pergunta/resposta) — 10 a 30
- [ ] Documentos (PDF, MD ou TXT) — preço, política, catálogo, se houver
- [ ] Status **pronto** (indexação terminou)
- [ ] Sem chave OpenAI: anotado o que fica de fora (áudio + documentos)

---

## 6. Follow-ups (depois do funil; antes ou junto do agente)

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

Tela conferida em 05/11/2026: `http://localhost:3000/app/settings/notifications`.

- [x] Tela existe e o comportamento bate com o produto
- [x] Em 05/11/2026 a faixa amarela estava certa: VAPID **ausente**
- [x] Par VAPID gerado neste `.env.local` (valores só no ambiente, não neste arquivo)
      Recarregar o app para a faixa mudar: Push com a aba fechada neste localhost
- [ ] Recarregar **Configurações → Notificações** e conferir que a faixa **não** pede mais `update.sh`
- [ ] Ligar Push de Nova mensagem neste Chrome e aceitar a permissão do navegador
- [ ] Na VPS: cartão some / muda para “Push já funciona com a aba fechada” + https
- [ ] Cada pessoa liga Push no próprio aparelho
- [ ] App no celular = PWA (`/app/settings/aplicativo`), não loja. No localhost sem https
      de outro IP da rede, o celular não instala.

Acesso pelo IP da LAN (`http://192.168.100.5:3000`, 05/11/2026): o Next bloqueava
`/_next/hmr` (console vermelho, handshake inválido). Liberar com
`ALLOWED_DEV_ORIGINS` no `.env.local` (não no GitHub). “Não seguro” no Chrome
é HTTP, esperado. Cookie de `localhost` **não** vale nesse IP — entrar de novo.
No celular, **Entrando…** / Rendering: o login chega no servidor, mas o
Next precisa aceitar a Server Action vinda do IP (`serverActions.allowedOrigins`
via `ALLOWED_DEV_ORIGINS`). Recarregar depois do restart. Não usar Google
nesse endereço. Teste de WhatsApp = app no celular + Inbox no PC.

Email na tabela permanece desligado — o produto ainda não envia aviso por e-mail.

Não rode `update.sh` no Windows. O par local já foi gerado. Na VPS o kit cria o par
sozinho; não gere outro se um já existir.

---

## 12.2 Segurança

Tela conferida em 05/11/2026: `http://localhost:3000/app/settings/security`.

- [x] MFA **Desativada** — padrão correto; não exigir de ninguém; carência 0
- [x] Códigos de recuperação: “Habilite MFA antes” — esperado sem autenticador
- [x] Chamada de voz: desligada + “este servidor não tem a chamada de voz instalada” — esperado (sem WaCalls no localhost)
- [x] Sessões: listagem “em breve”; botão Sair de todos existe
- [ ] MFA da conta local: só se formos testar o fluxo (guardar os códigos no mesmo instante)
- [ ] Exigir da equipe: não nesta máquina de teste
- [ ] Chamada de voz: **não ligar** neste teste (risco de bloqueio do WhatsApp + servidor sem o serviço)

---

## 12.3 Organização

Tela conferida em 05/11/2026: `http://localhost:3000/app/settings/tenant`.

- [x] É da empresa **Deskcomm Local**, não da instalação Atlas
- [ ] Nome/razão/CNPJ reais: só quando for um cliente de verdade (neste teste pode ficar Deskcomm Local)
- [ ] País BR / fuso America/Sao_Paulo — conferir se a tela já está assim
- [ ] DPO e política de privacidade: pular neste teste
- [ ] Limpeza automática de mídia: **deixar desligada**
- [ ] Zona de perigo: **não clicar** enquanto houver conversa de teste que queremos guardar

Admin → Tenants é a lista **nossa** de empresas. Esta tela é a ficha **deles**.

---

## 13. Atualização do sistema (VPS — não é cadastro do cliente)

Detalhe e a regra do que se perde / não se perde: seção **Cuidado: atualização
do GitHub oficial** no topo deste arquivo. Modelo reutilizável: checklist §13.

- [x] Estamos no **localhost** (`pnpm dev`) — tela **Atualização automática indisponível** é o esperado
- [x] Registrado: não copiar esta pasta para a VPS; marca só no banco; `config.toml` local não sobe
- [ ] Conferir de novo quando a primeira VPS de produção estiver no ar
- [ ] Na VPS: se indisponível, `bash hostgator-setup-kit/update.sh` duas vezes na primeira vez
- [ ] Botão **Atualizar agora** só para o dono da instalação
- [ ] Antes de atualizar a VPS: CHANGELOG → bloco **⚠️ Requer atenção**

Notas (05/11/2026): `http://localhost:3000/app/settings/atualizacao` mostrou
“Versão instalada: Ainda não confirmada” / “Execução não confirmada”. Correto:
não há agente do servidor nesta máquina. Não tentar consertar com o comando da tela aqui.

---

## 14. Depois (não é base)

- [ ] Skills (`agendamento`, `objecao-preco`) só se for personalizar texto
- [ ] Webhooks / formulário do site
- [ ] Google Agenda, Meta Ads, Conversões, Nuvemshop — se o cliente anuncia ou tem site
- [ ] Token de API / MCP — só se outro sistema for falar com o CRM
- [ ] Extensões JSON: só com roteiro de nicho escrito (não é a base)

Notas (05/11/2026): `/app/settings/meta-ads` — empresa Deskcomm Local. “Conta
padrão” veio com `admin@local.test` (e-mail, não `act_…`). Não salvar. Sem
token `ads_read`, pular.

Notas (05/11/2026): `/app/settings/conversoes` e as abas `historico` /
`diagnostico` / `links` — mesma tela, **do cliente**. Sem Pixel/dataset e sem
anúncio, não preencher. Histórico vazio é o esperado neste teste. Google Ads
da empresa só conecta se a VPS tiver Admin → Google.

Notas (05/11/2026): `/app/extensions` — nenhuma guia JSON instalada (esperado).
Atividade recente mostrou Honorários e Financeiro (14:18) — módulos oficiais
de Admin → Sistema, não o catálogo JSON. Chamada de voz e MFA desligados.

---

## Entrega

- [ ] Agente publicado no número certo
- [ ] Roteador ativo (se houver 2+ agentes), membros publicados
- [ ] Follow-ups publicados **e** ligados no agente
- [ ] Base de conhecimento com status pronto
- [ ] Memória publicada
- [ ] Um teste de cada situação do roteiro ok
- [ ] A pessoa sabe onde muda prompt, FAQ, horário e teto de IA

O que ficou de fora (e o efeito enquanto falta):

- Credencial de IA — Fila fica humana; agente não publica
- Modo de teste ainda ligado — IA não responde a número fora da lista autorizada

---

## API Tokens (11/11/2026)

Tela: Organização → API Tokens (`/app/settings/api-tokens`).
Estudo dos escopos: `implantacao/api-tokens.md`.
Contrato: `implantacao/token-para-outros-sistemas.md`.
Exemplos de fora (um de cada): `implantacao/token-exemplos-fora.md`.
Nunca cole `dsk_…` aqui.

---

## Não fazer

- Não cadastrar por SQL / banco — o motor lê a **versão publicada** na tela
- Não publicar sem a pessoa ver o teste
- Não pôr preço no prompt se o preço está no conhecimento
- Não ligar capacidade crítica sem explicar o que o agente passa a fazer sozinho
- Não colar segredo neste arquivo
