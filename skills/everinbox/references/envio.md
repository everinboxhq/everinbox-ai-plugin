# Configurar envio: domínios, IPs, pools e projeto

Aqui o envio é pelo ESP da Everinbox. Esta frente **gasta dinheiro** (IP dedicado é cobrado por mês) e **muda a reputação** de todos os projetos de um pool. A liberação é por enterprise: onde a pessoa não é admin da plataforma nem owner liberado pelo time, toda tool de envio responde 403 `SENDING_SETUP_NOT_ENABLED`. É falta de liberação, não bug.

## Conteúdo

- A ordem certa
- Domínios de envio
- Criar projeto
- IPs, pools e reverse DNS

## A ordem certa

1. `add-sending-domain`: devolve os registros DNS que o cliente precisa publicar (CNAME do return-path, dois CNAMEs DKIM, TXT de DMARC e o CNAME de tracking `track.<domínio>`). Um DMARC existente é mantido, nunca substituído.
2. `verify-sending-domain` até verificar (DNS leva minutos). Pode repetir à vontade.
3. `list-sending-pools` com `for_new_project=true` e então `create-project` com remetente em domínio verificado.
4. `list-sending-ips` e, para cada IP, `set-ip-reverse-dns` (e `verify-ip-reverse-dns` quando há hostname próprio).

## Domínios de envio

- `list-sending-domains`: cada domínio traz `verification_status` (`pending`, `verified`, `failed`) e o bloco de tracking. `ready=true` quer dizer autenticação verificada **e** host de tracking resolvendo com certificado ativo.
- `get-sending-domain`: um domínio com os registros e o status de cada um na última verificação.
- `get-domain-connect-url`: link de um clique do Cloudflare que aplica os registros de email na zona. A pessoa abre e aprova no navegador. `url` vem nula quando o clique único não está configurado. O mesmo link já vem em `add-sending-domain` quando a zona é Cloudflare e o domínio ainda não está verificado; o CNAME de tracking pode continuar pendente.
- `verify-sending-domain`: o campo `capacity` só vem preenchido com o domínio verificado e um projeto da conta esperando IP: `ready` quer dizer que já pode enviar, `no_free_ip` que o ESP está sem IP livre (o time é avisado). Verificar nunca aloca IP.
- **`delete-sending-domain`**: remove o domínio. É recusado enquanto algum projeto envia por ele ou rastreia links no host dele. Confirme antes.

## Criar projeto

**`create-project`** cria um projeto que envia pelo ESP. O remetente precisa estar em domínio verificado da conta (ou compartilhado com ela). O `pool_id` vem de `list-sending-pools` com `for_new_project=true`: opcional quando só há um pool, obrigatório quando há vários.

**Conta sem pool precisa de `pool_name`**: pergunte à pessoa como chamar o pool, nunca invente. O pool nasce com esse nome exato e recebe **o primeiro IP dedicado da conta, cobrado por mês** (o mês da alocação conta inteiro se o IP ficar mais de 24h). Confirme antes. `pool_name` é recusado quando a conta já tem pool. O `id` devolvido é o `project_id` das outras tools.

## IPs, pools e reverse DNS

- `list-sending-ips`: IPs da enterprise com os pools, o hostname de reverse DNS (PTR) e o status. `team_managed=true` quer dizer que o time da Everinbox ajusta PTR e pools daquele IP.
- `list-sending-pools`: os pools próprios e os compartilhados por outra enterprise (`shared=true`), com `ip_count`. Só pool próprio recebe IP pedido ou movido, e pool `team_managed` não recebe nenhum.
- **`request-sending-ip`**: aloca **mais um IP dedicado, cobrado por mês** (IP mantido por mais de 24h paga o mês inteiro da alocação e todos os seguintes enquanto a conta o mantiver). Cada chamada é um IP. Confirme com a pessoa antes de cada chamada. Informe exatamente um entre `pool_id` (pool próprio) e `new_pool_name`. Gere um `request_id` novo (UUID v4) por pedido novo e **reuse o mesmo** ao repetir uma chamada cuja resposta se perdeu: assim o retry devolve o IP que a primeira chamada pegou em vez de alocar outro. 409 quer dizer sem IP livre no momento (o time é avisado), nome de pool já usado ou pool `team_managed`; 400, pool compartilhado por outra enterprise. Depois, configure o PTR.
- **`set-sending-ip-pools`**: define por quais pools próprios um IP envia. A lista **substitui** os pools atuais do IP: inclua os que ele deve manter. Lista vazia deixa o IP sem pool da conta, ainda cobrado. Nunca cria pool nem cobra IP novo, mas **os projetos de todos os pools do IP compartilham a reputação dele**: confirme com a pessoa. Sem liberação de gestão de IP na enterprise responde 403 `IP_MANAGEMENT_NOT_ENABLED`. 409 quando o IP ou um pool é `team_managed`, ou quando a mudança deixaria um pool de projeto sem nenhum IP.
- `set-ip-reverse-dns`: sem `hostname`, o IP mantém o hostname da Everinbox e a Everinbox publica os dois registros. Com `hostname` (subdomínio de domínio **verificado** da enterprise) o cliente precisa publicar o registro A mostrado na resposta; `auto_published` diz se a Everinbox publicou o PTR sozinha.
- `verify-ip-reverse-dns`: confere ao vivo (A resolve para o IP e o PTR para o hostname) e grava o resultado.

Remover IP não tem tool: é feito pelo suporte da Everinbox.
