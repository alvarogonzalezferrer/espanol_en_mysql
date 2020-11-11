# Español en MySQL

Como ingresar correctamente los caracteres especiales del lenguaje español a MySQL.

Muchas veces encontramos bases de datos que tienen ingresos como **"Rubén"**, **"Julián"** almacenados como **"RubÃ©n"**, **"JuliÃ¡n"**, ó cosas similares donde los datos están, en apariencia, corruptos.

En este documento vamos a explorar la solución a dicho problema.

## Donaciones

Hacer este documento lleva mucho tiempo, esfuerzo e investigación.

Si quieres ayudarme a continuar con mi trabajo, podes ayudarme de diferentes maneras.

Registrarte con mis links referidos en:

* Uber Eats: Usa mi código al momento de pagar: **eats-eb0gd1** : http://ubr.to/EatsGiveGet

* Airbnb, obtenes un descuento si te registras con mi código (y yo también) :https://www.airbnb.co.cr/c/alvarog663

* Didi taxi: https://d.didiglobal.com/qMmgDGQszHDw6

* Bitso: https://bitso.com/?ref=vyzyc

**Donar** mediante Bitcoin a mi dirección, todo ayuda, pueden ser centavos, no importa!

**33Xh8YBQsMW8LGrnRR5wgEKXNzoaE5C95Q**

![Wallet 33Xh8YBQsMW8LGrnRR5wgEKXNzoaE5C95Q](btc/wallet.png)

**Muchas gracias por tu donación!**

## ATENCIÓN

LA INFORMACIÓN SE PROPORCIONA "COMO ESTÁ", **SIN GARANTÍA DE NINGÚN TIPO**, EXPRESA O IMPLÍCITA, INCLUYENDO PERO NO LIMITADO A GARANTÍAS DE COMERCIALIZACIÓN, IDONEIDAD PARA UN PROPÓSITO PARTICULAR E INCUMPLIMIENTO.

EN NINGÚN CASO LOS AUTORES Ó PROPIETARIOS DE LOS DERECHOS DE AUTOR SERÁN RESPONSABLES DE NINGÚN RECLAMO, DAÑOS U OTRAS RESPONSABILIDADES, YA SEA EN UNA ACCIÓN DE CONTRATO, AGRAVIO O CUALQUIER OTRO MOTIVO, DERIVADAS DE, FUERA DE, O EN CONEXIÓN CON LA INFORMACIÓN O SU USO U OTRO TIPO DE ACCIONES CON LA INFORMACIÓN PRESENTADA.

**SOLO SE PERMITE EL USO NO COMERCIAL DE ESTA INFORMACIÓN** (*Si te sirve en tu empresa, pagame, no seas ratón...*)

## Problemas y solución

Primero que nada, **NUNCA** modifiquen la base de datos de producción sin antes sacar copias de seguridad! No soy responsable de **NADA** que les pase con su base de datos, **esta información es meramente informativa y sin garantías de ningún tipo explicitas ó implícitas.**

En bases de datos heredadas, viejas, o mal configuradas, muchas veces nos encontramos con que los acentos, la eñe, y caracteres especiales, se han guardado de manera incorrecta.

Entonces, tenemos que datos como **"Rubén Héctor"** en la base de datos están almacenados como **"RubÃ©n HÃ©ctor"**, **"Julián"** como **"JuliÃ¡n"**, ó cosas similares donde los datos están, en apariencia, corruptos.

A veces esos datos se ven correctamente cuando los "traemos" con PHP y los mostramos en la web, pero siguen estando corruptos en la base de datos, y lo que es peor, si hacemos una búsqueda de "Julian", no nos va a traer a "Julián", por que la "*collation*", es decir el conjunto de reglas que se aplican para comparar caracteres en un charset es incorrecto.

El "*charset*" es un conjunto de símbolos y codificaciones, es decir, la forma en que la base de datos guarda internamente los datos, y es donde tenemos el problema.

Por defecto, MySQL usa *latin1* como *charset*, y  *latin1_swedish_ci* como *collation*. Las bases de datos viejas, heredadas ó mal configuradas estarán en ese juego de carácteres, que no sirve para manejar el español de manera correcta.

Vi muchas propuestas para resolver este problema, en foros y páginas de consulta a "programadores", un conjunto de muchas ""soluciones"" rebuscadas usando tablas de conversión, con tediosos procesos manuales, con perdida de datos, y dudosas conversiones ambiguas de los caracteres. Ninguna sirve realmente, por eso decidí escribir este articulo.

En realidad, las soluciones de los foros son algo incorrecto, no es necesario hacer semejante rodeo manual (que pasa si tenemos una base de datos con millones de registros? los vamos a revisar uno a uno con la tablita del foro?) para poder adecuar nuestros datos.

### Nota IMPORTANTE

Este documento apunta especialmente a bases de datos que trabajan con datos en español, si queremos usar otros caracteres, por ejemplo de idiomas como japonés, hebreo, árabe, etc debemos cambiar la configuración adecuadamente. Si debemos trabajar con muchos lenguajes combinados a la vez (por ejemplo un sistema global o universal), debemos apuntar a usar todo el sistema (PHP, MySQL, etc) con **utf8mb4**.

Mas info de utf8mb4 -> https://mathiasbynens.be/notes/mysql-utf8mb4

Latin1 charset (iso-8859) es 100% compatible para ser almacenado en un almacén de datos utf8. Todos los caracteres ascii y extended-ascii se almacenarán como de un solo byte.

Yendo del otro lado, de utf8 a Latin1 charset puede o no funcionar. Si hay caracteres de 2 bytes (caracteres más allá de 255 de ascii extendida), no se almacenarán en un almacén de datos de Latin1, <- *ESTE ES EL PROBLEMA QUE NORMALMENTE SUFREN LOS SISTEMAS EN ESPAÑOL*.

### Más información

Para no repetir toda la información sobre que es UTF8, que es Latin1, la codificación de carácteres internacionales, etc, les dejo algunos links interesantes.

* https://en.wikipedia.org/wiki/UTF-8
* https://en.wikipedia.org/wiki/Comparison_of_Unicode_encodings#In_detail
* https://en.wikipedia.org/wiki/ISO/IEC_8859-1 <-- **el infame latin1**
* https://en.wikipedia.org/wiki/Mojibake <-- este es el problema que sufrimos al usar latin1 cuando debimos usar utf8
* https://en.wikipedia.org/wiki/Windows-1252 <-- es bueno conocerlo
* https://balusc.omnifaces.org/2009/05/unicode-how-to-get-characters-right.html

## Pasos a realizar

1. Hacer **copia de seguridad** de los datos originales, un "dump" de toda la base de datos. Esto es lo MÁS IMPORTANTE, hacer backup y NO hacer experimentos en bases de datos de producción.

2. Asegurarse que **la base de datos, y todas las tablas están en charset utf8 y collation utf8_spanish_ci**, usando algo tipo

  ```SQL
  ALTER DATABASE mi_base_datos CHARACTER SET utf8 COLLATE utf8_spanish_ci;
  ```

  y para cada tabla

  ```SQL
  ALTER TABLE mi_tabla CHARACTER SET utf8 COLLATE utf8_spanish_ci;
  ```

3. Asegurarse que **la conexión a nuestra base de datos MySQL, desde PHP, esta siendo en UTF8**

Hay varias maneras de lograr esto, solo pongo la más compatible incluso con versiones viejas de PHP

  ```PHP
  // usando extensión mysqli (mysql improved) permite acceder a la funcionalidad proporcionada por MySQL 4.1 y posterior
  // conecto al server mysql
  $mysqli = new mysqli("localhost", "mi_usuario", "mi_clave", "test");

  // recordar verificar si realmente se conecto! acá no lo pongo para ahorrar espacio

  // con Mysqli, método preferido:
  $mysqli->set_charset("utf8");

  // otra manera no tan preferida
  // NO afectará a $mysqli->real_escape_string();
  $mysqli->query("SET NAMES utf8");
  $mysqli->query("SET CHARACTER SET utf8");

  // ----------------------------------------
  // PDO: extensión Objetos de Datos de PHP define una interfaz ligera para poder acceder a bases de datos en PHP.

  // solo a partir de PHP 5.3.6
  $pdo = new PDO("mysql:host=localhost;dbname=world;charset=utf8", 'mi_usuario', 'mi_contraseña');

  // viejo php
  $dbh = new PDO("mysql:$connstr",  $user, $password);
  $dbh->exec("set names utf8");
  ```

4. Asegurarse que nuestras paginas web tienen el **meta tag de UTF-8**

  ```HTML
  <head>
    <!-- aca el resto de tus cosas de tu pagina-->
    <meta charset="UTF-8">
  </head>
  ```

5. Los archivos php de tu proyecto estén **codificados en UTF-8**

Por ejemplo en Notepad++ elegimos *Codificación > UTF-8 sin BOM* y salvamos. Esto sera diferente según el editor de texto que usemos.

6. Y lo mas complejo, **CONVERTIR los datos corruptos de nuestra base de datos a UTF-8**

Aquí es donde los "trucos" que encontré en foros hacen agua, o se vuelven extremadamente complicados, y propensos a fallar y corromper todavía más los datos, así que les presento esta solución mas elegante.

Muchos *"programadores"* proponen casi manualmente revisar dato por dato con una tablita de conversión los caracteres incorrectos, una idea implausible si estamos trabajando con una base de datos corporativa con millones de registros...

Otra idea *incorrecta* es solo aplicar la configuración de la base de datos a UTF8, esto solo sirve si la base esta vacía, pero nuevamente ¿y si tengo millones de datos ya cargados incorrectamente?

La manera sencilla y automática es utilizar el mismo poder de MySQL, usando la función *CONVERT*, que permite conversiones entre *charsets*.

  ```SQL
  SELECT CONVERT(BINARY CONVERT('RubÃ©n HÃ©ctor' USING LATIN1) USING UTF8);
  ```

Lo convierte graciosamente a "Rubén Héctor".

Para una columna completa, por ejemplo, hacemos:

  ```SQL
  UPDATE tabla SET columna = CONVERT(BINARY CONVERT(columna USING LATIN1) USING UTF8);
  ```

Donde *tabla* y *columna* serian nuestra tabla y columna corrupta que queremos acomodar.

Opcional pero útil al terminar, ejecutar (**solo** tablas con motor  MyISAM, ARCHIVE, ó CSV)
InnoDB es el motor de almacenamiento predeterminado y más de uso general, y Oracle recomienda usarlo para tablas, excepto para casos de uso especializados: **esto no seria necesario en condiciones normales.**

  ```SQL
  # Para cada tabla

  # No usar en InnoDB, no tiene sentido
  REPAIR TABLE tabla;

  # Sirve para InnoDB
  OPTIMIZE TABLE tabla;
  ```
Se puede hacer para todas las tablas desde la linea de comandos (ingresando el password de root)

```
  $ mysqlcheck -u root -p --auto-repair --optimize --all-databases
```

Podemos hacer un pequeño programa que lo haga automáticamente para cada columna de texto de cada tabla de nuestra base de datos, de manera de acomodar toda la base de datos automáticamente.

Yo hice uno para un cliente hace poco (Nov-2020), que tenia una base de datos heredada toda en latin1 y no le permitía buscar palabras con acentos; ese problema lo solucioné al adaptar correctamente el código PHP heredado, sus paginas web, y también (y especialmente) la base de datos a usar UTF-8.

**El código estará proximamente en mi GitHub y es fácilmente adaptable para otros problemas similares.**

## Herramientas extra

Para finalizar este artículo, les dejo herramientas muy útiles en nuestras aventuras con bases de datos

Herramienta | Enlace
------------|---------------------
DBeaver | https://dbeaver.io/
MySQL Workbench | https://dev.mysql.com/downloads/workbench/
HeidiSQL | https://www.heidisql.com/download.php

Hoy aprendimos una valiosa lección, el poder del charset!

Saludos amiguitos, hasta la próxima aventura!

*Álvaro*
