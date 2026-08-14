# Templates — montar o email

O template é o email: HTML + assunto + nome do remetente. A copy e a oferta vêm
do cliente; seu papel é garantir que o email seja **Gmail-friendly**, tenha as
peças obrigatórias e não caia em armadilhas de entrega.

Tool: `create-template`. Editar: `update-template`. Reusar: `duplicate-template`.
Ver o HTML já salvo: `get-template` (o retorno de create/update **não** ecoa o
HTML de volta).

## Campos e o que importa

```
create-template:
  project_id: <UUID>
  name: "<nome interno descritivo>"
  subject: "<assunto>"
  sender_name: "<nome que aparece como remetente>"
  html: "<HTML completo do email>"
  pre_header: "<texto de preview>"   // opcional, muito recomendado
```

- **Não mande o campo `design`.** A plataforma gera o JSON do editor visual a
  partir do seu `html` sozinha. Passar `design` à mão significa duplicar o HTML
  inteiro escapado dentro do JSON — caro e fácil de corromper. Só envie `design`
  se você tem um layout autoral do editor que precisa preservar igual.
- **`pre_header`** é o texto de preview que aparece na caixa depois do assunto.
  Preencha — é espaço grátis de persuasão e melhora a taxa de abertura.

## Peças obrigatórias no HTML (compliance + entrega)

Todo template precisa ter, sem exceção:

1. **Link de descadastro:** a tag `[unsubscribe]` (ou `[descadastrar]`) dentro
   do rodapé. É lei e é sinal de saúde para o provedor.
2. **Identidade do remetente:** nome e **endereço físico/legal** do cliente no
   rodapé.
3. **Links válidos:** nada de `href="#"` ou link quebrado — provedores leem isso
   como sinal ruim.

## HTML Gmail-friendly

O Gmail (e Outlook) renderiza email como HTML dos anos 2000, não como página
moderna. Para não quebrar:

- Layout em **tabelas** (`<table>`), não `flex`/`grid`.
- **CSS inline** nos elementos, não `<style>` no topo (muitos clientes ignoram).
- Largura máxima ~600px (`max-width`).
- Imagens leves e sempre com `alt`; nunca um email só de imagem (vira spam).
- Texto de verdade no corpo, não texto dentro de imagem.

## Merge tags (personalização)

Inserem dado do contato/data no assunto, corpo ou pre_header. As principais:

- Contato: `[nome]`/`[name]`, `[first_name]`, `[email]`, `[cidade]`/`[city]`.
- Data: `[data_de_hoje]`, `[dia_hoje]`, `[hora_agora]`, `[mes_de_hoje]`.
- Especial: `[unsubscribe]`/`[descadastrar]`.

Personalizar o assunto com `[first_name]` costuma ajudar abertura — mas só se o
dado existir para a maioria dos contatos; senão fica "Olá ,".

## Spintax — evitar o agrupamento do Gmail

Quando o mesmo assunto+remetente se repete, o Gmail agrupa numa thread e esconde
as repetições. Para evitar, varie assunto e `sender_name` com **spintax**:
`{opção A|opção B|opção C}`. A plataforma sorteia uma variante por contato,
então cada envio parece distinto.

```
subject: "{Sua fatura chegou|Novidade na sua conta|Dá uma olhada nisso}"
sender_name: "{Ana da Empresa|Equipe Empresa|Empresa}"
```

Ferramentas de apoio: `analyze-template-variations` (conta as combinações do
spintax) e `apply-template-variations`. Use pelo menos 3 variantes em templates
que você vai enviar repetidamente.

> A *mecânica* do spintax é do produto. A escolha de **quais** ângulos e assuntos
> testar é a estratégia do cliente — não invente ofertas por ele.

## Remetentes

`list-template-senders` lista os `sender_name` já usados no projeto — útil para
manter consistência e planejar a rotação de remetente do spintax.
