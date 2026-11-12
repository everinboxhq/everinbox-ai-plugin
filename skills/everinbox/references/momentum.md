# Momentum

Momentum mede, pela rede Momentum, onde o email cai (caixa de entrada, promoções, spam) e funciona com um plano de cota diária de envios. Toda tool de Momentum age na enterprise do `project_id` informado. 403 `MOMENTUM_NOT_ENABLED` quer dizer que a conta ainda não foi liberada: quem libera é o time da Everinbox.

Número que vem nulo (`usage`, `used_today`, consumo do plano) quer dizer **desconhecido**, não zero: o Momentum estava inacessível naquele momento.

## Conteúdo

- Cota e remetentes
- Criar campanha Momentum
- Posicionamento na caixa de entrada
- Plano compartilhado entre enterprises

## Cota e remetentes

- `get-momentum-overview`: cota diária (renova à meia-noite de America/Sao_Paulo), status (`active`, `paused` quando o pagamento está bloqueado, `canceled`), remetentes com limite e uso do dia. `senders_over_quota` aponta limites que somam mais que a cota. Chame antes de criar campanha para escolher um remetente registrado com folga.
- `get-momentum-sender-volume`: envios por remetente e dia (até 31 dias): `real_sends`, `seed_sends` e `seed_share`. Serve para dimensionar o limite de um remetente contra o volume real.
- `set-momentum-sender`: registra um remetente ou muda o limite diário. O email precisa já ser remetente de um projeto da enterprise. Subir o limite tem que caber: a soma dos limites não passa da cota do plano nem do teto da enterprise num plano compartilhado. Reduzir sempre funciona. Só owner ou manager muda remetente.
- **`remove-momentum-sender`**: devolve a parte da cota ao plano, e campanhas Momentum novas desse remetente passam a ser recusadas até ele ser registrado de novo. Confirme antes.

## Criar campanha Momentum

`quote-momentum-campaign` faz a mesma checagem do `create-campaign`: devolve `fits_window`, `allocatable` (quanto do volume cabe), `shortfall_reason` (nulo quando cabe) e `min_duration_minutes`. A `duration` aceita `5m`, `10m`, `30m` ou `1h` a `23h`. `scheduled_time` (ISO 8601 com offset) para agendada; omita para envio imediato. Se não cabe: reduza o volume a `allocatable`, use janela maior, outro remetente ou outro dia. Remetente não registrado dá erro `MOMENTUM_SENDER_NOT_REGISTERED`: use `set-momentum-sender`.

`list-momentum-campaigns` lista as campanhas Momentum do projeto (filtros `search`, `status`, `type`, `sender_email`).

## Posicionamento na caixa de entrada

- `get-momentum-campaign-placement`: uma campanha (e cada envio, se recorrente), com contagens e taxas: `detection_rate` sobre o alocado, `inbox_*_rate` e `spam_rate` sobre o detectado, `rescue_rate` sobre spam. A detecção continua chegando por um tempo depois da janela de envio.
- `get-momentum-placement`: um período (início e fim inclusivos, `YYYY-MM-DD`), por dia, projeto, remetente e campanha, mais um `pool` com as mesmas taxas de todos os clientes Momentum como referência (nenhum identificado). Filtros `filter_project_id` e `sender_email`.

## Plano compartilhado entre enterprises

Só owner ou manager da enterprise que **paga** gerencia o plano.

- `get-momentum-plan`: cota, status, pagamento, consumo de hoje e as enterprises do plano com teto e remetentes. O gestor vê tudo; os demais veem os totais e só a própria enterprise.
- `list-momentum-plan-events`: histórico, mais novo primeiro (`next_cursor` vira `cursor` na página seguinte).
- `list-momentum-invitable-enterprises`: enterprises que o gestor pode convidar (as de que ele também é membro e que ainda não estão em plano).
- **`invite-enterprise-to-momentum-plan`**: convida outra enterprise, que só entra quando um owner ou manager dela aceita. Depois de dentro, ela divide a cota do plano. Confirme antes.
- `cancel-momentum-plan-invite`: cancela um convite pendente (o id vem de `get-momentum-plan`). Confirme antes.
- `list-momentum-incoming-invites`: convites pendentes para a enterprise do `project_id` entrar no plano de outra.
- **`answer-momentum-plan-invite`**: aceitar vincula a enterprise ao plano e ela passa a enviar da cota compartilhada (contrato Momentum antigo precisa ser encerrado antes, 409). Recusar só fecha o convite. Só owner ou manager da enterprise convidada responde. Confirme antes.
- **`remove-enterprise-from-momentum-plan`**: a enterprise para de enviar da cota do plano na hora e seus remetentes deixam de contar. Só owner ou manager da enterprise removida a tira (não o gestor do plano). A enterprise pagadora não sai (409): nesse caso o plano é cancelado. Confirme antes.
- **`set-momentum-enterprise-cap`**: teto de uma enterprise dentro do plano (máximo da cota diária que ela pode enviar). `cap=null` remove o teto; teto acima da cota equivale a nenhum. Muda quanto cada enterprise consegue enviar: confirme antes.
