# Missão Indicação — Colégio CPPEM

Página do programa de indicação do Colégio. Uma pessoa indica outra, e as duas
aparecem na **mesma linha** da aba `INDICAÇÕES` da planilha de leads, com
`COLEGIO` na coluna BU.

Irmã da `indicaçõesCPPEM`: mesma estrutura, mesmo HTML, mesmo CSS. O que muda é
o bloco `BU` no `script.js`, os tokens de cor e o texto dos benefícios.

```
index.html     a página inteira (HTML + CSS)
script.js      validação, envio e o fluxo de "indicar outra família"
public/        brasão do Colégio
vercel.json    cache dos estáticos
```

---

## 1. Backend: nada a fazer

O Apps Script é **um só** para as três BUs, já está publicado e já reconhece
`INDICACAO_COLEGIO`. Esta pasta **não tem cópia dele de propósito** — duas
cópias do mesmo backend viram duas versões divergentes na primeira correção.

O arquivo vive em `../indicaçõesCPPEM/google-apps-script.js`, e o passo a passo
de publicação está no README de lá.

O que chega na planilha:

| campo | coluna |
|---|---|
| data | A — Data |
| nome | B — Nome |
| telefone | C — Telefone |
| chave_pix | D — Chave pix indicador → **vazia nesta BU**, ver abaixo |
| nome_indicado | E — Nome Indicado |
| telefone_indicado | F — Telefone Indicado |
| bu | G — BU → `COLEGIO` |
| url | H — Página URL |

---

## 2. Publicar a página

Site estático. Na Vercel, importar o repositório e publicar sem build.

Domínio previsto, já reconhecido pelo backend caso o `?aba=` se perca num link
compartilhado: `indique.colegio.cppem.com.br`.

Publicar em outro domínio funciona — o `?aba=` decide sozinho. Mas se mudar,
vale acertar a lista `DOMINIOS` no Apps Script.

---

## 3. O bloco BU

É o único ponto obrigatório ao replicar. Tudo que varia entre as três páginas
mora aqui:

```js
const BU = {
  chave: "INDICACAO_COLEGIO",   // decide a coluna BU no backend
  nome: "COLEGIO",              // vai junto nos eventos de dataLayer
  whatsapp: "5581997076388",
  whatsappMsg: "...",
  pedirChavePix: false,         // ver abaixo
  selo: { ate: 100, prefixo: "R$ ", sufixo: "" }
};
```

### Por que a chave PIX está desligada aqui

No CPPEM a recompensa é paga em dinheiro, então a chave PIX é obrigatória. No
Colégio os benefícios são **desconto na mensalidade e fardamento**, aplicados
pela própria escola — não existe pagamento por PIX. Pedir a chave seria atrito
num formulário de um minuto, em troca de um dado que ninguém usaria.

O campo continua no `index.html`, escondido e fora da validação. Trocar
`pedirChavePix` para `true` devolve tudo, sem mais nenhuma alteração.

**Se o programa passar a pagar algo em dinheiro, vire essa linha.** A coluna
`Chave pix indicador` fica vazia nas linhas de BU `COLEGIO` até lá — é esperado,
não é falha.

### O selo animado

`selo` controla o círculo da faixa "e para quem você indicar". O anel fecha a
volta inteira enquanto o número sobe até `ate`. No CPPEM é `10` + `%`; aqui é
`100` com prefixo `R$ `. O valor **não** está no HTML — ele traz só o estado
inicial, para quem tem animação desligada no sistema.

---

## 4. Identidade visual

Bloco `:root` no topo do `<style>` — é o único lugar onde as cores são
definidas.

| | Colégio | CPPEM |
|---|---|---|
| fundo | `#0D1B3E` (navy) | `#0a0a0b` (preto) |
| destaque | `#C9A227` | `#af9256` |
| display | Cinzel | Oxanium |
| texto | DM Sans | Inter |

Duas coisas fora dos tokens:

- **A grade dourada** (`.bg-grade`) é a assinatura das páginas do Colégio,
  herdada do `Contato-Colegio`. Não existe na página do CPPEM.
- **O brasão de fundo** do hero usa o próprio logo do Colégio. O escudo é
  preenchido de azul-marinho, quase a cor do fundo, então em opacidade baixa
  sobra só o contorno dourado, o leão e os louros — que é exatamente o efeito
  de marca d'água desejado. A página do CPPEM usa o emblema do leão isolado.

Um detalhe que custou um teste: **`ª` em Cinzel maiúsculo** vira um "A"
sobrescrito estranho. Por isso o título da seção é "A partir da terceira, não
para" em vez de "da 3ª". Nos chips, que são DM Sans, o `3ª` sai certo e ficou.

---

## 5. O formulário

Quatro campos, todos obrigatórios (cinco se `pedirChavePix` for ligado):

| campo | validação |
|---|---|
| Seu nome completo | nome e sobrenome |
| Seu WhatsApp | DDD válido + 9 dígitos, o 9 na terceira posição |
| Nome do responsável indicado | nome e sobrenome |
| WhatsApp do responsável | mesma regra, e não pode ser igual ao seu |

O envio espera a requisição sair antes de mostrar sucesso (`await`), ao
contrário das LPs de captura, que disparam e redirecionam. Como a pessoa
**fica na página**, mostrar "enviado" sem ter enviado seria mentira visível na
próxima indicação.

### Indicar mais de uma família

Na tela de sucesso, **"Indicar outra família"** limpa apenas os dois campos do
indicado e devolve o formulário com nome e WhatsApp do indicador ainda
preenchidos. A partir da segunda aparece um contador — ele é local (só desta
sessão) e serve de confirmação visual. O placar que vale é o da escola,
conferido na confirmação da matrícula.

---

## 6. A seção "O que você está indicando"

Quem indica precisa de duas coisas: saber o que ganha e saber que pode botar a
cara no fogo. Os níveis resolvem a primeira; esta seção resolve a segunda —
sem ela a página só fala de prêmio, e indicar escola para um amigo é uma
decisão de reputação, não de prêmio.

O conteúdo veio todo de material que já existe, nada foi inventado:

| o quê | de onde |
|---|---|
| 8 fotos da escola (`public/colegio/`) | `pages captura leads/convitecolegio/public/colegio` — já reduzidas para 880×660 em webp, 368 KB as oito |
| textos alternativos das fotos | os mesmos do `convitecolegio`, mantidos palavra por palavra |
| "Colégio cristão com disciplina militarizada" | `SiteColegioCPPEM/components/features/Hero.tsx` |
| "Do 1º ano do Ensino Fundamental ao 3º ano do Ensino Médio" | idem |
| "Educação cristã, hierarquia, formação moral e preparação para concursos públicos desde a base escolar" | `SiteColegioCPPEM/app/page.tsx` (JSON-LD) |
| "Disciplina, fé e excelência acadêmica no mesmo lugar" | `convitecolegio/index.html` |
| @colegiocppem | `convitecolegio/index.html` |

As fotos foram **copiadas**, não referenciadas do outro deploy — mesma decisão
que o `convitecolegio` documenta para os depoimentos dele: a página não pode
quebrar se o outro projeto mudar de rota.

A escolha das legendas não é aleatória. **Cavalaria, Tatame e Xadrez** vêm
antes de Biblioteca e Auditório de propósito: são o que diferencia a escola de
qualquer outra e o que faz alguém lembrar dela numa conversa. Biblioteca toda
escola tem.

Detalhe de layout: oito fotos numa grade de quatro colunas, com a primeira
ocupando 2×2, deixam **uma célula vazia** no fim da última linha. Por isso a
última foto ocupa as duas que sobram — e por isso ela tem `aspect-ratio` 8/3 em
vez de 4/3, para a altura bater com as vizinhas.

---

## 7. O que NÃO está na página

O **checklist interno de entrega dos benefícios** (conferir matrícula + nível,
separar fardamento, registrar entrega, aplicar desconto) ficou de fora de
propósito: é procedimento da equipe, não informação para quem indica. Colocá-lo
numa página pública só criaria expectativa sobre um processo que a família não
controla.

---

## 8. Rastreamento

GTM server-side (`sgtm.cppem.com.br`), igual às outras páginas. Eventos no
`dataLayer`, todos com `bu: "COLEGIO"`:

| evento | quando |
|---|---|
| `indicacao_enviada` | envio aceito, com `indicacao_numero` |
| `indicacao_erro` | a requisição não saiu |
| `indicacao_nova_tentativa` | clique em "Indicar outra família" |

Não há evento de `Lead` aqui: indicação não é lead de venda e contaria como
conversão nas campanhas.

---

## O carrossel

As três páginas usam o mesmo carrossel infinito, com a mesma classe
`.carrossel`. Uma correção feita numa transfere para as outras.

Como funciona: duas cópias da mesma fita, lado a lado, e a animação arrasta o
conjunto para a esquerda. Quando a primeira cópia acaba de sair, a segunda
está exatamente onde a primeira começou, então o salto de volta ao início não
se vê.

O deslocamento é **`-50% - metade do gap`**, não `-50%` puro. Com duas cópias
separadas por um gap, metade da largura total cai no meio desse espaço, e a
emenda daria um solavanco a cada volta. Medido no navegador nas três páginas,
o deslocamento do CSS bate com a largura de uma cópia inteira na casa do
centésimo de pixel.

Outros detalhes que não são óbvios:

- A segunda cópia é `aria-hidden`. Sem isso, um leitor de tela leria a mesma
  lista duas vezes.
- As fotos individuais têm `alt=""` e quem carrega a descrição é o contêiner,
  com `role="img"` e um `aria-label` do conjunto. São várias fotos da mesma
  cena; descrever uma a uma só encheria o leitor de tela de repetição.
- A segunda cópia usa os **mesmos `src`**, então o navegador não baixa nada de
  novo: 368 KB as 8 fotos no total.
- Passar o mouse pausa. `:focus-within` também, para quem navega por teclado.
- Com "reduzir movimento" ligado no sistema, a animação some, a fita vira uma
  faixa rolável com scroll-snap e a cópia duplicada é escondida.

A velocidade fica no `style="--duracao:"` do próprio elemento, para cada página
ajustar sem mexer no CSS: 44s aqui, proporcional à quantidade de fotos.
