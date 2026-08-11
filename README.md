# Everinbox — Copilot para Claude Code

Plugin oficial da **Everinbox** para o [Claude Code](https://claude.com/claude-code). Conecta o Claude à sua conta Everinbox e ensina ele a operar a plataforma: campanhas, templates, listas, segmentos, métricas, entregabilidade, push e automações.

**O connector vem junto.** Instalar o plugin já configura o acesso — no primeiro uso o login abre sozinho.

## Instalar

```bash
/plugin marketplace add everinboxhq/everinbox-ai-plugin
/plugin install everinbox@everinbox-ai
```

Na primeira pergunta, o Claude abre o login da Everinbox. Você entra com a sua conta.

## O que dá pra pedir

Em português, do jeito que você falaria com alguém do time:

**Consultar**
- "Quais campanhas saíram essa semana no projeto X?"
- "Como foi a campanha de ontem?"
- "Meu spam rate subiu? Mostra os últimos 30 dias."
- "Quantos contatos tem esse segmento?"

**Montar**
- "Cria um segmento de quem abriu alguma coisa nos últimos 30 dias mas não clicou."
- "Faz um template de boas-vindas com o nome da pessoa no assunto."
- "Importa esses contatos na lista de newsletter."

**Disparar**
- "Agenda a campanha de sexta às 9h para o segmento de ativos."
- "Cancela a campanha que está agendada pra amanhã."

**Diagnosticar**
- "Por que a audiência veio menor do que eu esperava?"
- "A taxa de abertura caiu. O que mudou?"

## Seu acesso é o seu

A autenticação é por OAuth, com a sua conta. O Claude enxerga exatamente o que você enxerga — as enterprises e projetos em que você tem papel, nada além.

Se um projeto não aparecer, é permissão, não falha da ferramenta.

## O que ele confirma antes de fazer

Rascunhar é livre: criar template, montar lista, pré-visualizar segmento. Nada disso manda email.

**Ele para e pede confirmação antes de:**

- criar campanha (criar já é agendar ou disparar)
- editar campanha agendada ou no ar
- cancelar campanha
- apagar qualquer coisa
- mover contatos em volume
- mexer em automação ativa

Antes de confirmar, ele mostra o que vai acontecer: projeto, audiência com número de contatos, horário local, template e remetente.

## Requisitos

- Claude Code instalado
- Uma conta Everinbox com acesso a pelo menos um projeto

## Estrutura

```
.claude-plugin/
  marketplace.json      catálogo
  plugin.json           manifesto
.mcp.json               connector Everinbox
skills/everinbox/
  SKILL.md              descoberta, gotchas, guardrails
  references/           detalhe por frente, carregado sob demanda
    campanhas.md · templates.md · contatos-segmentos.md
    metricas.md · push-automacoes.md
```

## Contribuindo

Ao alterar o plugin, **suba a `version` no `.claude-plugin/plugin.json`** — sem isso quem já instalou não recebe a atualização.

Antes de abrir PR:

```bash
claude plugin validate ./
```

## Licença

MIT
