# Push — notificações web

Além de email, a everinbox envia **web push** (a notificação que aparece no
navegador/desktop de quem aceitou receber). Serve como canal complementar ao
email — mesma lógica de "falar com quem optou por ouvir", em outra superfície.

Não force push como substituto do email; é um canal a mais. O custo do email é
imbatível; push soma alcance, não troca.

## Montagem

1. `create-push-app` — cria o "app" de push do projeto (a origem que pede
   permissão ao navegador do usuário). `list-push-apps` / `get-push-app` para
   inspecionar.
2. `add-push-domain` / `list-push-domains` — os domínios autorizados a pedir
   permissão e disparar.
3. `create-push-tag` / `list-push-tags` — rótulos para segmentar quem recebe.
4. `get-push-audience-options` — descubra os recortes de audiência disponíveis
   antes de montar a campanha.
5. `create-push-campaign` — cria o disparo. `update-push-campaign`,
   `get-push-campaign-details`, `list-push-campaigns` para gerir.

## Medir

- `get-push-dashboard-overview` — visão geral do canal.
- `list-push-campaign-executions` — o que já saiu.

## Cuidados

- Só recebe push quem **aceitou a permissão** no navegador — a base de push é
  separada da base de email e cresce sozinha conforme os visitantes aceitam.
- Assim como no email, frequência demais faz o usuário bloquear as notificações,
  e bloqueio não se desfaz fácil. Trate a atenção com a mesma economia.
