# espanol_en_mysql
 Como ingresar correctamente el lenguaje español y sus caracteres especiales a MySQL 
Caracteres especiales en bases de datos MySQL
Problemas y solución

Primero que nada, NUNCA modifiquen la base de datos de producción sin antes sacar copias de seguridad! No soy responsable de NADA que les pase con su base de datos, esta información es meramente informativa pero sin garantías de ningún tipo explicitas ó implícitas.

En bases de datos heredadas, viejas, o mal configuradas, muchas veces nos encontramos con que los acentos y la eñe, y caracteres especiales, se han guardado de manera incorrecta.

Entonces, tenemos que datos como "Rubén Héctor" en la base de datos están almacenados como "RubÃ©n HÃ©ctor", "Julián" como "JuliÃ¡n", ó cosas similares donde los datos están, en apariencia, corruptos.

A veces esos datos se ven correctamente cuando los "traemos" con PHP y los mostramos en la web, pero siguen estando corruptos en la base de datos, y lo que es peor, si hacemos una búsqueda de "Julian", no nos va a traer a "Julián", por que la "collation", es decir el conjunto de reglas que se aplican para comparar caracteres en un charset es incorrecto.

El charset es un conjunto de símbolos y codificaciones, es decir, la forma en que la base de datos guarda internamente los datos, y es donde tenemos el problema.

Por defecto, MySQL usa latin1 como charset, y  latin1_swedish_ci como collation. Las bases de datos viejas, heredadas ó mal configuradas estarán en ese juego de carácteres, que no sirve para manejar el español de manera correcta.

Vi muchas propuestas para resolver este problema, en foros y páginas de consulta a "programadores", un conjunto de muchas ""soluciones"" rebuscadas usando tablas de conversión, con tediosos procesos manuales, con perdida de datos, y dudosas conversiones ambiguas de los caracteres. Ninguna sirve realmente, por eso decidí escribir este articulo. 

En realidad, esto es algo incorrecto, no es necesario hacer semejante rodeo para poder adecuar nuestros datos a UTF-8.

1. Hacer copia de seguridad de los datos originales, un "dump" de toda la base de datos. Esto es lo MÁS IMPORTANTE, hacer backup y NO hacer experimentos en bases de datos de producción.

2. Asegurarse que la base de datos, y todas las tablas están en charset utf8 y collation utf8_spanish_ci, usando algo tipo

ALTER DATABASE mi_base_datos CHARACTER SET utf8 COLLATE utf8_spanish_ci;

y para cada tabla

ALTER TABLE mi_tabla CHARACTER SET utf8 COLLATE utf8_spanish_ci;

3. Asegurarse que la conexión a nuestra base de datos MySQL, desde PHP, esta siendo en UTF8

Hay varias maneras de lograr esto, solo pongo la más compatible incluso con versiones viejas de PHP

// conecto al server mysql
$mysqli = new mysqli("localhost", "mi_usuario", "mi_clave", "test");

// recordar verificar si realmente se conecto! acá no lo pongo para ahorrar espacio

// con Mysqli, método preferido:
$mysqli->set_charset("utf8");

// otra manera no tan preferida
// NO afectará a $mysqli->real_escape_string();
$mysqli->query("SET NAMES utf8");
$mysqli->query("SET CHARACTER SET utf8");

// --- con PDO:

// solo a partir de PHP 5.3.6
$pdo = new PDO("mysql:host=localhost;dbname=world;charset=utf8", 'mi_usuario', 'mi_contraseña'); 

// viejo php
$dbh = new PDO("mysql:$connstr",  $user, $password);
$dbh->exec("set names utf8");
4. Asegurarse que nuestras paginas web tienen el meta de UTF-8

<meta charset="utf-8">

5. Los archivos php de tu proyecto (los *.php) estén codificados en UTF-8

Por ejemplo en Notepad++ elegimos Codificación > UTF-8 sin BOM y salvamos. Esto sera diferente según el editor de texto que usemos.

6. Y lo mas complejo, CONVERTIR los datos corruptos de nuestra base de datos a UTF-8

Aquí es donde los "trucos" que encontré en foros hacen agua, o se vuelven extremadamente complicados y propensos a fallar y corromper todavía más los datos, así que les presento esta solución mas elegante.

La manera sencilla es utilizar el mismo poder de MySQL, usando la función CONVERT, que permite conversiones entre charsets.

SELECT CONVERT(BINARY CONVERT('RubÃ©n HÃ©ctor' USING LATIN1) USING UTF8);

Lo convierte graciosamente a "Rubén Héctor".

Para una columna completa, por ejemplo, hacemos:

UPDATE tabla SET columna = CONVERT(BINARY CONVERT(columna USING LATIN1) USING UTF8);

Donde tabla y columna serian nuestra tabla y columna corrupta que queremos acomodar.

Podemos hacer un pequeño programa que lo haga automáticamente para cada columna de texto de cada tabla de nuestra base de datos, de manera de acomodar toda la base de datos automáticamente.

Yo hice uno para un cliente hace poco, que tenia una base de datos heredada toda en latin1 y no le permitía buscar palabras con acentos; ese problema fue solucionado por mi al adaptar correctamente el código PHP, sus paginas web, y también (y especialmente) la base de datos a usar UTF-8. 

El código esta en mi GitHub y es fácilmente adaptable para otros problemas similares. 

Hoy aprendimos una valiosa lección, el poder del charset! 

Saludos amiguitos, hasta la próxima aventura!

Álvaro