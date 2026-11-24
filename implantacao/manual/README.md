# Manual Atlas CRM

Manual **nosso** para quem vai **divulgar, vender, implantar e dar
manutenção**. Escrito em 11/11/2026 a partir do código e das telas deste
laboratório (`c:\DeskcommCRM`), não da memória.

Nunca cole senha, token `dsk_…` ou chave de IA nestes arquivos.

## Já existia documentação?

Sim — e não substitui este manual.

| Onde | Para quem | O que é |
|---|---|---|
| `VISION.md` | posicionamento do open source | O que o Deskcomm **é** |
| `docs/saas/` | dono de VPS que revende | Planos, Stripe/Asaas, contrato |
| `.agents/skills/deskcomm-operacao` | quem já usa | “Onde fica esta tela?” |
| `.agents/skills/deskcomm-cliente-novo` | implantador | Montar agente por nicho |
| `.agents/skills/deskcomm-instalar` | técnico | VPS, WhatsApp, update |
| `/app/ajuda` **no próprio CRM** | o cliente final | Caminho de cada tarefa |
| Curso em `curso/` | aulas visuais oficiais | Envelhece se o produto mudar |

O que **não** existia: um texto Atlas, em português de venda e operação,
com as duas camadas (instalação × empresa), o mapa das telas, o que
prometer e o que **não** ligar no lab.

Este manual é isso. Quando discordar da tela, **a tela manda**.

## Como ler

| Papel | Capítulos |
|---|---|
| Aprender o produto | [01](01-o-produto.md) → [02](02-como-funciona.md) → [03](03-mapa-de-telas.md) |
| Divulgar / vender | [01](01-o-produto.md) + [04](04-vender.md) |
| Implantar um cliente | [05](05-implantar.md) + [checklist](../checklist-cliente.md) |
| Manter VPS / lab | [06](06-manter.md) |
| O que veio de novo (lab **1.124.0**) | [07](07-o-que-e-novo.md) |

Lab local neste Windows: CRM em `http://localhost:3020` (não 3000), código **v1.124.0** (24/11/2026).
Empresa de teste: **Deskcomm Local**. Cadastro da instalação: **aberto**.

## Duas camadas (grave isto)

1. **Instalação** (`/admin`) — a VPS, a marca Atlas, o cadastro, o e-mail,
   os módulos, as empresas-clientes. Só o dono da instalação.
2. **Empresa** (`/app`) — Inbox, funil, agente, WhatsApp **desta** clínica
   ou loja. Os dados de uma empresa não vazam para outra.

Cliente novo = **empresa nova** no painel, não uma VPS nova. Uma VPS por
cliente só quando ele exige login e marca só dele, ou não pode cair junto
com os outros.

## O que este CRM não é

- Não é o **Atlas** da loja (`atlas-go` na porta 3010). Aquilo é outro
  produto (fila, senha, agenda da loja). Fala com o CRM por token.
- Não é chatbot enfeite. O agente **opera**: responde, agenda, move funil,
  dispara follow-up — com humano no comando.
- Não tem versão paga do software. MIT. Quem instala cobra o cliente
  **dele** (planos no painel, Stripe ou Asaas), se quiser.
