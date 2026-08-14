# Automações — envios que disparam sozinhos

Uma automação é um fluxo que reage a um evento: contato entra num segmento,
ganha uma tag, engaja (ou para de engajar). O caso clássico — e o mais valioso
para começar — é o **boas-vindas**: todo contato novo recebe o primeiro email
automaticamente, no melhor momento possível (logo depois de entrar, quando o
interesse está no pico).

## Por que o boas-vindas importa tanto para a entrega

O primeiro email para um contato novo é quando ele está **mais propenso a abrir**
— acabou de se cadastrar, lembra de você. Abertura alta logo de cara é o sinal
de reputação mais barato que existe. Automatizar isso garante que nenhum contato
novo fique "esfriando" numa lista esperando o próximo disparo manual.

## Como uma automação é montada

Diferente de campanha, a automação é construída em duas etapas:

1. `create-automation` — cria a automação **vazia** e já **ativa**, só com
   nome + descrição.
2. Monta o fluxo (nós + ligações):
   - `create-automation-node` — adiciona um nó por vez (bom para fluxos
     simples: gatilho → email).
   - `save-automation-flow` — substitui o grafo inteiro de uma vez (necessário
     para nós que o create-node recusa: `SPLIT`, `TRIGGER_ENGAGEMENT_*`,
     `FILTER_CONTACTS`, e para edições de grafo).

**Sempre chame `list-automation-node-types` antes de montar** — ele lista os
tipos válidos de gatilho, ação e condição. Não presuma nomes de nó.

## Peças de um fluxo

- **Gatilho** — o que faz o contato entrar. Ex: `TRIGGER_SEGMENT_ENTERED`
  (entrou num segmento), tag adicionada, engajamento.
- **Ações** — `EMAIL_SENDER` (manda um email; precisa de `template_id`,
  `template_version`, `sender_email`, `sender_name`, `subject`), adicionar tag,
  timer (esperar X antes do próximo passo), etc.
- **Condições** — `SPLIT` (divide o fluxo em ramos), filtros.

### Boas-vindas mínimo (gatilho → email)

```
1. create-automation → { name: "Boas-vindas", project_id }
2. create-automation-node → gatilho (ex: TRIGGER_SEGMENT_ENTERED,
     data.segment_id = <numeric_id do segmento>, NÃO o uuid)
3. create-automation-node → EMAIL_SENDER
     data: { template_id, template_version, sender_email, sender_name, subject }
```

## Cuidados que a API cobra

- **Gatilho de segmento usa o `numeric_id` do segmento** (string), não o UUID.
- O segmento do gatilho precisa ser **canal email** e **sem `audience_limit`**.
  Segmento com limite é um corte priorizado — um contato pode "parecer" que
  entrou só porque outro saiu do corte; a API recusa. Use um segmento sem limite
  para gatilho.
- Se a automação tem `entry_mode: "multiple"` e um gatilho de segmento, a API
  exige `frequency_cap_enabled` ligado (para o contato não reentrar sem parar).

## Gerir

- `list-automations`, `get-automation`, `get-automation-flow` — inspecionar.
- `set-automation-status` — ligar/pausar.
- `duplicate-automation` — reusar um fluxo que funciona.
- `get-automations-metrics` / `get-automation-step-metrics` — quanto cada passo
  entregou/engajou.

> Os **tipos** de fluxo (boas-vindas, re-engajamento, nutrição) são mecânica da
> plataforma. A régua de *qual sequência* e *qual conteúdo* maximiza retenção é
> estratégia do cliente — a skill entrega a capacidade, não o roteiro.
