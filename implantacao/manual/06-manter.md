# 06 — Manutenção

Duas máquinas, duas regras.

## Este Windows (`c:\DeskcommCRM`)

É **laboratório**. Não é git. Não é a VPS.

- Código e banco do lab: **v1.126.0** (oficial, 25/11/2026). Não entra sozinho da próxima vez.
- CRM: `http://localhost:3020`
- Não rode `update.sh` aqui.
- Não copie esta pasta para a VPS nem para o GitHub oficial.
- Não commite `.env.local`, `supabase/config.toml` (portas 5532x por
  causa do Fleetbase), `implantacao/`, patch local de mídia.
- Tela **Atualização automática indisponível** = esperado.
- MFA: padrão desligado. Não exigir no teste.
- Chamada de voz: **não ligar**.
- Cadastro: aberto (decisão Atlas / Instagram).
- Número de teste: Inbox na mão ok. Não prospectar.

Subir o CRM neste lab: o comando combinado na sessão (PowerShell +
`pnpm`). Não misturar a porta 5432 do Fleetbase.

## A VPS de produção

Fonte: clone **oficial** + kit (`hostgator-setup-kit` / Coolify).
Runbook: `docs/runbooks/deploy.md` e skill `deskcomm-instalar`.

Atualizar:

1. Ler `CHANGELOG.md` → bloco **Requer atenção**.
2. Backup do banco.
3. `update.sh` **na VPS** (o kit puxa imagem; não constrói na máquina
   do cliente).
4. Conferir `/` = 200 e `/app` = 307 (login), não 404.

Se a VPS já tem Traefik/Coolify, o `up -d` leva **os dois** composes
(senão o domínio dá 404 com o contêiner “healthy”).

Marca Atlas vive **no banco** (`/admin/marca`), não em variável
`NEXT_PUBLIC_*`. Uma imagem serve todas as marcas.

## O que quebra e onde olhar

| Sintoma | Onde |
|---|---|
| WhatsApp não conecta | Canais → Conexões. QR / status. Não é a tela de Webhooks. |
| IA não responde | Agente publicado? Número WORKING? Credencial validada? Canal autoriza IA? Modo teste? |
| Follow-up mudo | Publicado? Ligado no agente? Relógio/cron no ar? |
| Form do site não vira lead | Fonte em Webhooks → Receber dados. HMAC. URL pública da **VPS**. Aba Leads recebidos diz o porquê. |
| Token recusado | Prefixo `dsk_`. Escopo. Não revogado. Header, nunca query. |
| “Não consigo copiar o token” | Não dá. Gera outro, cola, revoga o velho. |
| Empresa A vê dado de B | Não deve. Pare e trate como incidente. Isolamento é invariante. |
| Domínio 404, container healthy | Faltou o compose do Traefik. |
| Confirmação de e-mail não chega | `/admin/email` + `update.sh` na VPS. |
| Quem se cadastra não tem plano | Plano com **Usar no cadastro**. |

## Integrações — manutenção

- Token: plaintext uma vez; no banco só hash. Rotação = criar + revogar.
- Webhook de saída: o outro sistema confere HMAC
  (`docs/integracao/webhooks-de-saida.md`). Reenviar na aba Atividade.
- URL interna (192.168, 10.x) na ação de webhook: o CRM **bloqueia**
  (SSRF), salvo destinos liberados em `/admin/destinos-internos`.

## Custo que você acompanha

- **IA:** Agente de IA → Uso e orçamento (por empresa) e
  `/admin/usage` (instalação).
- **WhatsApp:** teto e ritmo na Conexão. Campanha no ritmo mais lento.
- **VPS:** uma instalação, N empresas. 8 GB se CRM + banco + painel
  na mesma máquina (`docs/saas/coolify.md`).

## LGPD

Pedidos na empresa: Configurações → LGPD. Anonimizar preferível a
apagar. PDF de LGPD **não leva marca Atlas** — nomeia o **controlador**
(razão social da empresa). Você (revendedor) é operador.

## Quando atualizar o GitHub oficial

O original evolui. Esta pasta **não** recebe isso sozinha.

Produção: `git` do oficial na VPS + `update.sh`.
Lab Windows: só se você **decidir** puxar um clone novo, sem misturar
`implantacao/` nem `.env`.

## Checklist mensal (dono da instalação)

- [ ] CHANGELOG da versão atual vs a da VPS
- [ ] `/admin/usage` — alguma empresa estourando IA
- [ ] Backup do Postgres
- [ ] Certificado HTTPS
- [ ] Números: algum em estado ruim na Conexão
- [ ] Cadastro ainda no modo que você quer (aberto / aprovação / convite)
- [ ] SMTP ainda testa ok

## Onde estudar depois deste manual

| Quero | Abra |
|---|---|
| Afinar prompt com conversa real | skill `deskcomm-prompt` |
| Métrica / “a IA vende?” | skill `deskcomm-metricas` |
| VPS quebrada | skill `deskcomm-instalar` |
| Cobrar cliente | `docs/saas/cobranca.md` |
| Extensão de nicho em vez de fork | skill `deskcomm-extensao` |
| O produto na boca do cliente | `/app/ajuda` logado na empresa |
