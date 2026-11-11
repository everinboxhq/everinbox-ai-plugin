# Contatos, listas e segmentos

## Conteúdo

- Lista tem dois IDs — UUID vs `numeric_id`, e o que `create-list` exige
- Importar contatos — assíncrono, limites, o que não entra
- Regras de segmento — a armadilha do ID numérico, tipos de regra, custom fields
- Tags — CRUD e o caminho certo para volume
- Ciclo de vida do segmento — uso, status, histórico, antes de destruir
- Frequency cap — os dois caminhos e por que os campos não batem
- Frequency cap do projeto: vale para todo envio
- Custom fields: definição e valor por contato
- Gerir listas: arquivar, apagar, esvaziar
- Lote: criar e duplicar em volume

## Lista tem dois IDs

`create-list` devolve `id` (UUID) **e** `numeric_id` (inteiro). Os dois são usados em lugares diferentes:

- **UUID** — import, export, move, e o campo `list_contact_ids` de `create-campaign` (o schema da campanha pede UUID).
- **`numeric_id`** — regras de segmento.

> A documentação das duas tools diverge sobre `list_contact_ids`. Siga o schema de `create-campaign` (UUID) e **confirme com `get-campaign` depois de criar**: se a audiência vier zero, foi o ID errado.

`create-list` é a tool mais exigente do conjunto: precisa de `name`, `project_id`, **`project_numeric_id`** e **`enterprise_id`**. As outras auto-preenchem a enterprise; esta não.

## Importar contatos

`import-contacts` é **assíncrono**: devolve `job_id` e o progresso sai no `get-import-status`. Não trate como concluído no retorno.

- 1 a 10.000 contatos por chamada. Base maior, quebre em lotes.
- Dedup por email no servidor — não precisa limpar duplicata antes.
- **Aceita só `email` e `name`.** Custom field não entra por aqui. Quem espera importar cidade, CPF ou plano junto vai se frustrar: avise antes, não depois.

`import-contacts-batch` importa em até 10 listas numa chamada e devolve um `job_id` **por lista**: acompanhe cada um no `get-import-status`. Sucesso parcial é normal, leia item a item.

Import com verificação Sentinel (`sentinel_verify=true`) tem fluxo próprio: veja [Sentinel](sentinel.md).

`export-list-contacts` faz o caminho inverso. `duplicate-contacts` copia os contatos de uma lista para uma lista de outro projeto (a origem não muda) — em volume, confirme antes.

## Regras de segmento

`create-segment` monta `rules` com `include` (obrigatório, ao menos uma regra) e `exclude` (opcional).

### A armadilha principal

**Todo ID em `value` é numérico, nunca UUID.** Vale para `contact.list`, `contact.tag`, `contact.automation`, `contact.campaign` e `contact.segment`.

UUID ali **não dá erro** — a regra simplesmente casa zero contatos. É a falha mais cara do MCP porque parece sucesso.

### Os tipos de regra

| Métrica | Operadores | Valor |
|---|---|---|
| `contact.list` · `contact.tag` · `contact.automation` · `contact.campaign` · `contact.segment` | `in`, `nin` | array de IDs **numéricos** |
| `engagement.opens` · `engagement.clicks` · `engagement.delivered` | `gte`, `lte`, `gt`, `lt`, `eq` | número (aceita `date_range`) |
| `domain` | `in`, `nin` | `["gmail", "yahoo"]` |
| `contact.created_at` | — | exige `date_range` |
| `rfm.r_score` · `rfm.f_score` · `rfm.m_score` | `gte`, `lte`, `gt`, `lt`, `eq` | 1 a 5 |
| `custom_field` | depende do tipo | depende do tipo |

`date_range` aceita `{ last_days: N }` ou `{ date_from, date_to }` no formato `YYYY-MM-DD`.

### Custom fields

Rode **`list-custom-fields` antes** para pegar o `slug` e o `field_type`. Depois mande `{ metric: "custom_field", field_slug, field_type, operator, value }`.

**`city` e `cidade` são reservados** — resolvem para a cidade da enterprise, não do contato. Um campo de cidade por contato tem slug próprio do projeto (algo como `cidade_cliente`). Nunca assuma o slug.

Operadores por tipo:

- **text** — `eq`, `ne`, `contains`, `not_contains`; `exists`, `not_exists` (sem valor)
- **number** — `eq`, `ne`, `gt`, `gte`, `lt`, `lte`; `between` (`{min, max}`); `exists`, `not_exists`
- **date** — `before`, `after` (`YYYY-MM-DD`); `between` (`{from, to}`); `in_last_n_days`; `exists`, `not_exists`
- **select** — `in`, `not_in`; `exists`, `not_exists`
- **multi_select** — `contains_any`, `contains_all`, `contains_none`; `exists`, `not_exists`
- **boolean** — `is_true`, `is_false`, `exists`, `not_exists`

### Sempre pré-visualize

`preview-segment` antes de `create-segment`. Sem isso você descobre a regra errada quando a campanha sai vazia — ou pior, quando ela não sai.

`recalculate-segment` reprocessa um segmento existente. Útil quando a base mudou e o número parece velho.

`update-segment` altera as regras de um segmento que já existe. **Cuidado com segmento em uso:** campanha agendada com `list_mode: "variable"` resolve a audiência no envio — mexer na regra muda quem vai receber, sem aviso. Antes de editar, verifique se há campanha agendada apontando para ele; se houver, `preview-segment` com a regra nova e mostre a diferença de tamanho antes de salvar.

`get-contact` traz o perfil de um contato (email, nomes, validação, custom fields, UTMs, tags, listas). Passe **um só** entre `contact_id` (id numérico global) e `email` (busca exata, sem diferenciar maiúsculas; 404 = nenhum contato do projeto com esse email). Para descobrir ids: `export-list-contacts` ou `get-tag-contacts`.

Para apagar contato: `delete-contact` e `delete-contacts-batch`. Não têm volta — confirme sempre, e prefira mover para uma lista de descarte quando a intenção for só tirar de circulação.

## Tags

Tag é rótulo de contato — a forma mais leve de agrupar sem criar lista nova. Entram em regra de segmento via `contact.tag`.

CRUD: `list-tags`, `get-tag`, `create-tag`, `update-tag`, `duplicate-tag`, `delete-tag`. Em volume: `create-tags-batch` (1 a 25; nome de 3 a 50 caracteres, só letras, números, hífen e sublinhado, sem espaço).

Para ligar contato e tag existem dois caminhos, e escolher errado custa chamadas:

- **`assign-tags-to-contacts`**: uma ou várias tags em um ou vários contatos, numa chamada. É o caminho certo, inclusive para uma tag só.
- **`remove-tags-from-contacts`**: o inverso, também com uma ou várias tags.

**Os IDs aqui são numéricos**, tanto `contact_ids` quanto `tag_ids` — mesma regra dos segmentos. IDs muito grandes podem vir como string numérica; UUID não serve.

Consulta: `get-contact-tags` (as tags de um contato) e `get-tag-contacts` (os contatos de uma tag).

Antes de apagar uma tag, lembre que ela pode estar dentro de regra de segmento ativa — apagar muda silenciosamente quem o segmento casa.

## Ciclo de vida do segmento

| Pergunta | Tool |
|---|---|
| Quem está nele? | `get-segment-contacts` |
| Já terminou de processar? | `get-segment-status` |
| **Onde ele é usado?** | `get-segment-usage` |
| O que mudou nele? | `get-segment-history` |

**`get-segment-usage` antes de apagar ou editar pesado.** Ele mostra as campanhas e automações que referenciam o segmento — é literalmente o raio de alcance da mudança. Segmento que parece órfão costuma estar dentro de uma automação ativa.

`duplicate-segment` é o caminho seguro para experimentar regra nova sem mexer no que está em uso. `archive-segments` tira de circulação sem destruir; `delete-segment` apaga (confirme, e só depois de ver o `usage`).

Regra por localização ou UTM do contato: rode antes `list-location-values` (estado, país ou fuso, com `search`) ou `list-utm-values` (valores de uma coluna `utm_*`). A regra casa **valor exato**, e a de UTM diferencia maiúsculas: valor chutado não casa ninguém. `list-utm-values` é a origem do contato, não o UTM de um link dentro do email (esse é `get-utm-metrics`).

Segmento grande não fica pronto na hora: **`get-segment-status` antes de usar** em campanha, senão você agenda em cima de um segmento ainda processando.

## Frequency cap: dois caminhos, campos diferentes

O cap limita quantos emails os contatos de um segmento podem receber. Dá para configurar de dois jeitos — e **os nomes dos campos não batem entre eles**:

| | Inline no `create-segment` | Tool dedicada `set-segment-frequency-cap` |
|---|---|---|
| Janela | `send_start_hour` / `send_end_hour` | `window_start_time` / `window_end_time` (`"HH:MM"`) |
| Fuso | `timezone` | `timezone_mode` |
| Dias | `allowed_days` como texto (`['mon','tue']`) | `allowed_days` como **números** (0=domingo … 6=sábado) |

Use a tool dedicada para mexer só no cap sem tocar nas regras do segmento. Ela exige **pelo menos um** entre `max_per_day`, `max_per_week` e `max_per_month` — sem nenhum, a API recusa.

`apply-segment-frequency-cap-preset` aplica um preset nomeado (`ativos`, `mornos`, `frios`, `quase_mortos`); `list-frequency-cap-presets` mostra quais existem de fato no projeto. `get-segment-frequency-cap` lê o atual e `delete-segment-frequency-cap` remove.

O cap inline **só persiste com `enabled: true`** — mandar `enabled: false` ou omitir limpa a configuração.

Quando a audiência entregue vier menor que a esperada, cheque `get-campaign-frequency-cap-blocked` antes de procurar bug: pode ser o cap funcionando como configurado.

## Frequency cap do projeto

Além do cap por segmento, o projeto tem um cap próprio: o máximo de emails que **um contato** recebe por hora e por dia, somando toda campanha e automação do projeto. `get-project-frequency-cap` lê; `set-project-frequency-cap` define e **substitui os dois limites de uma vez**: envie sempre `max_per_hour` e `max_per_day`, com número para limitar e `null` para deixar sem limite (leia o atual antes, para não apagar o que já estava); `delete-project-frequency-cap` desliga o cap do projeto sem tocar nos caps de segmento.

**Set e delete mudam todo envio do projeto daqui para frente.** Confirme os números com a pessoa antes.

## Custom fields

`list-custom-fields` traz as definições (slug, tipo, opções). A definição do campo e o valor de cada contato são coisas separadas.

- `create-custom-field`: o slug vira a merge tag `[slug]` e a chave do valor no contato. Rode `list-custom-fields` antes: slug já usado é recusado, merge tags nativas (`nome`, `email` e afins) são reservadas, `select` e `multi_select` exigem `options`, e o projeto aceita no máximo 200 campos.
- `update-custom-field`: muda nome de exibição, opções, obrigatoriedade, métricas ou o papel de cidade. Slug e tipo não mudam; `options` substitui a lista inteira.
- `delete-custom-field`: **sem volta**, e a merge tag `[slug]` para de resolver nos emails que a usam. Confira segmentos e templates antes e confirme. Exige papel OWNER ou MANAGER.
- `set-contact-custom-fields`: grava valores em **um** contato, com merge (o que você não manda fica como está, `null` limpa). Valor inválido ou slug desconhecido **não derruba a chamada**: volta em `errors` enquanto os válidos são salvos, então leia o resultado.

## Gerir listas

- `update-list`: renomeia, bloqueia ou desbloqueia (`status` `active` ou `blocked`) ou marca como seedlist. `is_seed` só liga: seedlist nunca volta a ser lista normal. Lista interna não é editável.
- `archive-lists`: esconde (ou restaura, com `archived=false`) até 100 listas sem apagar nada. É o que se usa quando a pessoa só quer tirar a lista de vista.
- **`delete-lists`**: apaga até 100 listas (os contatos não são apagados), **mas cancela campanhas que miravam só essas listas, pausa automações que as usam e tira as listas da audiência das outras campanhas**. Não há como desfazer. Prefira `archive-lists` e confirme sempre, dizendo o que será cancelado e pausado.
- **`clear-list`**: tira contatos da lista e mantém a lista. Por padrão tira **todos**, e as associações não voltam. Dá para restringir a um critério só: `only_cancelled=true` ou `sentinel_status="undeliverable"`. Lista interna não pode ser esvaziada. Confirme sempre.

## Lote: criar e duplicar em volume

`create-lists-batch` e `create-segments-batch` criam de 1 a 25 por chamada, `duplicate-segments-batch` e `duplicate-tags-batch` duplicam até 50. Todos devolvem um resumo mais CSV por item, e sucesso parcial é o normal: relate o que falhou.

`duplicate-segments-batch` entre projetos **tira as regras de pertencimento** (`contact.list`, `contact.tag`, `contact.campaign`, `contact.automation`, `contact.segment`), porque elas apontam IDs numéricos do projeto de origem. Sobram só regras que não dependem do projeto (engajamento, domínio, RFM, criação), e a cópia casa **outra audiência**. Avise antes.
