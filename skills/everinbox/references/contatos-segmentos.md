# Contatos, listas e segmentos

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

`export-list-contacts` faz o caminho inverso. `move-contacts` move entre listas — em volume, confirme antes.

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

Para apagar contato: `delete-contact` e `delete-contacts-batch`. Não têm volta — confirme sempre, e prefira mover para uma lista de descarte quando a intenção for só tirar de circulação.

## Frequency cap

`create-segment` aceita `frequency_cap` opcional: limites por dia, semana e mês, dias permitidos, janela de horário (`send_start_hour`/`send_end_hour`) e `timezone` (`"project"` ou `"contact"`).

Presets: `ativos`, `mornos`, `frios`, `quase_mortos`.

**Só persiste com `enabled: true`** — mandar `enabled: false` ou omitir limpa a configuração.

Contatos barrados pelo cap aparecem em `get-campaign-frequency-cap-blocked`. Vale olhar quando a audiência entregue veio menor que a esperada: pode não ser erro, pode ser o cap funcionando.
