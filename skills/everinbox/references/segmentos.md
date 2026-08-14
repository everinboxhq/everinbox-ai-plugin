# Segmentos — recortar a base

Um segmento é um recorte **vivo** da base definido por regras. É a ferramenta que
faz a higiene de entrega acontecer na prática: em vez de mandar para "todo
mundo", você manda para "quem abriu nos últimos dias". Ele recalcula sozinho
conforme os contatos engajam.

Tool principal: `create-segment`. Inspecionar: `preview-segment`, `get-segment`,
`get-segment-contacts`, `list-segments`.

## O erro que faz o segmento não casar nada

Dentro das regras, todo ID em `value` é **numeric_id** (`"123"`), **nunca
UUID**. Isso vale para `contact.list`, `contact.tag`, `contact.campaign`,
`contact.automation`, `contact.segment`. UUID ali falha em silêncio — o segmento
fica vazio e não dá erro. Pegue o numeric_id em `list-lists` / `list-tags` /
`list-campaigns`.

## Anatomia das regras

`rules` tem `include` (obrigatório, ≥1 regra) e `exclude` (opcional). Um contato
entra se casa o include e não casa o exclude. Tipos de regra disponíveis:

| Métrica | Para quê | Operadores / valor |
|---|---|---|
| `engagement.opens` / `engagement.clicks` / `engagement.delivered` | recorte por engajamento (o mais importante) | `gte/lte/gt/lt/eq` + número, com `date_range` opcional |
| `contact.list` / `contact.tag` / `contact.campaign` / `contact.automation` / `contact.segment` | pertence a / não pertence a | `in` / `nin` + array de **numeric_id** |
| `domain` | por provedor (gmail, yahoo...) | `in` / `nin` + nomes de domínio |
| `contact.created_at` | data de entrada do contato | precisa de `date_range` |
| `custom_field` | campo próprio do cliente | ver `list-custom-fields` primeiro |
| `rfm.r_score` / `f_score` / `m_score` | scores 1–5 (avançado) | `gte/lte/gt/lt/eq` |

`date_range` é `{ last_days: N }` (relativo) ou
`{ date_from, date_to }` (absoluto, `YYYY-MM-DD`).

## O recorte que todo cliente precisa: "engajados recentes"

Este é o segmento para o primeiro envio e para proteger a entrega. Ideia:
"abriu ou clicou nos últimos N dias".

```
create-segment:
  project_id: <UUID>
  name: "Engajados recentes"
  rules:
    include:
      - { metric: "engagement.opens", operator: "gte", value: 1,
          date_range: { last_days: 30 } }
```

Para o inverso (base "fria", que **não** deve receber cedo na rampa), monte um
segmento com `engagement.opens eq 0` num período — e mande para ele por último,
gradualmente, só depois que a reputação estiver saudável.

## Controle de frequência (não cansar o contato)

O segmento aceita `frequency_cap` para limitar quantos emails um contato recebe
por dia/semana/mês e em quais horas/dias. Isso evita fadiga e reclamação de
spam. Configure `enabled: true` com os limites (`max_per_day`, etc.); há também
presets nomeados. Detalhe fino na descrição da tool `create-segment` e em
`set-segment-frequency-cap` / `list-frequency-cap-presets`.

## Antes de usar um segmento numa campanha

Sempre confira o tamanho: `preview-segment` ou `get-segment` mostra a contagem.
Um segmento que, por regra mal montada, resolve para a base inteira é como se
apaga o ganho de reputação num envio só. Confirme a contagem com o cliente antes
de agendar.

Nota: segmento com `audience_limit` é um "corte priorizado" e **não pode ser
usado como gatilho de automação** (a API recusa) — para automação, use segmento
sem limite. Ver `references/automacoes.md`.
