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
