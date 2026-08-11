# Avaliações

Cenários que medem se a skill faz diferença. Cada arquivo descreve um pedido real e o comportamento esperado.

Não há runner embutido — a avaliação é feita rodando o pedido com a skill carregada e conferindo contra `expected_behavior`. Para estabelecer a linha de base, rode o mesmo pedido **sem** a skill e compare.

```bash
# com a skill
claude --plugin-dir ./ -p "<query do eval>"

# sem a skill (linha de base)
claude -p "<query do eval>"
```

Cada cenário aqui existe porque corresponde a uma falha que o schema sozinho não evita. Se um deles passar igual com e sem a skill, ou a skill não está agregando naquele ponto, ou o cenário está mal escrito.

| Arquivo | O que mede |
|---|---|
| `01-disparo-imediato.json` | Não tratar "draft" como rascunho seguro; confirmar antes de comprometer envio |
| `02-segmento-id-numerico.json` | Usar ID numérico em regra de segmento e pré-visualizar antes de criar |
| `03-pausar-automacao.json` | Perguntar sobre a fila pendente ao pausar automação |
| `04-template-recorrente.json` | Buscar merge tags reais do projeto e exigir Spintax em envio repetido |
| `05-descoberta.json` | Não inventar ID; descobrir enterprise → projeto antes de agir |
