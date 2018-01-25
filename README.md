# Lambda
Tradução do livro [Mostly adequate guide to FP (in javascript)](https://github.com/MostlyAdequate/mostly-adequate-guide) para o português.

# Capítulo 05: Codificação por Composição

Isto é `compose`:

```js
const compose = (f, g) => x => f(g(x)) 

```

`f` e `g` são funções e `x` é o valor "canalizado" através delas.

Composição é uma função de criação. Você criador de funções, seleciona duas com traços que deseja combinar e as junta para gerar uma nova função.

```js
const toUpperCase = x => x.toUpperCase()
const exclaim = x => `${x}!`
const shout = compose(exclaim, toUpperCase)

shout('send in the clowns') // "SEND IN THE CLOWNS!"
```

A composição de duas funções retornam uma nova função. Isso faz todo sentido: Composição de duas unidades do mesmo tipo (neste caso funções) deve render uma nova unidade deste mesmo tipo. Você não conecta duas peças de lego juntas e tem um [lincoln log](https://en.wikipedia.org/wiki/Lincoln_Logs). Há uma teoria aqui, alguma lei subjacente que vamos descobrir no tempo certo.

Em nossa definição de compose, o `g` vai executar antes do `f` criando um fluxo da direita para a esquerda dos dados. Isto é uma forma muito mais legível de aninhar as chamadas das funções. Sem `compose` o código acima ficaria assim:

```js
const shout = x => exclaim(toUpperCase(x))
```

Em vez de dentro para fora, nos executamos as funções da direita para a esquerda, vejamos um examplo em que a sequencia importa:

```js
const head = x => x[0];
const reverse = reduce((acc, x) => [x].concat(acc), []);
const last = compose(head, reverse);

last(['jumpkick', 'roundhouse', 'uppercut']); // 'uppercut'
```

`reverse` vai retornar a lista contrário while `head` devolve o primeiro item da lista. O resultado é a função `last`. A sequência de funções deve ser aparente aqui. Podemos definir a versão da esquerda para a direita, no entanto, desta forma nos refletimos a versão matemática muito mais de perto. Esta certo, a composição vem direto dos livros de matemática. Talvez seja a hora de olhar um propriedade é válida em toda composição.

```js
// associativity
compose(f, compose(g, h)) === compose(compose(f, g), h);
```

Composição é assotiativa, isso quer dizer que não importa como você agrupa duas composições a ordem de execução sera a mesma. Estão se escolhemos deixar a palavra maiúscula, podemos escrever:

```js
compose(toUpperCase, compose(head, reverse));
// or
compose(compose(toUpperCase, head), reverse);
```

Uma vez que não importa como nos agrupamos as chamadas de `compose`, o resultado sera o mesmo. Isso nos permite escrever composições e usalas como a seguir:

```js
// previously we'd have to write two composes, but since it's associative, 
// we can give compose as many fn's as we like and let it decide how to group them.
const arg = ['jumpkick', 'roundhouse', 'uppercut'];
const lastUpper = compose(toUpperCase, head, reverse);
const loudLastUpper = compose(exclaim, toUpperCase, head, reverse);

lastUpper(arg); // 'UPPERCUT'
loudLastUpper(arg); // 'UPPERCUT!'
```

A aplicação da associação nos da flexibilidade e a paz de espírito de que o resultado sera equilvalente. A definição um pouco mais variada e complicada é incluida da biblioteca de suporte para este livro e é a definição normal que você encontrara em bibliotecas como [lodash](http://lodash.com), [underscore](http://underscorejs.org) e [ramda](http://randajs.com).

Um agradável benefício da associatividade é que qualquer grupo de funções podem ser extraídas ou juntadas em sua própria composição. Vamos brincar com a refatoração do exemplo anterior:

```js
const loudLastUpper = compose(exclaim, toUpperCase, head, reverse);

// -- or ---------------------------------------------------------------

const last = compose(head, reverse);
const loudLastUpper = compose(exclaim, toUpperCase, last);

// -- or ---------------------------------------------------------------

const last = compose(head, reverse);
const angry = compose(exclaim, toUpperCase);
const loudLastUpper = compose(angry, last);

// more variations...
```
Não há respostas certas ou erradas - Estamos apenas conectando nossos legos juntos e da maneira qu quisermos. Normalmente é melhor agrupar as funções de forma reutilizável como `last` e `angry`. Se

## Pointfree

...
