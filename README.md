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

`reverse` vai retornar a lista contrário enquanto `head` devolve o primeiro item da lista. O resultado é a função `last`. A sequência de funções deve ser aparente aqui. Podemos definir a versão da esquerda para a direita, no entanto, desta forma nos refletimos a versão matemática muito mais de perto. Esta certo, a composição vem direto dos livros de matemática. Talvez seja a hora de olhar uma propriedade é válida em toda composição.

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
Não há respostas certas ou erradas - Estamos apenas conectando nossos legos juntos e da maneira qu quisermos. Normalmente é melhor agrupar as funções de forma reutilizável como `last` e `angry`.

## Pointfree
O Estilo Pointfree significa nunca dizer quais são os seus dados. Desculpa. Isso significa que a funções nunca devem mencionar o dado o qual operam. Funções de primeira classe, currying e composição jogam juntas para criar este estilo.
```js
// not pointfree because we mention the data: word
const snakeCase = word => word.toLowerCase().replace(/\s+/ig, '_');

// pointfree
const snakeCase = compose(replace(/\s+/ig, '_'), toLowerCase);
```
Vejá como nos aplicamos parcialmente o `replace`? O que estamos fazendo é canalizando nossos dados através de cada função de 1 argumento. Currying nos ajuda a preparar nossas funções para apenas receber seus dados, operar sobre eles e passa-los adiante. Outra coisa é notar que você não precisa dos dados para construir suas funções na versão pointfree, equanto na versão pointful nos precisamos ter nossa "palavra" antes de qualquer coisa.

Vejamos outro exemplo:
```js
// not pointfree because we mention the data: name
const initials = name => name.split(' ').map(compose(toUpperCase, head)).join('. ');

// pointfree
const initials2 = compose(join('. '), map(compose(toUpperCase, head)), split(' '));

initials('hunter stockton thompson'); // 'H. S. T'
```
O código pointfree pode novamente, nos ajuda a remover nomes desnecessários e manter-nos conciso e genérico. Pointfree é um teste decisivo para programação funcional pois através dele sabemos que temos pequenas funções com entrada e saída. Não é possivel compor um loop `while` por exemplo. Estejá avisado, contudo, pointfree é uma espada de corte duplo e pode algumas vezes ofuscar a nossa intenção, nem todo código funcional é pointfree e isso é O.K. Vamos utilizar onde podemos e ficar com funções normais onde for necessário.

## Debugging

Existem alguns erros comuns no compose como o uso do `map`, uma função de dois argumentos, deve sempre primeiro aplicar o 1 parâmetro.

```js
// wrong - we end up giving angry an array and we partially applied map with who knows what.
const latin = compose(map, angry, reverse);

latin(['frog', 'eyes']); // error

// right - each function expects 1 argument.
const latin = compose(map(angry), reverse);

latin(['frog', 'eyes']); // ['EYES!', 'FROG!'])
```
Se vocẽ estiver com problemas para debugar uma composição, podemos usar essa útil, mas impura função rastreadora para ver o que esta acontecendo.
```js
const trace = curry((tag, x) => {
  console.log(tag, x);
  return x;
});

const dasherize = compose(
  join('-'),
  toLower,
  split(' '),
  replace(/\s{2,}/ig, ' '),
);

dasherize('The world is a vampire');
// TypeError: Cannot read property 'apply' of undefined
```

Algo esta errado aqui, vamos rastrear.
```js
const dasherize = compose(
  join('-'),
  toLower,
  trace('after split'),
  split(' '),
  replace(/\s{2,}/ig, ' '),
);

dasherize('The world is a vampire');
// after split [ 'The', 'world', 'is', 'a', 'vampire' ]
```
Ah! Precisamos passar o `toLower` como argumento para o `map` pois ele trabalha em uma matriz.
```
const dasherize = compose(
  join('-'),
  map(toLower),
  split(' '),
  replace(/\s{2,}/ig, ' '),
);

dasherize('The world is a vampire'); // 'the-world-is-a-vampire'
```

A função de rastremento nos ajuda a ver os dados em determinado ponto com a finalidade de debugar. Linguagens como haskell e purescript tem funções similares para facilidade do desenvolvedor.

Composição é uma ferramenta para contruir programas e, e com sorte o faria, é apoiada por uma teoria que garante que as coisas funcionem para nós. Vamos examinar essa teoria.

## Teoria das categorias
A teoria das categorias é um ramo abstrato da matemática que pode formalizar conceitos de vários ramos diferentes, como teoria de conjuntos, teoria de tipos, teoria de grupo, lógica e muito mais. Trata principalmente de objetos, morfismos e transformações, o que reflete a programação de forma bastante próxima. Aqui está um gráfico dos mesmos conceitos vistos a partir de cada teoria separada.

![categories theory](https://github.com/MostlyAdequate/mostly-adequate-guide/blob/master/images/cat_theory.png)

Desculpe, não queria assustar você. Não espero que você esteja intimamente familiarizado com todos esses conceitos. Meu objetivo é mostrar-lhe a quantidade de duplicação que temos para que você possa ver por que a teoria das categorias visa unificar essas coisas.

Na categoria teoria, temos algo chamado ... uma categoria. É definido como uma coleção com os seguintes componentes:

- Uma coleção de objetos
- Uma coleção de morfismos
- Uma noção de composição nos morfismos
- Um morfismo distinto chamado identidade

A teoria das categorias é abstrata o suficiente para modelar muitas coisas, mas vamos aplicar isso aos tipos e funções, que é o que nos importa no momento.

**Uma coleção de objetos.** Os objetos serão tipos de dados. Por exemplo, `String`, `Boolean`, `Number`, `Object`, etc. Nós geralmente visualizamos os tipos de dados como conjuntos de todos os valores possíveis. Pode-se olhar para `Boolean` como o conjunto de [true, false] e `Number` como o conjunto de todos os valores numéricos possíveis. Tratar tipos como conjuntos é útil porque podemos usar a teoria de conjuntos para trabalhar com eles.

**Uma coleção de morfismos.** Os morfismos serão nossas funções puras de cada dia.

**Uma noção de composição sobre os morfismos.** Isso, como você pode ter imaginado, é o nosso novo brinquedo - compor. Nós discutimos que nossa função de composição é associativa, o que não é coincidência, pois é uma propriedade que deve ser realizada para qualquer composição na teoria das categorias.

Esta imagem demonstra a coposição:

![without composition](https://github.com/MostlyAdequate/mostly-adequate-guide/blob/master/images/cat_comp1.png)
![composition](https://github.com/MostlyAdequate/mostly-adequate-guide/blob/master/images/cat_comp2.png)

Este é o exemplo em código:
```js
const g = x => x.length;
const f = x => x === 4;
const isFourLetterWord = compose(f, g);
```
**Um morfismo distinto chamado identidade.** Vamos introduzir uma função útil chamada id. Esta função simplesmente toma alguma entrada e a cospe de volta. Dê uma olhada:
```js
const id = x => x;
```

Você pode se perguntar "O que, no inferno, é tão útil?". Usaremos amplamente esta função nos capítulos seguintes, mas por agora pense nela como uma função que pode suportar o nosso valor - uma função que se divide em dados todos os dias.

`id` deve jogar bem com a composição. Aqui está uma propriedade que sempre é válida para cada função unária (unária: função de um argumento) f:
```js
// identity
compose(id, f) === compose(f, id) === f;
// true
```
Ei, é como a propriedade de identidade em números! Se isso não for imediatamente claro, leve algum tempo com ele. Compreenda a futilidade. Nós estaremos vendo id usado em todo o lugar em breve, mas por enquanto vemos que é uma função que atua como um suporte para um determinado valor. Isso é bastante útil quando se escreve um código sem ponto.

Então, você tem, uma categoria de tipos e funções. Se esta é a sua primeira introdução, imagino que você ainda está um pouco confuso com o que é uma categoria e por que é útil. Construiremos esse conhecimento ao longo do livro. A partir deste momento, neste capítulo, nesta linha, você pode, pelo menos, vê-lo como fornecendo-nos algum conhecimento sobre composição, ou seja, as propriedades de associatividade e identidade.

Quais são as outras categorias, você pergunta? Bem, podemos definir um para gráficos direcionados com sendo os objetos, as setas sendo morfismos e composição apenas sendo uma concatenação de caminho. Podemos definir com `Numbers` como `objetos` e >= como morfismos (na verdade, qualquer ordem parcial ou total pode ser uma categoria). Há montes de categorias, mas para os propósitos deste livro, nos ocuparemos apenas do definido acima. Nós já aramos o suficientemente a superfície e devemos seguir em frente.

## Em Suma

Composição conecta suas funções como uma serie de tubos. Os dados fluirão através da nossa aplicação tal como devem - funções puras são entradas para gerar uma saida depois de tudo, então quebrar essa cadeia iria trazer um resultado inesperado, tornando nosso software inútil.

Nós mantemos a composição como um princípio de design acima de todos os outros. Isto é porque queremos mantes nossa aplicação simples e razoável. A teoria das categorias vai desempenhar um papel importante na arquitetura de nossa aplicação, na modelagem de efeitos colaterais e garantindo a exatidão.

## Exercício

Para cada exercício, vamos considerar o objeto `Car` com a seguinte forma:

```js
{
  name: 'Aston Martin One-77',
  horsepower: 750,
  dollar_value: 1850000,
  in_stock: true,
}
```

Use `compose()` para rescrever as funções abaixo.

```js
const isLastInStock = (cars) => {  
  const lastCar = last(cars)  
  return prop('in_stock', lastCar)  
}
```
---

Considerando a função a seguir:

```js
const average = xs => reduce(add, 0, xs) / xs.length
```

Use a função `average` para refatorar `averageDollarValue` como uma composição.

---

Refatore `fastestCar` usando `compose()` e outras funções pointfree-style.

```js
const fastestCar = (cars) => {  
  const sorted = sortBy(car => car.horsepower);  
  const fastest = last(sorted);  
  return concat(fastest.name, ' is the fastest');  
}; 
```
