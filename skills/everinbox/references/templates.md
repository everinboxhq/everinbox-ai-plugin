# Templates, conteúdo e imagens


Obrigatórios: `project_id`, `name`, `html`, `subject`, `sender_name`. A enterprise é auto-preenchida.

**Sobre o `design` (JSON do editor Unlayer): não envie.** Mandando só `html`, o design é gerado — um bloco único que o editor abre normalmente. Só passe `design` quando você tem um layout autoral do editor que precisa preservar literalmente (copiado de um `get-template`).

**A resposta do create/update é enxuta e não devolve `html` nem `design`.** Para ler o conteúdo, `get-template`.

## Merge tags: rode `list-merge-tags` antes de escrever o HTML

A personalização é por tag entre colchetes (`[nome]`, `[email]`, `[data_de_hoje]`), dentro de `html`, `subject` ou `pre_header`.

**As tags disponíveis são do projeto, não universais.** `list-merge-tags` devolve as que existem de verdade ali — as de sistema **e** os campos customizados daquele projeto. Chame antes de gerar qualquer HTML: é o que impede inventar `[nome_completo]` ou `[primeiro_nome]`, que não existem e saem impressas literalmente no email.

A resposta vem em dois grupos:

- **`system`** — tags da plataforma. Algumas têm versão por idioma, e o `locale` indica o idioma do **valor renderizado**: `[dia_hoje]` → "segunda-feira", `[day_today]` → "Monday", `[dia_actual]` → "lunes". Tags com o mesmo `concept` apontam para a mesma informação em idiomas diferentes — **escolha a que bate com o idioma do template**, senão o email sai misturado. Tag sem `locale` rende igual em qualquer idioma (email, IDs, UTMs, dado cru do contato).
- **`custom`** — campos criados pelo projeto, com o tipo e, em select/multi_select, as opções válidas.

Algumas tags trazem **`recommended_html`**, um bloco pronto — é o caso do rodapé de descadastro. Use esse bloco como base em vez de escrever o seu.

**O descadastro é obrigatório.** Template sem ele é problema de entregabilidade e de lei — sinalize antes de deixar seguir.

Ao explicar as tags para a pessoa, use linguagem natural ("o primeiro nome do contato"), não os nomes técnicos de categoria ou tipo de campo.

`pre_header` é o texto de preview no cliente de email. Vale a pena: é a segunda linha que a pessoa lê.

## Imagens: a galeria do projeto

`list-gallery-items` lista as imagens do projeto com busca por nome (`search`), filtro por `tags`, `mime_type` e data, e ordenação por `recently_used`, `recently_uploaded`, `most_used` ou `size_desc`. Cada item traz `url`, dimensões, tamanho, `used_count` e `last_used_at`.

Antes de subir imagem nova, **procure na galeria** — reaproveitar evita duplicata e mantém o peso do email baixo.

`get-gallery-item-usage` mostra onde uma imagem está sendo usada. **Rode antes de apagar ou trocar**: imagem em uso em template ativo quebra o email de quem receber depois.

`upload-gallery-image` sobe, `update-gallery-item` edita metadados e tags, `delete-gallery-item` apaga (confirme antes). Por padrão a listagem esconde arquivados e apagados.


## Versões e remetentes

`list-template-versions` mostra o histórico de versões de um template. Importa porque **campanha aponta para uma versão específica** (`template_version`): editar o template não muda o que uma campanha já agendada vai enviar, e uma campanha antiga pode estar presa numa versão velha.

`list-template-senders` lista os remetentes já usados no projeto. Vale consultar antes de inventar um `sender_email` — remetente novo não tem histórico de reputação.

`duplicate-template` copia; `duplicate-templates-batch` copia em lote; `rename-templates-batch` renomeia em lote. `delete-template` apaga — **rode `get-template-usage` antes**, porque template referenciado por campanha agendada ou automação ativa não deve sumir.

## Como o template está performando

| Pergunta | Tool |
|---|---|
| Como esse template performa no geral? | `get-template-metrics` |
| Onde ele é usado? | `get-template-usage` |
| Qual link está levando o clique? | `get-template-clicks-by-url` |
| Uma versão é melhor que a outra? | `get-template-versions-metrics` |
| Ele está desgastado? | `get-template-saturation` |

`get-template-clicks-by-url` é o mais subutilizado: clique agregado esconde que 90% vai para um único link. Quando alguém pergunta "por que o CTR é bom mas ninguém converte", a resposta costuma estar aí.

## Saturação: quando o vencedor cansa

`get-template-saturation` mede o quanto um template está **queimado** de tanto ir para a mesma audiência — a fadiga do vencedor.

O padrão é conhecido: o template campeão vira o default, roda toda semana, e o engajamento decai devagar o bastante para ninguém notar. Saturação alta é o sinal de que ele precisa **descansar ou ser renovado**, não ser disparado de novo igual.

Duas saídas quando a saturação está alta: rotacionar a criação, ou expandir as variantes de Spintax do template.

## Spintax: variação que protege a entrega

Spintax é notação de variação no texto: `{Oi|Olá|E aí}`. A cada envio a plataforma sorteia uma opção, então cada email sai diferente.

Isso não é firula de copy — é entregabilidade. **Assunto e remetente idênticos fazem o Gmail agrupar os envios numa mesma conversa**, e essa pilha deriva para Promoções ou Spam. Variar quebra o agrupamento e protege a taxa de abertura.

**A doutrina: 3 ou mais variantes tanto no `subject` quanto no `sender_name`**, para qualquer template que vá para a mesma audiência mais de uma vez.

- `analyze-template-variations` detecta os pontos de variação num texto (assunto, remetente, pre-header ou HTML) e devolve os grupos encontrados. Só precisa do `text` — não pede `project_id`.
- `apply-template-variations` renderiza uma versão concreta a partir das escolhas.

Ao montar template para envio recorrente, **verifique o Spintax antes de criar a campanha**. Template repetido sem variação é a causa silenciosa mais comum de abertura caindo sem motivo aparente.
