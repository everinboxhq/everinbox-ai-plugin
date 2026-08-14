# Campanhas — enviar

Uma campanha amarra um **template** a uma **audiência** e a um **quando**. É o
envio em si. Tool: `create-campaign`. Antes de disparar de verdade, monte como
rascunho/agendada e confirme com o cliente.

## O que toda campanha precisa

`create-campaign` tem muitos campos obrigatórios; os que mais confundem:

- **Audiência — os dois arrays são sempre obrigatórios.** `list_contact_ids` e
  `segment_ids` precisam existir na chamada; passe `[]` no que não usar. Pelo
  menos um dos dois tem que ter item. Pode combinar lista + segmento.
  - `list_contact_ids` usa **numeric_id** da lista.
  - `segment_ids` usa **UUID** do segmento.
- **`template_id` (UUID) + `template_version` (inteiro, ex: 1)** — ambos
  obrigatórios.
- **`custom_from_message`** — obrigatório; use `"custom"` se não tem valor
  específico.
- **`sender_email` / `sender_name`** — o remetente. `sender_name` pode ser `""`.

## Tipo de envio × agendamento

Dois eixos independentes: **quando** (`type_send`) e **o que** (`type`).

**Enviar agora (rascunho controlado):**
```
type: "regular"
type_send: "imediato_estrangulamento"   // "estrangulamento" = envio com throttle
status: "draft"
```

**Agendar para depois:**
```
type: "regular"
type_send: "agendado_estrangulamento"
scheduled_time: "2026-01-15T14:00:00.000Z"   // ISO 8601
selected_timezone: "UTC-03:00"                // obrigatório ao agendar
status: "scheduled"
```

- **`duration`** define a janela de throttle (ex: `"1h"`, `"30m"`): a plataforma
  espalha o envio ao longo desse tempo em vez de despejar tudo de uma vez. Isso
  é a rampa acontecendo no envio — mantenha uma janela em vez de disparo
  instantâneo, principalmente com base grande.
- **`list_mode`**: `"fixed"` congela a audiência no momento da criação
  (recomendado para agendadas); `"variable"` resolve a audiência na hora do
  envio.

## Recorrente e Teste A/B

**Recorrente** (`type: "recurrent"`): repete em dias da semana. Exige
`recurrence_config` com `weekdays` (0=domingo..6=sábado) e `end_type`
(`never`/`date`/`count`), mais `scheduled_time` e `type_send` agendado.

**Teste A/B** (`type: "ab_test"`): exige `ab_test` com 2–4 variantes (cada uma
com `variant_label` A–D + `template_id` + `template_version`),
`sample_percentage` (1–50), `test_duration` e `winning_metric`
(`open_rate`/`click_rate`). A plataforma envia para a amostra, escolhe a
vencedora e manda para o resto. Também precisa de agendamento.

> O A/B é a *ferramenta* de teste. **O que** testar (qual assunto, qual ângulo)
> é decisão do cliente.

## Checklist antes de disparar

1. A audiência é o recorte certo? Confira a contagem (`preview-segment`).
   Para os primeiros envios, é o segmento de engajados — não a base inteira.
2. Assunto e remetente têm spintax se for envio repetido?
3. O template tem rodapé + `[unsubscribe]`?
4. Horário e timezone corretos? (Confirme o timezone do cliente; agendar em
   UTC errado envia na hora errada.)
5. **Cliente aprovou?** Só então mude para `sending`/confirme o agendamento.

## Depois

- `list-campaigns`, `get-campaign` — estado.
- `get-campaign-metrics`, `get-campaign-timeline` — resultado (ver
  `references/metricas-e-entrega.md`).
- `cancel-campaign` — cancela uma agendada antes de sair.
- `update-campaign` / `duplicate-campaign` — ajustar ou reusar.

## Não mexer (infra da everinbox)

`create-campaign` tem campos de roteamento de saída (pool de IP / provedor).
Isso é infraestrutura interna da everinbox e **não** é alavanca do cliente:
omita e a plataforma usa o padrão do projeto. O cliente controla audiência,
conteúdo e ritmo; a entrega técnica é da plataforma.
