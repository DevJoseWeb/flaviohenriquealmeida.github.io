---
layout: post
title:  "map x flatMap, entendendo a diferença na prática"
description: Neste artigo veremos a diferença entre as funções map() e flatMap(), inclusive como implementar esta última para que possamos escrever um código ainda mais elegante.
date:   2017-10-31 08:00:00 -0300
categories:
permalink: /map-flatmap-entendendo-diferenca-na-pratica/
author: flavio_almeida
tags: [map, flatMap, functional, programming, javascript, ecmascript]
image: logo.png
---
Neste artigo veremos a diferença entre as funções `map()` e `flatMap()`, inclusive como implementar esta última para que possamos escrever um código mais elegante. Vamos ao problema.

## Um problema

Temos uma lista de notas fiscais e cada nota possui uma lista de itens:

```javascript
const notas = [
    {
        data: '2017-10-31',
        itens: [
            { conta: '2143', valor: 200 },
            { conta: '2111', valor: 500 }
        ]
    },
    {
        data: '2017-07-12',
        itens: [
            { conta: '2222', valor: 120 },
            { conta: '2143', valor: 280 }
        ]
    }, 
    {
        data: '2017-02-02',
        itens: [
            { conta: '2143', valor: 110 },
            { conta: '7777', valor: 390 }
        ]
    },     
];
```
Precisamos totalizar o valor de todos os itens com a conta `2143`. E agora?

## Tentando solucionar com map()

Podemos apelar para a função `map()`, aquela que permite aplicar uma transformação em cada elemento do array resultando em um novo array. Nesse sentido, queremos transformar cada nota iterada em um array de itens para que possamos realizar `filter` e `reduce` para filtrar e totalizar respectivamente:

```javascript
// não funciona como esperado! O resultado será 0!
const totalDeUmaConta = 
    notas.map(nota => nota.itens)
        .filter(item => item.conta == '2143')
        .reduce((total, item) => total + item.valor, 0);

console.log(totalDeUmaConta);      
```

Infelizmente o código acima não funciona, porque o resultado de `map()` será um multidimensional, ou seja, um array de arrays:

```javascript
// resultado do map
[
    [
        { conta: '2143', valor: 200 },
        { conta: '2111', valor: 500 }
    ],
    [
        { conta: '2222', valor: 120 },
        { conta: '2143', valor: 280 }
    ],
     [
        { conta: '2143', valor: 110 },
        { conta: '7777', valor: 390 }
    ]
]
```

Para que a função `filter()` funcione como esperado, o array deveria ter uma estrutura unidimensional:

```javascript
// estrutura esperada por filter
[
    { conta: '2143', valor: 200 },
    { conta: '2111', valor: 500 },
    { conta: '2222', valor: 120 },
    { conta: '2143', valor: 280 },
    { conta: '2143', valor: 110 },
    { conta: '7777', valor: 390 }   
]
```

Uma solução é abdicarmos do `map` e utilizarmos `reduce` para **achatarmos** o array multidimensional em um array de uma dimensão apenas. Vejamos:

```javascript
// Agora sim!

totalDeUmaConta = notas
    .reduce((array, nota) => array.concat(nota.itens), [])
    .filter(item => item.conta == '2143')
    .reduce((total, item) => total + item.valor, 0);

console.log(totalDeUmaConta);
```

Se o leitor compreendeu o motivo pelo qual o primeiro `reduce` foi necessário, já sabe a finalidade do `flatMap`, até porque este `reduce` esta exercendo sua função.

## Uma solução com flatMap()

A função `flatMap` realiza um `map` de uma função sobre uma coleção de dados, porém achatando o resultado final em um nível,  isto é, retornando um array de uma dimensão apenas. Infelizmente não há implementação desta função no JavaScript, mas nada nos impede de criá-la. 

Podemos adicionar a função `flatMap` no `prototype` de `Array`. Aliás, este autor já fez algo semelhante no artigo <a href="http://cangaceirojavascript.com.br/como-embaralhar-arrays-algoritmo-fisher-yates/" target="_blank">"Como embaralhar arrays com o algoritmo Fisher–Yates"</a>:

```javascript 
Array.prototype.flatMap = function(cb) {

    return this.map(cb).reduce((destArray, array) => 
        destArray.concat(array), []);
}

totalDeUmaConta = notas
    .flatMap(nota => nota.itens)
    .filter(item => item.conta == '2143')
    .reduce((total, item) => total + item.valor, 0);

console.log(totalDeUmaConta);
```

## Conclusão

A função `flatMap()` esta incluída nas práticas da programação funcional. Apesar de o JavaScript não implementá-la podemos materializá-la com pouco esforço tornando nosso código mais elegante do que já é. 

E você? Consegue enxergar outra situação na qual a função `flatMap` o pouparia de muito esforço? Deixe sua opinião!