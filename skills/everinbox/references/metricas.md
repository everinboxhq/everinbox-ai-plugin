# Métricas e entregabilidade

## De onde vem cada número

| Pergunta | Tool |
|---|---|
| Como foi esta campanha? | `get-campaign-metrics` |
| Como foi o dia/mês do projeto? | `get-project-dashboard` |
| Quem abriu, clicou, quando? | `get-campaign-timeline`, `get-campaign-contact` |
| A campanha rodou mesmo? | `get-campaign-executions` |
| Por que a audiência veio menor? | `get-campaign-frequency-cap-blocked` |
| Quem ganhou o A/B? | `get-campaign-ab-test-results` |
| Quantas campanhas em cada estado? | `get-campaigns-status-metrics` |
| Como cada segmento da campanha foi? | `get-campaign-segment-metrics` |
| Por estado ou país do contato? | `get-location-metrics` |
| Por UTM (origem, mídia, campanha)? | `get-utm-metrics` |
| Estou queimando reputação? | `get-postmaster-metrics-*` |

`get-campaign-metrics` devolve `delivered`, `opens`, `unique_opens`, `clicks`, `unique_clicks`, `bounces`, `unsubscribes`, `complaints` — e as taxas já calculadas. Não recalcule à mão.

`get-campaigns-status-metrics` responde no nível do projeto: quantas campanhas em `draft`, `scheduled`, `sending`, `published` e `canceled`, mais o engajamento agregado, opcionalmente numa janela de datas. É a visão certa para "o que está agendado?" e "o que ficou parado em rascunho?".

## Dashboard do projeto

`get-project-dashboard` sem filtro devolve o consolidado do projeto. Com filtro, ele fatia:

- `date_range` (`from`/`to` em `YYYY-MM-DD`)
- `campaign_ids`, `automation_ids`, `automation_node_ids`, `segment_ids`
- `source`: `"campaign"` ou `"automation"` — separa o que veio de disparo do que veio de fluxo
- `filter_type`: `"campaigns"`, `"automations"`, `"segments"` ou `"unified"` — como agrupar
- filtros por custom field do contato

Quando alguém pergunta "como foi o mês", o `unified` costuma ser a resposta certa; quando pergunta "o que a automação está trazendo", filtre por `source: "automation"`.

## Ler a taxa com honestidade

- **Compare janelas do mesmo tempo de vida.** Métrica de campanha matura: abertura continua entrando por dias. Comparar um envio de ontem com um de duas semanas atrás é comparar número jovem com número maduro, e inventa queda que não existe.
- **Taxa se lê junto com volume.** 40% de abertura em 8 mil contatos e 12% em 400 mil são coisas diferentes; a primeira não move o consolidado.
- **Audiência heterogênea dilui.** Somar um segmento grande e pouco engajado ao envio derruba a taxa consolidada por definição — sem que ninguém tenha engajado menos. Se a taxa caiu e o volume subiu, olhe a composição antes de concluir.
- **`unique_opens` e `opens` respondem perguntas diferentes.** Alcance é único; intensidade é total.
- **Abertura é sinal fraco hoje.** Proteção de privacidade infla abertura. Clique e conversão mentem menos.

Quando um número não veio da API, diga **"indisponível"**. Não estime.

## Postmaster e reputação

Três níveis, do panorama ao detalhe:

1. `get-postmaster-metrics-enterprise` — a enterprise inteira
2. `get-postmaster-metrics-project` — um projeto
3. `get-postmaster-metrics-domain` — série **diária, sem agregação**, de um domínio: spam rate, reputação de domínio e de IP, sucesso de autenticação, % de criptografia, erros de entrega

O caminho natural é de cima para baixo: se o rollup destoa, desça para o domínio e olhe dia a dia.

`enterprise_id` omitido usa a enterprise do token OAuth.

### Antes de ler

- `list-postmaster-connections` e `list-postmaster-domains` — o que está conectado e mapeado
- `connect-postmaster` / `map-postmaster-domain` / `sync-postmaster-domains` — para ligar e sincronizar
- `list-postmaster-alert-configs` e `upsert-postmaster-alert-configs` — alertas
- `list-postmaster-alert-history` — o que já disparou

**Domínio sem Postmaster conectado não tem esses dados.** Se as métricas vierem vazias, verifique a conexão antes de concluir que a reputação está boa — ausência de dado não é sinal verde.

### O que olhar

- **Spam rate** é o número que mais importa. Subiu, é urgente: taxa alta compromete a entrega de tudo que sai daquele domínio.
- **Reputação de domínio e de IP** são separadas e caem por motivos diferentes.
- **Falha de autenticação** (SPF, DKIM, DMARC) é problema de configuração, não de conteúdo — e some assim que o DNS é corrigido.
- **Reputação cai rápido e sobe devagar.** Um pico de spam custa semanas. Vale sempre parar e investigar antes de aumentar volume.

## Cortes por segmento, localização e UTM

- `get-campaign-segment-metrics`: desempenho de cada segmento usado numa campanha (destinatários, entregues, aberturas e cliques únicos, bounces, descadastros, taxas). Contato em vários segmentos conta em cada um, então `has_overlap=true` significa que as parcelas passam de 100%. Com `breakdown_available=false` não há número por segmento e `reason` diz por quê (`predates_tracking`, `snapshot_skipped`, `not_recorded`): não estime. Eventos contam até 15 dias após cada envio; em recorrente, no máximo os 30 envios dos últimos 60 dias.
- `get-location-metrics`: métricas por `location_state` (padrão) ou `location_country`, com a localização que o contato tinha **no momento do envio**. Valor vazio é desconhecido, e emails anteriores à gravação da localização caem todos aí. Passe `date_from` e `date_to` curtos: sem janela varre todo o histórico. Não tem corte por nó de automação: para isso use `get-project-dashboard` com `location_states` ou `location_countries`.
- `get-utm-metrics`: métricas por dimensão UTM (`utm_source`, `utm_medium` por padrão, `utm_campaign`, `utm_term`, `utm_content`) dos links rastreados dos emails. Mesma regra de janela curta. Para um nó de automação específico use `get-project-dashboard` com `automation_node_ids` mais `custom_field_filters`. `list-utm-values` lista as UTMs que o **contato** carrega, que é outra coisa.

## Bounce Guard e desbloqueio de soft bounce

`get-bounce-guard` lê a proteção de bounce do projeto (padrão sem config salva: desligada, modo `auto`). `update-bounce-guard` liga ou muda: modos `auto` (avisa em 2%, cancela em 5%), `warn` (só avisa, nunca cancela), `guard` (cancela nos limites recomendados) e `custom` (exige `custom_warn_threshold` menor que `custom_act_threshold`, em %). Exige papel OWNER ou MANAGER. **Isso decide quando campanha é cancelada sozinha**: confirme modo e limites.

Desbloquear contato barrado por soft bounce (falha temporária: caixa cheia, throttling, deferral):

1. `get-soft-bounce-unblock-preview` (por projeto, janela `date_from` a `date_to`, com no máximo 90 dias entre elas) mostra contagem por classificação. Hard bounce aparece só para informação e nunca é desbloqueado.
2. Mostre as classificações e contagens à pessoa e espere ela escolher quais desbloquear.
3. **`unblock-soft-bounced-contacts`** com as classificações confirmadas: contato desbloqueado volta a receber email. Descadastrado e quem marcou spam continuam bloqueados. Cada projeto ganha um job em segundo plano.
4. `get-unblock-status` até `completed` ou `failed`. Se vier `resumed=true`, o desbloqueio está completo mas `unblocked_count` pode estar até 10.000 abaixo do real.
