# Promises js explanation

[![Creative Commons License](https://i.creativecommons.org/l/by-sa/4.0/88x31.png)](http://creativecommons.org/licenses/by-sa/4.0/)

This work is licensed under a [Creative Commons Attribution-ShareAlike 4.0 International License](http://creativecommons.org/licenses/by-sa/4.0/).

Una función puede devolver un valor

```js
function sumar(v1, v2) {
    return v1 + v2
}
console.log(sumar(1, 2))
```
<!-- tmc
3
-->

A veces obtener ese valor tarda tiempo. Por ejemplo si dependemos de una conexión a un servidor.
Entonces en ese caso se proveía un callback para ser llamado cuando terminaba.
```js
require('https').get('https://www.google.com.ar/', function(res) {
    console.log(res.statusCode)
})
```
<!-- tmc
200
-->

Esto funciona, pero tiene varios problemas. Por un lado, si queremos hacer más de una cosa cuando
termina tenemos que hacer una función que conozca todo lo que hay que hacer. También si hay que
hacer varias cosas en serie queda una como callback de otra como callback de otra y el código
se hace difícil de leer.

Entonces ahí nace el concepto de Promesa. La Promesa representa una promesa de un valor futuro. Es
decir que la operación no te devuelve el valor que querés sino un objeto que eventualmente tendrá
ese valor.

Una implementación simplificada de Promise es algo así:


```js
var https = {
    get: function(url) {
        var promise = {
            resolved: false,
            value: null,
            thens: [],
            resolve: function(value) {
                this.value = value
                this.resolved = true
                for (var i = 0; i < this.thens.length; i++) {
                    this.thens[i](value)
                }
            },
            then: function(callback) {
                if (this.resolved) {
                    callback(this.value)
                } else {
                    this.thens.push(callback)
                }
            }
        }
        require('https').get(url, function(res) {
            promise.resolve(res)
        })
        return promise
    }
}
var promise = https.get('https://www.google.com.ar/')
promise.then(function(res) {
    console.log(res.statusCode)
})
promise.then(function(res) {
    console.log(res.headers['content-type'])
})
```
<!-- tmc
200
text/html; charset=ISO-8859-1
-->

Promise fue agregado a Javascript por lo que no hay que hacer todo eso.
Una función puede crear una promesa. Al hacerlo recibe dos funciones, una para llamar cuando tiene
éxito y una para llamar cuando falla. Quien espera el valor puede agregar callbacks al éxito con
`then` y al fallo con `catch`

```js
const get = function(url) {
  return new Promise((resolve, reject) => {
    const lib = url.startsWith('https') ? require('https') : require('http');
    const request = lib.get(url, (response) => {
      if (response.statusCode < 200 || response.statusCode > 299) {
        reject(response);
      }
      resolve(response)
    });
    request.on('error', (err) => reject(err))
  })
};


function run() {
  get('https://www.google.com.ar/')
    .then((res) => console.log(res.statusCode))
    .catch((err) => console.error(err));
}
run()
```
<!-- tmc
200
-->

Las promesas se pueden encadenar para que una se resuelva con el valor de una anterior.

```js
const suma = function(a, b) {
  return new Promise((resolve, reject) => { resolve(a+b) })
};

const resta = function(a, b) {
  return new Promise((resolve, reject) => { resolve(a-b) })
};

function run() {
  suma(1, 2)
    .then((res) => {
      console.log('el resultado de la suma es', res)
      return resta(res, 5)
    })
    .then((res) => {
      console.log('el resultado de la resta es', res)
    })
}
run()
```
<!-- tmc
el resultado de la suma es 3
el resultado de la resta es -2
-->


Esta sintaxis se puede hacer aún más linda con async y await. Esto fue agregado recientemente y
puede no estar disponible en viejas plataformas.

```js
const get = function(url) {
  return new Promise((resolve, reject) => {
    const lib = url.startsWith('https') ? require('https') : require('http');
    const request = lib.get(url, (response) => {
      if (response.statusCode < 200 || response.statusCode > 299) {
        reject(response);
      }
      resolve(response)
    });
    request.on('error', (err) => reject(err))
  })
};

async function run() {
    try {
      const res = await get('https://www.google.com.ar/')
      console.log(res.statusCode)
    } catch (err) {
      console.error(err)
    }
}

run()

```
<!-- tmc
200
-->

Este ejemplo hace exactamente lo mismo que el ejemplo anterior, pero usando la sintaxis nueva.

`await` traduce automáticamente el código que continúa como si fuese código a ser ejecutado en
el `then` de la Promise.

Sólo se puede llamar a `await` dentro de una función marcada como `async`.

La función al ser `async` hace que su valor de retorno sea automáticamente una Promise, porque
si hubo una llamada a `await` se necesita que ella hubiese concluido para tener su propio valor.

```js
async function sumar(a, b) {
    return a+b
}
sumar(1, 2).then((res) => console.log(res))
```
<!-- tmc
3
-->

También se pueden combinar varias Promise en una sola para esperar a que todas terminen.

```js
async function sumar(a, b) {
    return a+b
}
async function restar(a, b) {
    return a-b
}
async function esperar(time) {
    return new Promise((resolve) => setTimeout(() => resolve(), time))
}
async function run() {
    const [suma, resta, _] = await Promise.all([sumar(1, 2), restar(1, 2), esperar(1)])
    console.log(suma, resta)
}
run()
```
<!-- tmc
3 -1
-->

Y encadenar cosas es más fácil con async/await!
```js
const suma = async function(a, b) {
  return a+b
};

const resta = async function(a, b) {
  return a-b
};

async function run() {
  const res_suma = await suma(1, 2)
  console.log('el resultado de la suma es', res_suma)
  const res_resta = await resta(res_suma, 5)
  console.log('el resultado de la resta es', res_resta)
}
run()
```
<!-- tmc
el resultado de la suma es 3
el resultado de la resta es -2
-->
