# Contatos, listas, tags e campos

A base é a fundação. Contatos vivem no projeto; você os **organiza** em Listas
(agrupamento estático, ex: "Leads landing junho") e os **marca** com Tags
(rótulos transversais, ex: "comprou", "webinar"). Segmentos (arquivo à parte)
são recortes *vivos* por cima disso.

## Ordem de trabalho

1. `list-lists` — veja se a lista já existe antes de criar outra.
2. `create-list` — cria a lista. **Guarde os dois IDs do retorno:**
   - `id` (UUID) → use em import/export/mover contatos.
   - `numeric_id` → use em `list_contact_ids` de campanha.
3. `import-contacts` — sobe os contatos (assíncrono).
4. `get-import-status` — acompanhe o job até terminar.

## Importar contatos

`import-contacts` aceita de 1 a 10.000 contatos por chamada, é assíncrono
(retorna um `job_id`) e **deduplica por email** no servidor. Cada contato exige
`email`; `name` é opcional.

```
import-contacts:
  project_id: <UUID do projeto>
  list_id: <UUID da lista>
  contacts: [{ email, name }, ...]   // 1–10.000
→ retorna job_id → get-import-status(job_id) até status final
```

Para bases maiores que 10 mil, quebre em lotes de até 10 mil e importe em
sequência para a mesma lista; a deduplicação por email evita duplicados entre
lotes.

**Higiene na importação (ver a regra de entrega no SKILL.md):** suba só quem
optou por receber. Lista comprada ou base parada há muito tempo é o caminho mais
rápido para o spam e para queimar o domínio no primeiro envio.

## Tags

Tags marcam contatos por comportamento/atributo sem tirá-los da lista. Úteis
depois como filtro de segmento (`contact.tag`).

- `create-tag` / `create-tags-batch` — cria rótulos.
- `assign-tags-to-contacts` / `add-contacts-to-tag` — aplica.
- `remove-tags-from-contacts` / `remove-contacts-from-tag` — remove.
- `list-tags`, `get-tag-contacts` — inspeciona.

## Campos personalizados

Dados próprios do cliente por contato (cidade, plano, score...). Antes de usar
qualquer campo em segmento ou merge tag, **descubra o slug real**:

- `list-custom-fields` — retorna cada campo com seu `slug` e `field_type`.
- ⚠️ `city`/`cidade` são **reservados** (resolvem para a cidade da enterprise).
  Um campo de cidade por contato tem slug próprio do projeto (ex:
  `cidade_cliente`). Nunca presuma que o slug é `city`.

## Exportar / mover

- `export-list-contacts` — exporta os contatos de uma lista (usa o UUID).
- `duplicate-contacts` — copia contatos entre listas.

## Apagar (cuidado)

`delete-contact` / `delete-contacts-batch` removem contatos. É destrutivo e a
plataforma usa soft-delete, mas ainda assim confirme com o cliente antes. Para
"parar de mandar para alguém", o caminho normal é o contato se descadastrar pelo
link do rodapé, não deletar.
