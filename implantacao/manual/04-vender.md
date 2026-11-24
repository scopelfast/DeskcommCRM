# 04 — Divulgar e vender

Fale a língua de quem paga. Não venda “MCP” nem “RLS”. Venda: o cliente
fala no WhatsApp, a IA atende com o dado da empresa, o funil anda, você
vê tudo, os dados ficam na sua operação.

## Pitch em 20 segundos

> O Atlas CRM é o comercial de mesa: WhatsApp, funil e um atendente de
> IA que **trabalha** — agenda, qualifica, cobra quem sumiu. Instalado
> na sua VPS (ou na nossa). Sem mensalidade do fabricante. Você é dono
> do sistema.

Âncora: “alternativa a Kommo / Octadesk, com o código e os dados seus”.

## O que mostrar na demo (ordem)

1. Inbox com conversa real (lab: já funciona na mão).
2. Funil com o vocabulário **do nicho** (paciente, não lead).
3. Agenda: marcar na conversa (quando o agente estiver publicado).
4. Follow-up: “quem não responde em 24 h recebe isto”.
5. Radar: quem está esfriando.
6. Só se o cliente tiver site: Webhooks → form vira card.
7. Só se for integrar loja/ERP: um token, uma frase — “o outro sistema
   puxa; o CRM também avisa”.

Não comece por Credenciais, API Tokens ou `/admin`.

## Pacotes comerciais (sugestão Atlas)

O software não trava feature. **Você** empacota o serviço:

| Oferta | Entrega | Não inclui |
|---|---|---|
| **Atender** | VPS (ou acesso), Inbox, 1 número, equipe, funil básico | Agente publicado |
| **Atender + IA** | Credencial, 1 agente, base FAQ, follow-up de silêncio | Roteador, B2B, cobrança in-app |
| **Operar** | Tudo acima + funil do nicho + modelos de clínica/loja + treino | Módulos que o cliente não pediu |
| **Integrar** | Token e/ou webhook para o sistema dele (Atlas loja, n8n, site) | Reescrever o ERP |

Módulos (`crm_b2b`, propostas, comandas, cold call, voz) são **add-on
seu**, porque na instalação padrão eles nem aparecem.

## Por nicho — uma frase cada

| Nicho | A frase | Follow-up que fecha a venda |
|---|---|---|
| Clínica | A recepção não deixa consulta morrer no WhatsApp. | Modelos prontos: consulta, exame, cirurgia, falta |
| Imobiliária | Quem pediu imóvel e sumiu volta sozinho. | Silêncio na etapa “sei o que oferecer” |
| Serviços | Orçamento enviado cobra sozinho. | 3 dias na etapa de orçamento |
| Curso | Quem pediu o link e não entrou leva o link de novo. | 24 h em “quer entrar” |
| Loja | Carrinho/pagamento parado ganha lembrete no ritmo do número. | 2 h / 24 h em “aguardando pagamento” |
| Advocacia | Entende o caso e **não** promete êxito. | Silêncio + caso para urgência |

Preço, convênio, política: **base de conhecimento**, não prompt. Prompt
que inventa preço é o motor vetando na cara do cliente.

## Cadastro aberto (Instagram)

Decisão Atlas neste lab: **Administração → Cadastro** com os dois
interruptores **desligados** = modo aberto. `/signup` cria conta e
empresa na hora.

Antes de divulgar o link da **VPS**:

1. **Administração → Marca** (Atlas).
2. **Administração → E-mail** (senão confirmação e “esqueci senha” não
   chegam) + `update.sh` **na VPS**, não neste Windows.
3. HTTPS no domínio.
4. Se for cobrar pelo sistema: plano com **Usar no cadastro**.

Convite (Tenants / Equipe) continua valendo para cliente que você monta
à mão.

## Objeções

| Eles dizem | Você diz |
|---|---|
| “Já tenho WhatsApp Business” | O app continua. O CRM é a mesa: funil, IA, equipe, histórico. |
| “Tenho medo de ban” | Ritmo, janela, aquecimento e modo teste são da Conexão. Número novo não dispara campanha no dia 1. |
| “A IA vai falar besteira” | Publica depois do Teste. Base de conhecimento. Humano na Fila. Teto de gasto. |
| “Meus dados na nuvem de vocês?” | Self-host. VPS sua ou nossa, isolada por empresa. LGPD na tela. |
| “Integra com meu sistema?” | Token (puxa) e webhook (avisa). Contrato em [token-para-outros-sistemas.md](../token-para-outros-sistemas.md). |
| “É o Atlas da loja?” | São dois produtos. A loja manda WhatsApp **por** este CRM. |

## O que não vender neste lab

- Chamada de voz / WaCalls / SIP.
- Prospecção no WhatsApp de teste.
- “Atualização automática” no localhost.
- Que o form do site na internet bate em `localhost:3020`.

## Material oficial de apoio (não é Atlas)

- Posicionamento: `VISION.md`
- SaaS / planos: `docs/saas/README.md` e `docs/saas/cobranca.md`
- Contrato/LGPD com o cliente final: `docs/saas/contrato-e-lgpd.md`
- White-label: `docs/white-label.md`
