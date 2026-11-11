# Push e automações

## Push

### Estrutura

Um **push app** por projeto (`list-push-apps`, `get-push-app`, `create-push-app`), domínios registrados (`list-push-domains`, `add-push-domain`) e tags (`list-push-tags`, `create-push-tag`).

Antes de montar campanha: `get-push-audience-options` mostra o que dá para segmentar.

### `create-push-campaign`

Obrigatórios: `project_id`, `push_app_id`, `name`, `title`, `body`, `segment_ids`.

**Push é segmento-only.** Diferente do email, não existe `list_contact_ids` — a audiência vem de `segment_ids` (1 a 100). Quem quer disparar "para a lista X" precisa de um segmento que a represente.

Limites: `title` até 200 caracteres, `body` até 1.000, no máximo **2 `actions`** (cada uma com `action`, `title` até 50 e `url`).

- `status`: só `"draft"` ou `"scheduled"` — não há disparo imediato pela tool.
- `type`: `"regular"` ou `"recurrent"`. **A recorrência de push não tem `count`**: `end_type` é só `"never"` ou `"date"` (o email tem os três).
- `delivery_strategy`: `"standard"` ou `"best_time"`.
- `scheduled_at` + `selected_timezone` para agendar.
- `smart_replace_tag` substitui uma notificação anterior com a mesma tag em vez de empilhar — útil em atualização de status.
- `silent` entrega sem alerta; `require_interaction` mantém a notificação até a pessoa agir.

Acompanhamento: `list-push-campaigns`, `get-push-campaign-details`, `list-push-campaign-executions`, `get-push-dashboard-overview`.

**Push tem limite de caracteres real na tela, menor que o do schema.** Título longo é truncado pelo sistema operacional — escreva para caber, não para o máximo.

## Automações

### Inspecione antes de escrever

`list-automations` e `get-segment` primeiro. Fluxo ativo é a superfície que mais quebra em silêncio: o erro não aparece na hora, aparece no contato que não recebeu.

### `save-automation-flow` substitui tudo

Esta tool **troca o grafo inteiro** — nós e arestas. Não é edição incremental: o que você não mandar, some. Para mexer num fluxo existente, leia o grafo atual, altere o que precisa e devolva o conjunto completo.

Use `save-automation-flow` quando:

- a edição é de nível de grafo (mover conexões, reorganizar ramos)
- o tipo de nó é recusado pelo create de nó único: **`SPLIT`, `TRIGGER_ENGAGEMENT_*` e `FILTER_CONTACTS`**

Para nó isolado, as tools de nó são mais seguras: `create-automation-node`, `update-automation-node` e `delete-automation-node`. Em volume, use `create-automation-nodes-batch`, `update-automation-nodes-batch` e `delete-automation-nodes-batch` — mesmo cuidado, e leia o resultado item a item.

### Formato

- `nodes[].type` usa as strings do frontend: `"trigger:tag_added"`, `"action:email_sender"`, `"conditional:split"`.
- `edges` carregam o roteamento: `source_id`, `target_id`.
- **Aresta de split precisa de `type: "customSplit"`** mais `split_group` e `split_weight`. Sem isso o editor perde os percentuais do ramo — o fluxo roda, mas fica ilegível para quem abrir depois.

### Duplicar em vez de arriscar

`duplicate-automation` (e `duplicate-automations-batch`) é o caminho seguro para experimentar: copie, altere a cópia, valide, e só então mexa na que está no ar.

`rename-automations-batch` renomeia em lote.

## Guardrails desta frente

| Livre | Confirme antes |
|---|---|
| `list-*`, `get-*` de push e automação | `create-push-campaign` (agenda um envio) |
| `create-push-tag`, `add-push-domain` | `update-push-campaign` em campanha agendada |
| `duplicate-automation`, `list-automation-node-types` | **`save-automation-flow`** — substitui o grafo |
| `get-automation-flow`, métricas de automação | `update`/`delete-automation-node` em fluxo **ativo** |
| | **`set-automation-status`** — ligar, e principalmente pausar com `discard_pending` |
| | `delete-automation`, `delete-push-campaign` |
| | `create-automation-enrollment` (põe contatos no fluxo) e **`cancel-automation-enrollment`** |

Ao mexer em automação ativa, diga o que vai mudar em linguagem de fluxo ("o ramo de quem não abriu em 3 dias passa a esperar 5"), não em JSON. Quem aprova precisa entender o efeito, não o payload.

## Automação: o ciclo completo

`create-automation` cria; `get-automation` traz o cabeçalho; `get-automation-flow` traz o grafo (nós e arestas); `update-automation` altera o cabeçalho; `delete-automation` apaga.

**`list-automation-node-types` antes de montar ou editar fluxo.** Ela devolve os tipos válidos de gatilho, ação e condição — é o que evita inventar tipo de nó que o backend recusa.

### Ligar e desligar: cuidado com a fila

`set-automation-status` muda o estado: `ACTIVE`, `PAUSED`, `DRAFT`, `INACTIVE`, `FINISHED`.

**Pausar não cancela o que já está na fila.** Emails enfileirados continuam saindo depois do `PAUSED` — só param com `discard_pending: true`.

Isso importa numa emergência ("pausa isso agora, tem erro no link"): pausar sem `discard_pending` deixa o erro sair para quem já estava na fila. Quando a intenção for parar de verdade, **pergunte explicitamente se descarta os pendentes** e diga o que cada opção faz.

### Diagnóstico do fluxo

| Pergunta | Tool |
|---|---|
| Onde os contatos estão desistindo? | `get-automation-step-metrics` |
| Como vão as automações do projeto? | `get-automations-metrics` |
| Quantas em cada estado? | `get-automations-status-metrics` |
| O que mudou nessa automação? | `get-automation-history` |

`get-automation-step-metrics` é o mais útil dos quatro: dá o funil nó a nó — quantos chegam e quantos convertem em cada passo, opcionalmente por janela de data. Fluxo que "não performa" quase sempre tem um nó específico onde todo mundo cai.

`get-automation-history` responde "quem mexeu nisso e quando" — o primeiro lugar a olhar quando um fluxo mudou de comportamento sem ninguém assumir.

## Inscrever contatos em massa numa automação

`create-automation-enrollment` inscreve um segmento ou uma lista (`source_type` `segment` ou `list`, com o `numeric_id` em `source_id`) numa automação, liberando aos poucos em `rate_per_hour` no nó de gatilho escolhido (precisa ser gatilho de contato daquela automação).

- A amostra é **congelada no início do job**: sorteio por padrão (ou por id, com `random_sample=false`), até `max_contacts`, ou a origem inteira se omitido. O segmento mudar depois não altera a amostra.
- Cada contato entra pelas regras normais do gatilho: modo de entrada, cooldown, frequency cap de entrada e cobrança valem como em qualquer gatilho.
- Só libera enquanto a automação está `ACTIVE`. Pausada, o job espera e ninguém se perde.
- Só um job ativo por automação: o segundo é recusado (409) até o primeiro terminar ou ser cancelado.

`list-automation-enrollments` mostra os últimos 20 jobs e `get-automation-enrollment` acompanha um (liberados de total, taxa, erro). **`cancel-automation-enrollment`**: quem já foi liberado fica na automação, os restantes não entram. Confirme antes.
