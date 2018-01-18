---
layout: post
title:  "Revisitando Set em JavaScript, pegadinha e truques"
description: XXXX.
date: 2018-01-08 06:00:00 -0300
categories:
permalink: /revisitando-set-javascript-pegadinha-truques/
author: flavio_almeida
tags: [javascript, set, data structure, tips, tricks, truques, conjunto]
image: logo.png
---

Precisamos criar um **conjunto** com 20 números aleatórios de um a 100. Primeiro, vamos declarar a função `generateRandomInt` que será a responsável pela geração de números aleatórios na faixa que indicarmos:

```javascript
const generateRandomInt = (min, max) => 
    Math.floor(Math.random() * (max - min + 1)) + min;
```
Agora, através de `Array.from`, criaremos um novo array já preenchido com números aleatórios:

```javascript
const generateRandomInt = (min, max) => 
    Math.floor(Math.random() * (max - min + 1)) + min;

const numbers = Array.from(new Array(20), () => 
    generateRandomInt(1, 100));

console.log(numbers);
```
O código anterior só foi possível porque o segundo parâmetro da função `Array.from` é uma função de *mapeamento*, isto é, será aplicada em todos os elementos `undefined` do Array criado por `new Array(20)` resultando em um array com apenas números aleatórios. Todavia, se escrutinarmos o valor de `numbers`, veremos que potencialmente podemos ter números duplicados:

```javascript
54,70,27,86,29,80,68,21,25,81
49,93,49,39,77,86,47,9,87,95
```
Essa duplicação não faz sentido pois um conjunto possui **elementos distintos**.

>*<a href="https://en.wikipedia.org/wiki/Set_(mathematics)" target="blank">In mathematics, a set is a collection of distinct objects"</a>*

Precisamos alterar nosso código para que se coadune com a teoria dos conjuntos!

## Primeira solução

Uma solução clássica é verificar se o número existe no array antes de ser adicionado.

```javascript
const generateRandomInt = (min, max) => 
    Math.floor(Math.random() * (max - min + 1)) + min;

let counter = 10;
const numbers = [];

while(counter > 0) {
    const randomNumber = generateRandomInt(1, 100);
    if(!numbers.some(number => number === randomNumber)) {
        numbers.push(randomNumber);
        counter++;
    } 
}   
```

Noss solução funciona, todavia nada disso precisaria ser feito se tivéssemos utilizado a estrutura de dados `Set` (conjunto) introduzida no ES2015 (ES6).

## Utilizando Set 

Vamos alterar o código anterior para que faça uso do `Set`. Criamos um conjunto através da instrução `new Set()` e através do método `add` da instância retornada adicionamos novos elementos. Todavia, caso o elemento já exista no conjunto, ele não será adicionado novamente: 

```javascript
const generateRandomInt = (min, max) => 
    Math.floor(Math.random() * (max - min + 1)) + min;

const numbers = new Set();
while(numbers.size < 20) 
    numbers.add(generateRandomInt(1, 100));
console.log(numbers);
```

A condição `numbers.size < 20` é necessária para que tenhamos 20 elementos, pois o número gerado já pode ter sido incluído e, como vimos, o método `add()` não permitirá sua inclusão.

Consultar a propriedade `numbers.size` pode não ser tão performático caso a lista tenha um tamanho considerável, nesse sentido vamos utilizar outra estratégia.

Dessa vez, verificaremos se  elemento existe antes de incluído, pois precisamos dessa informação para saber se incrementamos ou não nosso contados. Verificamos a existência através do método `has()` que retorna `true` ou `false` caso o elemento exista ou não:

Vamos ao código:

```javascript
const generateRandomInt = (min, max) => 
    Math.floor(Math.random() * (max - min + 1)) + min;

const numbers = new Set();
let counter = 1;
while(counter <= 20) {
    const randomNumbers = generateRandomInt(1, 100);
    if(!numbers.has(randomNumbers)) {
        numbers.add(randomNumbers);
        ++counter;
    }  
}  
console.log(numbers);
```

Opcionalmente, podemos omitir a instrução `if` da seguinte forma:

```javascript
const generateRandomInt = (min, max) => 
    Math.floor(Math.random() * (max - min + 1)) + min;

const numbers = new Set();
let counter = 1;
while(counter <= 20) {
    const randomNumbers = generateRandomInt(1, 100);
    !numbers.has(randomNumbers)
    && numbers.add(randomNumbers) 
    && ++counter;
}  
console.log(numbers);
```

A estrutura `Set` possui <a href="https://developer.mozilla.org/pt-BR/docs/Web/JavaScript/Reference/Global_Objects/Set" target="_blank">outros métodos</a>, porém vale destacar `forEach` e `clear`. O primeiro itera no conjunto na **ordem de inclusão** e o segundo remove todos os seus elementos:

```javascript
// código anterior omitido
numbers.forEach(number => console.log(number)); // imprime cada item 
numbers.clear(); // esvaziou o conjunto
console.log(numbers); // Set(0) {}
```

Realizamos um *overview* sobre `Set` em JavaScript, no entanto, há uma pegadinha que pode pregar uma peça no programador desaviado.

## Pegadinha no uso de Set

Temos um conjunto de livros:

```javascript
const livros = new Set([
    { nome: 'Cangaceiro JavaScript', isbn: '9788594188014' },
    { nome: 'MEAN', isbn: '9788555190469' },
]);
```

Vamos adicionar um livro repetido para logo em seguida verificarmos o tamanho do `Set`:

```javascript
const livros = new Set([
    { nome: 'Cangaceiro JavaScript', isbn: '9788594188014' },
    { nome: 'MEAN', isbn: '9788555190469' },
]);

const livro = { nome: 'MEAN', isbn: '9788555190469'})
livros.add(livro);

console.log(livros.size); // 3!!!
```

Para a supressa de alguns, o método `add()` adicionou um livro duplicado! Mesmo se tivéssemos utilizado o método `has()` para verificar se o livro já existe ou não ele retornararia `false`. Por que isso acontece?

A explicação é a mesma que este autor utilizou para explicar o comportamento do método `Array.includes` no artigo <a href="http://cangaceirojavascript.com.br/array-includes-vs-array-some/" target="_blank">"Array.includes vs Array.some"</a>. Vamos recordar. 

Internamente o método `has` e `add` utilizam o operador `===` para comparar o novo elemento com os já existentes. Essa comparação será feita com base no valor primitivo dos seguintes tipos:

* Boolean
* Null
* Undefined
* Number
* String
* Symbol (ES2015)

Todavia, quando ocorre uma comparação com um elemento do tipo `Object`, o interpretador verificará se ambos apontam para o mesmo endereço de memória. Sendo assim, por mais que tenhamos criado um novo objeto com estrutura idêntica ao objeto do nosso conjunto, eles serão diferentes.

Voltando ao exemplo anterior, se tentarmos adicionar o mesmo livro que acabamos de adicionar, não conseguiremos, pois a variável de referência `livro` aponta para o mesmo objeto que inserimos em nosso conjunto:

```javascript
const livros = new Set([
    { nome: 'Cangaceiro JavaScript', isbn: '9788594188014' },
    { nome: 'MEAN', isbn: '9788555190469' },
]);

const livro = { nome: 'MEAN', isbn: '9788555190469'})

livros.add(livro);
console.log(livros.size); // 3!!! Incluiu!

livros.has(livro); // true, existe no conjunto
livros.add(livro); // não adiciona novamente
console.log(livros.size); // continua 3
```

Agora que passamos pela pegadinha que o `Set` pode nos pregar, que tal aprendermos dois truques que podemos utilizar no dia a dia?

## Truque 1: transformando uma lista em conjunto

Precisamos do total de ID's não duplicados de um `Array`. Uma maneira de resolver o problema é criando um novo `Array` com elementos não duplicados:

```javascript
const ids = [100, 110, 100, 200, 300, 100];
const uniqueIds = [];

ids.forEach(id => 
    !uniqueIds.some(uniqueId => 
        uniqueId === id) && uniqueIds.push(id));

console.log(uniqueIds).length;
```

Todavia, podemos criar um `Set` apartir de um array passando para seu `constructor` um `Array`. 

```javascript
const ids = [100, 105, 100, 200, 300, 110];
const uniqueIds = new Set(ids);
console.log(uniqueIds.size); // 5
```

Excelente, um código muito mais enxuto. 

## Truque 2: transformando um conjunto em uma lista

Vimos como descobrir a quantidade de elementos únicos dentro de uma lista convertendo-a para um conjunto. Mas se precisarmos filtrar ou realizar algum mapeamento nos valores do conjunto? Essas operações existem apenas no tipo `Array`. E agora?

A boa notícia é que através do *Spread Operator* podemos transformar facilmente um conjunto (Set) em uma lista (Array):

```javascript
const ids = [100, 105, 100, 200, 300, 110];
const filteredIds = [...new Set(ids)]
    .filter(uniqueId => uniqueId > 100);

console.log(filteredIds); 
// [105, 200, 300, 110]
```

Essa e a dica anterior simplificam demais a transformação de um conjunto para uma lista e de uma lista para conjunto.

## Conclusão
