# Funil Shark v2 — biomédica injetora

Formulário de captação e qualificação de leads da GreenHub para a campanha de
**tráfego pago para biomédica injetora** (toxina botulínica / pacientes premium).

Página única, sem build: `index.html` é o arquivo que vai pro ar.

## Fluxo

```
capa → nome → @ do Instagram → WhatsApp → faturamento → investimento → agendamento
```

A pessoa é classificada em três níveis:

| Nível | Regra | Para onde vai |
|---|---|---|
| **A** | fatura acima de 30k **e** toparia investir ~R$ 1.000/mês | tela de agendamento (Calendly) |
| **B** | fatura acima de 30k, mas "não é o meu momento" | tela de agendamento (Calendly) |
| **C** | fatura até 30k | tela de obrigado — nem vê a pergunta do investimento |

A regra vive na constante `FATURAMENTO_NIVEL_C` no topo do `<script>`.

## Arquivos

| Arquivo | Pra que serve |
|---|---|
| `index.html` | **produção** — CRM + GTM/pixel + notificação de WhatsApp, tudo ligado |
| `teste-debug.html` | painel no canto mostrando cada evento do dataLayer e cada chamada do pixel ao vivo (sem CRM, sem WhatsApp) |
| `teste-gtm.html` | com GTM, pra usar o Preview do Tag Assistant (sem CRM, sem WhatsApp) |
| `teste.html` | só o formulário, sem rastreio nenhum |

Os três arquivos de teste são **gerados a partir do `index.html`** — ao mexer no
formulário, mexa só no `index.html` e regere os outros.

## Rodar local

```bash
python3 -m http.server 8123
```

Depois `http://localhost:8123/teste-debug.html` (ou `index.html` pra testar com
CRM e WhatsApp de verdade — cuidado, cria lead).

## Rastreio

**GTM:** `GTM-K5K5SFP4` (conta GREEN SHARK) · **Pixel Meta:** `1001727392691930`

O pixel **não** está no HTML: ele entra como tag base no GTM (All Pages), e cada
evento do dataLayer abaixo vira uma tag `trackCustom`.

| Evento no dataLayer | Evento no Meta | Quando |
|---|---|---|
| `iniciou_formulario` | `Etapa1_IniciouForm` | clicou no CTA da capa |
| `colocou_nome` | `Etapa2_Nome` | nome validado |
| `colocou_instagram` | `Etapa3_Instagram` | @ preenchido |
| `colocou_telefone` | `Etapa4_WhatsApp` | número validado |
| `colocou_faturamento` | `Etapa5_Faturamento` | 1ª pergunta (param `faturamento`) |
| `colocou_investimento` | `Etapa6_Investimento` | 2ª pergunta (param `investimento`) |
| `lead_nivel_a/b/c` | `LeadNivelA/B/C` | classificação final (params `nivel`, `faturamento`) |
| `visualizou_calendario` | `VisualizouCalendario` | o calendário apareceu de fato na tela |
| `agendou_reuniao` | — | o Calendly avisou que a reunião foi marcada |
| `lead_classificado` | — | genérico, com `nivel` como parâmetro |

Ao ler o funil no Meta: **nível C não passa pela Etapa 6** (é filtro proposital,
não gargalo). O número comparável com a Etapa 6 é `Etapa5_Faturamento − LeadNivelC`.

## CRM

Webhook do CRM (Green Hub), disparado em dois momentos: **depois do telefone**
(cria o card, garante o lead mesmo se ela abandonar) e **no final** (completa as
respostas). A dedupe é por telefone — a segunda chamada atualiza o mesmo card.

Campos enviados: `nome_e_sobrenome`, `instagram`, `whatsapp` (número),
`qual_seu_faturamento_medio_mensal`, `investiria_1000_reais_em_anuncio`.

No nível C, como a pergunta do investimento não é feita, o campo recebe
`"Não perguntado (fatura até 30k)"` em vez de vir vazio.

## Calendly

Embed inline na tela final (`CALENDLY_URL`), com `name` e `a1` (WhatsApp)
preenchidos pela query string — o objeto `prefill` do widget não chega no iframe.

Dois detalhes que já custaram tempo, não mexa sem saber:

- o script do Calendly **não** aplica classe no container; a altura vive na nossa
  `.calendly-box`, senão o iframe colapsa pra ~150px;
- **não** dá pra pré-carregar o iframe fora da tela: o navegador não renderiza
  iframe escondido e o Calendly aparece em branco ao ser revelado.

Se o Calendly não der sinal em 12s, a caixa vira um botão "agendar em outra aba".

## Pendências

- [ ] hospedar (Vercel/Pages) e apontar a campanha
- [ ] conferir se a 1ª pergunta personalizada do evento do Calendly é o telefone
      (é onde o `a1` cai)
- [ ] idioma do evento do Calendly está em inglês
