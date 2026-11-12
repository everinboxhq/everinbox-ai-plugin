# Sentinel: verificação de emails

Sentinel verifica se um email é entregável. **Verificar gasta crédito Sentinel, um por contato verificado.** Esta frente é toda sobre mostrar o custo antes e esperar o ok.

## Conteúdo

- Verificar contatos de uma lista
- Import com verificação
- Leads que chegam pela API
- Consultar saldo e histórico

## Verificar contatos de uma lista

1. `get-sentinel-entitlement` (barato, usa saldo em cache) diz se a enterprise pode verificar agora; `get-sentinel-usage` traz o saldo ao vivo.
2. `preview-sentinel-verification` não gasta nada. Selecione por `list_ids` **ou** `contact_ids` (um dos dois). Devolve `needs_verification` (o que seria verificado e cobrado), `already_verified` (veredito ainda válido, pulado a menos que `include_verified=true`) e `in_verification` (outro job está verificando, nunca cobrado duas vezes). **Sempre chame primeiro e mostre o número à pessoa.**
3. **`create-sentinel-verification-job`**: gasta crédito. Exige confirmação explícita e `max_contacts` igual ao número que a pessoa viu (somando `already_verified` quando `include_verified=true`): a API recusa o job inteiro se cobrir mais que isso, para ninguém ser cobrado por número que não viu. Exige papel OWNER ou MANAGER, módulo de contatos e email habilitado. Erros com prefixo `insufficient_credits:` e `nothing_to_verify:` já trazem mensagem para repassar à pessoa.
4. Acompanhe com `get-sentinel-verification-job` (status `reserved`, `submitted`, `completed` ou `failed`, progresso e contagens). `list-sentinel-verification-jobs` lista os jobs do projeto, mais novos primeiro, e `list-sentinel-list-verifications` mostra a última verificação por lista (em andamento ou das últimas 24h). Mantenha `limit` pequeno e só pagine quando a pessoa pedir.

## Import com verificação

`import-contacts` com `sentinel_verify=true` verifica antes de importar. Acompanhe com `get-import-verification` (o `job_id` vem dessa chamada):

- `extracting` ou `submitted`: Sentinel ainda verifica, volte depois (base grande leva minutos).
- `completed`: pronto. Mostre o resumo (`unique_emails`, `skipped_fresh`, `submitted`, `deliverable`, `undeliverable`, `risky`, `unknown`, `barred`, `will_import`) e **pergunte** antes de seguir. Uma verificação completa espera 7 dias por confirmação e depois expira.
- `confirm-import-verification`: importa os contatos sem os emails barrados (inválidos). Idempotente: a segunda chamada responde `already_confirmed=true` e não importa de novo. Depois acompanhe com `get-import-status`.
- `insufficient_credits`, `failed` ou `expired`: a verificação não terminou. **`import-without-verification`** importa os contatos originais **sem filtro**: só a pedido explícito da pessoa, porque email inválido entra na base. É recusado enquanto a verificação ainda roda.
- `confirmed`: já foi entregue ao import.

## Leads que chegam pela API

`update-sentinel-settings` (só owner ou manager) muda a verificação do projeto:

- `verify_api_leads`: **ligar gasta crédito Sentinel em cada lead novo que chegar pela API.** Rode `get-sentinel-api-lead-estimate` antes (volume diário recente e crédito que gastaria), mostre à pessoa e confirme.
- `api_leads_daily_limit`: máximo de leads verificados por dia. **Obrigatório enquanto `verify_api_leads` estiver ligado**; `null` remove.
- `validity_days`: por quanto tempo um veredito vale antes de reverificar; `null` nunca expira.

Informe `project_id` mais pelo menos um dos três; só o que você manda muda.

## Consultar saldo e histórico

`get-sentinel-usage` devolve saldo, cota diária, consumo do mês e um extrato paginado (compras, verificações, estornos, mais novo primeiro). Mantenha `limit` pequeno.
