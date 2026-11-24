# 05 — Implantar um cliente

Cliente novo **não** se configura em Deskcomm Local. Deskcomm Local é o
laboratório. Cliente = **Administração → Tenants → Novo tenant**.

E-mail do responsável = e-mail **do cliente**, nunca o seu. Senão a
empresa vira sua e o convite não sai.

Modelo reutilizável: [checklist-cliente.md](../checklist-cliente.md).
Copie para `pacote-<slug>.md`. Nunca cole chave no arquivo.

Pacote do lab: [pacote-deskcomm-local.md](../pacote-deskcomm-local.md).

## Ordem que o sistema impõe

Cada bloco exige o anterior. Publicar agente **recusa** sem número
WORKING e sem credencial validada.

1. **Tenant** — nome, razão, CNPJ, e-mail do dono, plano (ou isento).
   Copiar convite (24 h). Sem SMTP, manda o link no WhatsApp.
2. **Marca da empresa** — Configurações → Marca (a da clínica; a Atlas
   já está em `/admin/marca`).
3. **Equipe** — convites, papéis, quem assume quando a IA passa.
4. **Canais → Conexões** — QR até WORKING. Modo teste: lista de
   confiança. Proteção de envio. Grupos só se forem usar.
5. **IA → Credenciais** — chave **na tela**, validar. Se o atendimento
   não for OpenAI, chave OpenAI à parte para áudio e base.
6. **IA → Provedores** — atendimento = modelo com ferramentas;
   roteador/follow-up = modelo barato (ou Automático).
7. **Funil** — etapas, uma ganhou, uma perdeu, passo do agente,
   vocabulário, motivos de perda, funil ativo.
8. **Tipos de agendamento** — se o negócio marca horário.
9. **Conhecimento** — 10–30 FAQ; PDF se houver. Status **pronto**.
10. **Follow-ups** — clínica: **Começar de um modelo**. Outros: silêncio
    na etapa que abandona. **Publicar** o fluxo.
11. **Agente** — prompt do nicho (esqueleto em
    `.agents/skills/deskcomm-cliente-novo/references/prompt-do-agente.md`).
    Número, funis, materiais, **follow-ups que arma**, horário,
    pacote Vender se for o caso. Salvar **rascunho**.
12. **Testar** — 5 frases do nicho. A pessoa vê.
13. **Publicar** — a **pessoa** clica, ou manda por escrito.
14. **Roteador** — só se houver 2+ agentes no mesmo número.
15. **Memória** — regras da casa. Publicar versão (admin).
16. **Webhooks** — se tiver site/form. Fonte + funil + HMAC.
17. **Token** — se outro sistema (Atlas loja, n8n) for falar com o CRM.
    Copiar plaintext **uma vez**. Escopo mínimo (`mcp:write` para envio).
18. **Uso e orçamento** — teto mensal de IA.
19. **Abrir o número** ao público só quando a lista de teste não basta.

## Prompt: o que não escrever

O motor já impõe: assistente virtual, não inventar preço, STOP, janela,
agenda só com ferramenta. Repetir isso no prompt gasta contexto e pode
**vetar** a resposta.

Não cite `lead_id`, etapa `qualified`, nome de ferramenta.
Não “manda tudo pro Fulano” — a IA para de usar a agenda.
Preço vive na **base**, não no texto.

## Dois produtos no mesmo cliente

Se o cliente também leva o **Atlas loja**:

1. CRM no ar, número WORKING.
2. Token `mcp:write` na empresa **dele**.
3. Na loja: Perfil → WhatsApp → site do CRM + token + link público da loja.
4. Fila: Chamar / Faltou / Avisar mandam se a senha tem celular.

Não misture o token da Deskcomm Local com o do cliente.

## Critério de “pronto”

- Agente publicado no número certo
- Follow-ups publicados **e** ligados no agente
- Base com status pronto (ou anotado o que ficou de fora)
- Memória publicada
- Um teste de cada frase do roteiro ok
- A pessoa sabe onde muda prompt, FAQ, horário e teto
- Ela sabe abrir a **Fila** e devolver ao automático

## Tempo (ordem de grandeza, não SLA)

Triagem 1 h. Conexão + funil + FAQ 2–4 h. Agente + teste 2–4 h.
Nicho com quatro follow-ups de clínica: mais uma tarde para ler os
textos na voz da casa. Integração (token/webhook): à parte, com o
sistema de fora no ar.
