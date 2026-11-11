# Financeiro: receita de anúncio por clique de email

Três tools somente de leitura que respondem "quanto o projeto faturou com os cliques dos emails". A receita vem do relatório do GAM já cruzado com os cliques no servidor, com a configuração de atribuição do projeto (rede, domínio do publisher, chave UTM) e a conversão de moeda aplicadas. **Use estas tools em vez de consultar ClickHouse ou Postgres**: uma chamada substitui o cruzamento inteiro.

| Pergunta | Tool |
|---|---|
| Quanto o projeto faturou no período? | `get-financial-summary` |
| Como foi dia a dia? | `get-financial-daily` |
| Qual campanha ou nó de automação rendeu? | `get-financial-breakdown` |

- `get-financial-summary` devolve `total_revenue` (USD), `total_impressions`, `total_clicks`, `total_sessions` e `epc` (receita por clique).
- `get-financial-daily` traz os mesmos números por data, em ordem crescente. Serve para tendência, projeção e comparar períodos. **A receita das últimas 48h é preliminar**, porque o GAM a reestima.
- `get-financial-breakdown` só traz linhas para projetos cuja rede GAM registra a chave `utm_evxcamp`. Projetos atribuídos pela chave `utm_source=opevx` (a maioria dos clientes OPEX) reportam um total por dia sem divisão por campanha, e a tool volta **vazia** para eles: é limitação da configuração do ad server, não bug. Se voltar vazia mas o resumo tiver receita, informe o total e diga que a divisão por campanha está indisponível. **Nunca divida o total entre campanhas para preencher o buraco.**

Erros de acesso: 403 `INSUFFICIENT_SCOPE` quer dizer que a conexão foi autorizada antes de poder ler receita: peça para desconectar e reconectar o connector Everinbox, aprovando "Visualizar receita e dados financeiros". 403 `FINANCIAL_ACCESS_DENIED` quer dizer que a conta da pessoa não tem acesso financeiro na Everinbox.
