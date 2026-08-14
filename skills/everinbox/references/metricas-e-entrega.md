# Métricas e entrega — medir e proteger a reputação

Enviar é metade do trabalho; a outra metade é ler o que voltou e reagir. Duas
famílias de dado: **resultado da campanha** (abriu, clicou) e **saúde de
entrega/reputação** (bounce, spam, o que o Gmail acha de você).

## Resultado das campanhas

- `get-campaign-metrics` — abertura, clique, entrega de uma campanha.
- `get-campaign-timeline` — a evolução no tempo.
- `get-campaign-ab-test-results` — vencedora do A/B.
- `get-template-metrics` / `get-template-clicks-by-url` — desempenho por
  template e quais links foram clicados.
- `get-project-dashboard` — visão geral do projeto.

### Como ler (o essencial)

- **Taxa de abertura** — quantos abriram sobre quantos receberam. Sinal de que o
  assunto+remetente funcionaram e de que você chegou na caixa de entrada.
- **Taxa de clique / CTOR** — quem clicou; o sinal de intenção real.
- Numa janela de vários envios, taxa se mede **por pessoa única**, não por
  evento — senão quem recebeu mais vezes distorce a conta.

## Saúde de entrega — os sinais que você não pode ignorar

Estes decidem se os próximos envios chegam:

- **Bounce** — email que não existe/rejeitou. Bounce alto diz ao provedor que
  sua lista é suja. Remova os hard bounces e não reenvie.
- **Reclamação de spam** — contato marcou "spam". É o pior sinal possível.
  Poucos já derrubam a reputação.
- **Descadastro** — saudável em dose normal; um pico súbito indica que você
  mandou para quem não devia ou com frequência demais.

**Regra de reação:** se bounce ou spam sobem, **desacelere** — volte a enviar só
para os mais engajados e segure o crescimento de volume até normalizar. Entrega
se recupera reagindo cedo, não empurrando mais volume.

## Postmaster — a visão do Gmail sobre você

O Google Postmaster Tools mostra como o Gmail enxerga seu domínio (reputação,
taxa de spam, autenticação). A everinbox integra isso:

- `connect-postmaster` / `list-postmaster-connections` — conectar/ver conexão.
- `list-postmaster-domains` / `map-postmaster-domain` / `sync-postmaster-domains`
  — gerir os domínios monitorados.
- `get-postmaster-metrics-domain` / `-project` / `-enterprise` — as métricas do
  Gmail no nível que você quiser.
- `list-postmaster-alert-configs` / `upsert-postmaster-alert-configs` /
  `list-postmaster-alert-history` — alertas automáticos quando algo piora.

Conectar o Postmaster no onboarding é altamente recomendado: é o painel que
avisa cedo quando a reputação começa a cair, antes de a entrega despencar.

## Financeiro (se o projeto usa)

`get-financial-summary` / `get-financial-daily` / `get-financial-breakdown` e
`get-utm-metrics` ligam envio a receita/UTM, quando o cliente rastreia isso.
