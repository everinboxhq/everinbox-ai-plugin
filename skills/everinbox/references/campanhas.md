# Campanhas e templates

## Os três tipos

| Tipo | O que é | Exigências |
|---|---|---|
| `regular` | Envio único | O básico |
| `recurrent` | Envio repetido | `recurrence_config` (`weekdays` 0–6, `end_type`) + agendamento |
| `ab_test` | Teste A/B | `ab_test` com 2 a 4 variantes + agendamento |

`recurrent` e `ab_test` **só existem agendados** — exigem `scheduled_time` e `type_send: "agendado_estrangulamento"`.

`end_type` da recorrência: `"never"` (infinito), `"date"` (com `end_date` futura) ou `"count"` (com `end_count`).

No A/B: `sample_percentage` de 1 a 50, `test_duration` de `"5m"` a `"6h"`, `winning_metric` é `open_rate` ou `click_rate`. Vale definir `fallback_variant_label` — é a variante usada quando ninguém bate o `min_threshold`.

## Campos que se esquecem

- **`template_version`** é obrigatório e não tem padrão óbvio. Pegue do `get-template` (`current_version`).
- **`custom_from_message`** é obrigatório; use `"custom"` quando não houver valor específico.
- **`duration`** é a janela de estrangulamento (`"1h"`, `"30m"`). Não é o horário do envio.
- **`sender_name`** aceita string vazia, mas remetente sem nome aparece feio na caixa.
- `reply_to` é opcional e pode ser de outro domínio.
- `audience_limit` corta a audiência num teto — útil para teste controlado.

## Agendamento e fuso

`selected_timezone` vai como offset: `"UTC-03:00"`, `"UTC+01:00"`.

O `scheduled_time` é ISO 8601. **Confirme com a pessoa se o horário que ela falou já está no offset combinado** antes de criar — é o erro que manda a campanha três horas fora. Na resposta, sempre diga o horário local.

Cada projeto tem seu próprio `timezone` (veja em `list-projects`); não assuma que é o mesmo da pessoa.

## `list_mode`

- `"fixed"` — congela a audiência no momento da criação. Recomendado para agendado: o que você conferiu é o que sai.
- `"variable"` — resolve no envio. A audiência pode ter mudado desde a criação.

## Estados: o que dá para editar

`update-campaign` atualiza só os campos enviados, mas o estado manda:

- Campanha **filha de recorrente** não é editável.
- `dispatch_status: "success"` só aceita cancelamento.
- **Recorrente** só é editável em `draft`.
- Campanha **enviando** só aceita `status: "canceled"`.

O enum de `status` em `update-campaign` é maior que no create — inclui `active`, `completed`, `paused`, `ab_test_awaiting`. **Leia o estado com `get-campaign` antes de tentar editar**; erro de estado costuma voltar sem explicação clara.

## Templates

Obrigatórios: `project_id`, `name`, `html`, `subject`, `sender_name`. A enterprise é auto-preenchida.

**Sobre o `design` (JSON do editor Unlayer): não envie.** Mandando só `html`, o design é gerado — um bloco único que o editor abre normalmente. Só passe `design` quando você tem um layout autoral do editor que precisa preservar literalmente (copiado de um `get-template`).

**A resposta do create/update é enxuta e não devolve `html` nem `design`.** Para ler o conteúdo, `get-template`.

### Merge tags: rode `list-merge-tags` antes de escrever o HTML

A personalização é por tag entre colchetes (`[nome]`, `[email]`, `[data_de_hoje]`), dentro de `html`, `subject` ou `pre_header`.

**As tags disponíveis são do projeto, não universais.** `list-merge-tags` devolve as que existem de verdade ali — as de sistema **e** os campos customizados daquele projeto. Chame antes de gerar qualquer HTML: é o que impede inventar `[nome_completo]` ou `[primeiro_nome]`, que não existem e saem impressas literalmente no email.

A resposta vem em dois grupos:

- **`system`** — tags da plataforma. Algumas têm versão por idioma, e o `locale` indica o idioma do **valor renderizado**: `[dia_hoje]` → "segunda-feira", `[day_today]` → "Monday", `[dia_actual]` → "lunes". Tags com o mesmo `concept` apontam para a mesma informação em idiomas diferentes — **escolha a que bate com o idioma do template**, senão o email sai misturado. Tag sem `locale` rende igual em qualquer idioma (email, IDs, UTMs, dado cru do contato).
- **`custom`** — campos criados pelo projeto, com o tipo e, em select/multi_select, as opções válidas.

Algumas tags trazem **`recommended_html`**, um bloco pronto — é o caso do rodapé de descadastro. Use esse bloco como base em vez de escrever o seu.

**O descadastro é obrigatório.** Template sem ele é problema de entregabilidade e de lei — sinalize antes de deixar seguir.

Ao explicar as tags para a pessoa, use linguagem natural ("o primeiro nome do contato"), não os nomes técnicos de categoria ou tipo de campo.

`pre_header` é o texto de preview no cliente de email. Vale a pena: é a segunda linha que a pessoa lê.

## Imagens: a galeria do projeto

`list-gallery-items` lista as imagens do projeto com busca por nome (`search`), filtro por `tags`, `mime_type` e data, e ordenação por `recently_used`, `recently_uploaded`, `most_used` ou `size_desc`. Cada item traz `url`, dimensões, tamanho, `used_count` e `last_used_at`.

Antes de subir imagem nova, **procure na galeria** — reaproveitar evita duplicata e mantém o peso do email baixo.

`get-gallery-item-usage` mostra onde uma imagem está sendo usada. **Rode antes de apagar ou trocar**: imagem em uso em template ativo quebra o email de quem receber depois.

`upload-gallery-image` sobe, `update-gallery-item` edita metadados e tags, `delete-gallery-item` apaga (confirme antes). Por padrão a listagem esconde arquivados e apagados.

## Criar muitas campanhas de uma vez

`create-campaigns-batch` cria de 1 a 100 campanhas numa chamada só. Existe por um motivo prático: **`create-campaign` é limitada a 1 requisição por segundo**, então criar em laço é lento e esbarra em throttling.

Regras próprias do batch:

- **Todas as campanhas são do mesmo projeto.** `project_id` (e `enterprise_id` opcional) vão **uma vez no topo** — não repita dentro de cada objeto.
- Cada item segue exatamente as regras do `create-campaign`: os dois arrays de audiência, `template_version`, `custom_from_message`, e o mesmo par `type_send`/`status`.
- **Sucesso parcial é o normal.** Uma campanha ruim não aborta as outras; cada uma tem sua transação. O resultado vem com status por índice e um resumo `{ total, created, failed }`.

**Sempre leia o resultado item a item e relate os que falharam, com o erro.** Um batch que "funcionou" com 3 de 40 falhando é três disparos que não vão acontecer — e ninguém vai perceber até o dia.

> Batch de 100 campanhas é 100 envios comprometidos de uma vez. Confirme a lista inteira antes de executar — resumida por nome, horário e audiência, não em JSON.

## Sequência: montar e agendar uma campanha

1. `list-projects` — pegue `id`, `numeric_id`, `timezone`, `sender_domain`.
2. `list-campaigns` com `search` — confirme que o disparo não existe ainda.
3. `list-templates` / `get-template` — pegue `template_id` e `current_version`.
4. Audiência: `list-lists` ou `list-segments`; se for segmento novo, `preview-segment` antes de criar.
5. **Mostre para a pessoa** projeto, audiência (com número de contatos), horário local, template e remetente. **Espere o ok.**
6. `create-campaign`.
7. `get-campaign` — confira a audiência resolvida. Se vier zero, pare e avise.
