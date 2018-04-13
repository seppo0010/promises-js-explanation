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

get('https://www.google.com.ar/')
  .then((res) => console.log(res.statusCode))
  .catch((err) => console.error(err));
```
<!-- tmc
200
-->

Esta sintaxis se puede hacer aún más linda con async y await.

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
    return "finished"
}

run()
  .then((val) => console.log(val))

```
<!-- tmc
200
finished
-->

`await` hace implícitamente la parte de `then` y `catch`. La función al ser marcada como `async`
puede usar `await` (sino tira un error) y automáticamente su valor de retorno es una promesa y no
el valor retornado directamente.
