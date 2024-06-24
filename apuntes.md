
Las operaciones basadas en agregaciones nos permiten procesar la data que tenemos registrada para obtener resultados que aportan información útil. En otras palabras, nos permiten darle más valor a la data que tenemos al transformarla en información.
Usualmente, estas operaciones las utilizamos para agrupar registros o contarlos. Los group by y count que usualmente utilizamos en bases de datos relacionales, aunque también incluyen otras operaciones como sort o limit, para ordenar y mostrar un límite de registros respectivamente.


Para poder realizar estas operaciones de agregación MongoDB nos da 3 alternativas: la tubería de agregación (aggregation pipeline), la función MapReduce, y operaciones de agregación con propósito simple. En este post cubriremos la tubería de agregación y todos los operadores que utilizamos en conjunto con ella.

Tubería de Agregación:
Para utilizar la tubería de agregación de MongoDB, utilizamos el método aggregate de la colección sobre la cual deseemos realizar las operaciones. Dicho método recibe como parámetro un arreglo de “etapas”, en la que cada etapa consta de un objeto que representa una operación a realizar. Funciona como una tubería, ya que la data pasa por las etapas en el orden en que se pongan en el arreglo, es decir, la data resultante de la primera etapa, es la entrada de la segunda etapa, y así consecutivamente hasta que la última etapa retorna el resultado final obtenido.
La sintaxis para ejecutar un comando de agregación sería la siguiente:

db.collection.aggregate( [ { <etapa> }, { <etapa> } ] );

Y cada etapa está compuesta por un operador de etapa que representa una función y un objeto que representa una o varias expresiones que se pasa como parámetro a la función.
Los operadores que podemos utilizar a este nivel se denominan operadores de etapa, y los describiremos a continuación:
NOTA: Para los ejemplos continuaremos con los datos de ejemplo que utilizamos en los post anteriores.
MongoDB: Advanced Querys
Operaciones CRUD en MongoDB

Operadores de etapa:

$match:
{ $match: { } }
Este operador funciona de forma similar al método find() que ya vimos en las operaciones de crud. Básicamente nos permite filtrar documentos de la colección según la consulta que pasemos como parámetro..
Ej: Para obtener los estudiantes de nombre “Juan” lo hacemos de la siguiente forma:

db.estudiantes.aggregate([{$match : {nombre : 'Juan'}}])

$project:
{ $project: { <especificación(es)> } }
Nos permite modificar la data de un documento al mostrarla como resultado, según unas especificaciones que pasamos como parámetro  ya sea removiendo campos, o agregando nuevos mediante el uso de otras expresiones.
Ej: Si queremos solo ver el nombre de los estudiantes registrados, lo hacemos de la siguiente forma:

db.estudiantes.aggregate([{$project : {nombre:1, _id:0}}])

Ahora, también podemos agregar campos nuevos al documento utilizando expresiones. La más básica de ellas sería mostrar un campo ya existente pero como un alias.

Ej: para mostrar el apellido como “segundoNombre” lo haríamos de la siguiente forma:

db.estudiantes.aggregate([{$project : {nombre:1, _id:0, segundoNombre:"$nombre"}}])

Aquí utilizamos $ dentro de un string seguido del nombre del campo al que queremos hacer referencia.
Para ejemplos más útiles, MongoDB nos provee varios operadores de expresión que podemos utilizar para construir las etapas de las agregaciones.
Por ejemplo, si quisiéramos obtener el nombre y apellido del estudiante lo haríamos mediante una expresión que utilice el operador de strings $concat, de la siguiente forma:

db.estudiantes.aggregate([{$project : {_id:0, nombreCompleto : {$concat : ["$nombre"," ","$apellido"]}}}])

Más adelante veremos otros operadores de expresiones con los cuales podremos construir nuestras etapas.

$sort:
{ $sort: { : , : … } }
Nos permite ordenar los documentos obtenidos en la agregación. Como parámetro recibe un objeto con los diferentes campos sobre los cuales se ordenará, y un orden. El orden puede ser 1 si es ascendente o -1 si es descendente.
Ej: Si queremos obtener los estudiantes ordenados según su edad de forma ascendente, lo haríamos de la siguiente forma:

db.estudiantes.aggregate([{$project : {nombre:1,edad:1,_id:0}},{$sort : {edad : 1}}])

$limit:
{ $limit: }
Nos permite limitar el número de documentos obtenidos por la agregación en un entero que pasamos como parámetro.
Ej: Si queremos obtener solo 3 estudiantes más jóvenes, lo haríamos así:

db.estudiantes.aggregate([{$project : {nombre:1,edad:1,_id:0}},{$sort : {edad : 1}},{$limit : 3}])

$skip:
{ $skip: }
Nos permite saltarnos los n primeros documentos que se obtengan en la agregación, donde n es un entero que pasamos como parámetro.
Ej: Si queremos obtener los 3 estudiantes más jóvenes luego de los primeros 2, lo haríamos usando el siguiente comando:

db.estudiantes.aggregate([{$project : {nombre:1,edad:1,_id:0}},{$sort : {edad : 1}},{$skip:2},{$limit : 3}])

IMPORTANTE: Recuerden que las etapas de una agregación se ejecutan en el orden que las coloquemos en el arreglo. Por lo tanto, si coloco la etapa del $limit antes de la del $skip, el $skip recibirará como entrada los documentos ya limitados a cierto número y puede que no obtengamos el resultado deseado. Por esto usualmente la etapa con el $limit se coloca al final, luego de haberse realizado todos los filtros y ordenamientos necesarios.

$count:
{ $count: }
Nos permite obtener el número de documentos que se tienen en la etapa específica de la agregación en la que se llama, pasamos como parámetro un string que servirá como alias a la consulta.
Ej: Si queremos obtener la cantidad de estudiantes menores a 20 años, hacemos lo siguiente:

db.estudiantes.aggregate([{$match : {edad : { $lt : 20}}}, {$count : "Estudiantes menores a 20 años"}])

$group:
{ $group: { _id: , : { : }, … } }
El commando group agrupa documentos según una determinada expresión y genera un documento por cada grupo el cual será la salida de esta etapa, y la entrada para la próxima etapa en la tubería.
Cada documento generado está compuesto por un_id que tendrá como valor el resultado de la expresión el cual será la clave primaria del documento, por lo tanto, este _id es obligatorio.
Y opcionalmente, puede contener campos adicionales obtenidos a través de operadores de acumulación o acumuladores, los cual veremos más detalladamente más adelante.
Tanto el_id como los acumuladores aceptan como parámetro una expresión válida.
Ej: Si deseamos agrupar los estudiantes separando los mayores de edad de los menores de edad. Debemos pasar una expresión

db.estudiantes.aggregate([{$group : { _id : {$lt : ['$edad',18]}, edades : {$push : {edad : '$edad'}}}}])

Aquí podemos ver, que la expresión dada ({$lt : [‘$edad’,18]}) retorna true o false, por lo que estos son los ids de cada grupo generado, y cada documentos es clasificado en el grupo respectivo dependiendo si su valor cumple con el criterio. Para este ejemplo usamos el operador $push el cual explicaré más adelante.
De la misma forma, una expresión válida puede ser el nombre de uno de los campos directamente.
Ej: Si queremos agrupar los estudiantes por el programa que cursan, hacemos lo siguiente:

db.estudiantes.aggregate([{$group : { _id : '$programa', count : {$sum : 1}}}])

Aquí, el id de cada documento generado será el mismo campo “programa”, y como campo adicional usamos un contador, el cual utiliza el acumulador $sum.

$sample:
{ $sample: { size: } }
Este operador nos permite obtener una muestra aleatoria de n cantidad de documentos donde n es un entero positivo que pasamos como valor del parámetro size.
Ej: Para obtener una muestra aleatoria de 2 estudiantes hacemos lo siguiente:

db.estudiantes.aggregate([{$sample : {size : 2}}])

En el ejemplo podemos ver que a pesar de llamar el mismo método exactamente 2 veces, obtenemos documentos diferentes, al ser aleatorio el proceso de selección.

$out:
{ $out: «» }
Este operador nos permite obtener los documentos obtenidos en la tubería de agregación y agregarlos a una colección específica. Este operador debe utilizarse solamente como última etapa de la tubería.
Ej: Si quisiéramos generar una colección con los grupos generados previamente por la etapa $group en donde se dividieron a los estudiantes según el programa que cursaban, lo haríamos de la siguiente forma:

db.estudiantes.aggregate([{$group : { _id : '$programa', count : {$sum : 1}}},{$out : 'programas'}])

Nótese que este comando no nos muestra en resultado en la consola de comandos, por lo que para verificar que se realizó, hacemos una consulta sobre la colección generada, y en efecto, podremos ver que existe, y que sus documentos son los mismos generados en la etapa de agrupamiento.

$addFields
{ $addFields: { : <expresión>, … } }
Este operador nos permite añadir campos a una colección de documentos, especificando el nombre de los campos nuevos a agregar, cada uno con una expresión que generará el valor del campo nuevo para cada documento.
Ej: Si quisiéramos agregar 2 campos nuevos a la colección de estudiantes, uno que represente la sumatoria de sus notas, y otro que nos diga si es miembro de algún club, lo haríamos de la siguiente forma:

db.estudiantes.aggregate([{$addFields : { totalNotas : { $sum : '$notas' }, esMiembroDeClub : { $gt : ['$grupos',null]}}}])

En este ejemplo usamos un truquito con el operador $gt, ya que el operador $exists solo está disponible para querys y no para agregados. MongoDB permite comparaciones entre tipos de datos, y es por ello que cuando $grupos sea diferente de null (esto implica que debe ser diferente a cualquier tipo de dato, no solamente arrays), será considerado como mayor que null y retornará true.
IMPORTANTE: Este método no escribe en la bdd, es decir, los documentos en la bdd no serán modificados.

$sortByCount
{ $sortByCount:  <expresión> }
Este operador nos permite hacer un $group dada la expresión que pasamos como parámetro y un $count de todos los elementos, y posteriormente, ordenarlo según el número de elementos de forma descendente.
Ej: Con el siguiente comando podemos agrupar los estudiantes por programa, y ordenar los programas por número de estudiantes registrados en ellos:

db.estudiantes.aggregate([{$sortByCount : '$programa'}])

Operadores de expresiones:
Son los operadores que podemos utilizar en las expresiones que usamos para construir etapas.
Estos operadores se dividen en varios tipos, de los cuales cubriremos los más importantes de cada tipo.

Operadores comparativos:
Estos operadores nos permiten hacer comparaciones entre los atributos de los documentos agregados. La sintaxis varía un poco con respecto a los operadores comparativos que utilizamos en una query.
$eq, $ne:
{ $operador: [ <expresión1>, <expresión2> ] }
Estos operadores nos permiten comparar la igualdad o diferencia de dos expresiones que le pasamos como parámetro.
$eq retorna true cuando los valores son equivalentes y false cuando no lo son, $ne hace lo opuesto.
Ej: Para mostrar un campo que nos indique true o false si el estudiantes se apellida Smith, lo hacemos de la siguiente forma:

db.estudiantes.aggregate({$project : {_id:0,nombre:1,apellido:1,apellidaSmith:{$eq : ['$apellido','Smith']}}})

$gt, $gte, $lt, $lte:
{ $operador: [ <expresión1>, <expresión2> ] }
Estos operadores nos permiten comparar desigualdades, $gt y $gte para “mayor que” y “mayor o igual que” respectivamente, y $lt y $lte para “menor que” y “menor o igual que” respectivamente.
Retornar true cuando se cumple el criterio y false en caso contrario.
Ej: Si quisiéramos mostrar un campo que nos indicara true o false si el estudiante es mayor a 21 años lo haríamos de la siguiente forma:

db.estudiantes.aggregate({$project : {_id:0,nombre:1,apellido:1,mayorA21:{$gt : ['$edad',21]}}})

$cmp:
{ $cmp: [ <expresión1>, <expresión2> ] }
Finalmente, como último de los operadores de comparación está el $cmp, ya que es algo diferente a los demás en lo que respecta al valor retornado. En este caso, este operador se utiliza para determinar si un valor es menor, igual o mayor a otro valor.
Retorna -1 si el primer valor es menor al segundo, 0 si son iguales y 1 si el primer valor es mayor al segundo.
Ej: Podemos mostrar en un campo nuevo si un estudiante es menor, mayor o tiene exactamente 21 años.

db.estudiantes.aggregate({$project : {_id:0,nombre:1,apellido:1,edad:1,relacionA21:{$cmp : ['$edad',21]}}})

Como podemos ver en el resultado, los menores muestran un -1, los que tienen 21 años muestran 0 y los mayores muestran un 1.

Operadores booleanos:
Estos operadores permiten evaluar varias expresiones booleanas y retornar true o false como resultado de la evaluación.
$and:
{ $and: [ <expresión1>, <expresión2>, … ] }
Evalua varias expresiones y retorna true si y solo si todas las expresiones retornan true, de lo contrario retorna false.
Ej: Para mostrar un campo que indique si un estudiante se llama “María” y tiene menos de 30 años, hacemos lo siguiente:

db.estudiantes.aggregate({$project : {_id:0,nombre:1,apellido:1,edad:1,mariaMayorA30:{$and : [{$eq : ['$nombre','Maria']},{$lt : ['$edad', 30]}]}}})

$or:
{ $or: [ <expresión1>, <expresión2>, … ] }
Evalua varias expresiones y retorna true si al menos una de  las expresiones retornan true, de lo contrario retorna false.
Ej: Para mostrar un campo que indique si un estudiante se apellida “Linarez” ó tiene más de 22 años hacemos lo siguiente:

db.estudiantes.aggregate({$project : {_id:0,nombre:1,apellido:1,edad:1,linarezOMayorA22:{$or : [{$eq : ['$apellido','Linarez']},{$gt : ['$edad', 22]}]}}})

$not:
{ $not: [ <expressión> ] }
Evalúa una expresión booleana y retorna el valor contrario al obtenido. Es decir, si el resultado de la expresión es true, retorna false y viceversa.
Ej: Para mostrar un campo que indique si un estudiante no se llama “William Smith” podemos hacer lo siguiente:

db.estudiantes.aggregate({$project : {_id:0,nombre:1,apellido:1,edad:1,noWilliamSmith:{$not : { $and : [{$eq:['$nombre','William']},{$eq : ['$apellido','Smith']}]}}}})

Operadores aritméticos:
Estos operadores nos retorna el resultado de una operación aritmética dado los parámetros que le pasemos al operador.
$abs:
{ $abs: <número> }
Nos retorna el valor absoluto de un número dado.
$add, $multiply
{ $operador: [ <expresión1>, < expresión2>, … ] }
Nos permiten realizar las operaciones básicas de suma ($add) y multiplicación ($multiply). Reciben como parámetro n cantidad de expresiones que retornen un número o fecha.
$substract, $divide,$mod:
{ $operador: [ <expresión1>, < expresión2> ] }
Nos permiten realizar operaciones básicas de resta ($substract), división ($divide) y módulo ($mod). A diferencia de las anteriores, estas solo reciben 2 parámetros.
En el caso de la resta, al primer parámetro le será restado el segundo. Para la división, el primer elemento será dividido entre el segundo.
Ej: Para mostrar el uso de estos operadores en un ejemplo, mostraremos en un campo adicional el valor absoluto de la resta de la edad del estudiante menos  12 multiplicado por dos, y en otro le sumaremos 2 a la edad dividido entre 5.

db.estudiantes.aggregate([{$project : { _id: 0, edad: 1, valorAbsolutoRestaMultiplicacion: {$abs: { $subtract: ['$edad', {$multiply: [12,2]}]}}, sumaDivision: {$add: ['$edad',{$divide:['$edad',5]}]}}}])

$pow:
{ $pow: [ <número>, ] }
Retorna el resultado elevar un número dado a un exponente dado.
$sqrt:
{ $sqrt: <número> }
Retorna la raíz cuadrada de un número dado.
$exp:
{ $exp: }
Retorna el valor del número de Euler (e) elevado a un exponente dado.
$ln:
{ $ln: <número> }
Retorna el valor del logaritmo natural de un número dado.
$log:
{ $log: [ <número>, ] }
Retorna el valor del logaritmo en base dada de un número dado;
Ej: Para dar un ejemplo fácil de comprobar utilizando estos operadores utilizaremos una consulta en la que utilizaremos la edad y el número 3, cuando sea necesario, en cada una de ellas y las mostraremos todas como un campo nuevo proyectado, utilizando el siguiente comando:

db.estudiantes.aggregate([{$project : { _id: 0, edad: 1, edadElevadoALa3: {$pow : ['$edad',3]}, raizCuadradaDeEdad : {$sqrt: '$edad'}, eElevadoALaEdad : {$exp : '$edad'}, logaritmoNaturalDeEdad: {$ln : '$edad'}, logaritmoEnBase3DeEdad : {$log : ['$edad',3]}}}])

Podemos utilizar la calculadora para comprobar estos valores, pero lo importante es ver que podemos utilizar expresiones o números pasados directamente o combinaciones de ambas.
También podemos utilizar otras variables que guarden valores acumulados, pero los veremos más adelante.
$ceil, $floor, $trunc:
{ $operador: <número> }
$ceil nos retorna el número entero mayor más cercano a un número dado, $floor nos retorna el número entero menor más cercano y $trunc nos retorna la parte entera del número dado.
Nótese que la diferencia entre el operador $floor y $trunc se encuentra en el procesamiento de números negativos. Si tomamos como por ejemplo el número -8.23, el número entero menor más cercano es -9 y la parte entera del número es -8.
Ej: Como ejemplo veremos los tres operadores utilizados con el mismo parámetro el cual será el valor de 5 menos la edad dividido entre 4.

db.estudiantes.aggregate([{$project: {_id :0, edad:1, menosEdadEntre4 : {$subtract: [5,{$divide:['$edad',4]}]},ceil : {$ceil: {$subtract: [5,{$divide:['$edad',4]}]}},floor : {$floor: {$subtract: [5,{$divide:['$edad',4]}]}},trunc : {$trunc: {$subtract: [5,{$divide:['$edad',4]}]}}}}])

Nuevamente, luego veremos cómo utilizar variables internas para no tener que repetir el código entre las diferentes expresiones.

Operadores sobre strings:
Los siguientes operadores los podemos utilizar para realizar operaciones sobre cadenas de caracteres (strings) que usamos comúnmente, como concatenar, separar, entre otras. Las expresiones que podemos utilizar como parámetros deben resolverse a un string para que se puedan realizar correctamente.
$concat:
{ $concat: [ <expresión1>, <expresión2>, … ] }
Nos permite concatenar strings que pasamos como parámetros al operador. Estos strings pueden ser expresiones que se resuelvan a un string.
$split:
{ $split: [ , ] }
Este operador nos permite dividir un string que pasamos como parámetro en un arreglo de strings en el cual cada elemento es un string los cuales están separados por el delimitador que también pasamos como parámetro.
Ej: Para demostrar el uso de estos operadores, vamos a utilizar una consulta la cual nos muestre el nombre completo del estudiante (nombre y apellido) y luego dividiremos estos strings utilizando como delimitador la letra a.

db.estudiantes.aggregate([{$project : { _id:0, nombreCompleto : {$concat: ['$nombre',' ','$apellido']}, nombreCompletoSeparadoPorA : { $split : [ {$concat: ['$nombre',' ','$apellido']},'a']}}}])

Nota: El operador $split separa los caracteres antes y después del delimitador, por lo que el delimitador es ignorado en el resultado.

$indexOfBytes:
{ $indexOfBytes: [ , , , ] }
Este operador nos permite buscar la ocurrencia de un substring en un string, ambos pasados por parámetro. Opcionalmente podemos agregar un inicio y un fin que representan índices del string sobre los cuales estará delimitada la búsqueda.
Si se encuentra el substring varias veces en el string, el operador solo retornara el índice de la primera coincidencia.
Nota: Al igual que los arrays, el primer índice de cada string es 0.
Ej: Si deseamos buscar  las ocurrencias de la letra a en el nombre completo del estudiante, desde la 3ra hasta la décima letra, hacemos lo siguiente:

db.estudiantes.aggregate([{$project : { _id:0, nombreCompleto : {$concat :['$nombre',' ','$apellido']}, indiceDeAenNombreCompleto : { $indexOfBytes : [{$concat: ['$nombre',' ','$apellido']},'a',2,9]}}}])

$strcasecmp:
{ $strcasecmp: [ <expresión1>, <expresión2> ] }
Permite comparar dos strings que pasamos como parámetro sin considerar las minúsculas o mayúsculas.
El operador retorna 1 si el primer string es “mayor” al segundo, 0 si son iguales o -1 si es “menor”.
El criterio de comparación, así como la forma en la que se termina si es mayor o menor están relacionadas solamente a caracteres ASCII. Y funciona de la siguiente forma:
Se compararán los strings caracter por caracter, y en el primer caracter que sea diferente, si el primer caracter se encuentra antes que el segundo en la tabla ASCII se considerará el primer string menor al segundo, y si se encuentra después, se considerará mayor.
Para realizar una búsqueda que si tome en cuenta las mayúsculas o minúsculas utilizamos el operador $cmp que ya vimos anteriormente.
Ej: Como ejemplo vamos a comparar el string “maRiA” con los nombres  que tenemos registrados en la colección de estudiantes.

db.estudiantes.aggregate([{$project: { _id:0, nombre:1, esMaria: {$strcasecmp : ['$nombre','maRiA']}}}])

$toLower, $toUpper:
{ $toLower: <expresión> } / { $toUpper: <expresión> }
Nos permiten convertir todos los caracteres de un string a minúsculas ($toLower) o mayúsculas ($toUpper).
Ej: Para mostrar los nombres en minúsculas y los apellidos en mayúsculas de los estudiantes hacemos lo siguiente:

db.estudiantes.aggregate([{$project: {_id:0, nombreEnMinusculas:{ $toLower : '$nombre'}, apellidoEnMayusculas: {$toUpper: '$apellido'}}}])

$strLenCP:
{ $strLenCP: }
Este operador nos permite obtener la cantidad de caracteres que hay en un string.

Operadores de grupos (set):
Estos operadores realizan operaciones de grupos sobre arrays. La principal diferencia entre arrays y set es que los sets ignoran los posibles elementos repetidos en los arrays, tratando cada elemento del array como único.
Podemos pasar expresiones como parámetros a estos operadores siempre y cuando retornen un arreglo.
$setEquals:
{ $setEquals: [ <expresión1>, <expresión2>, … ] }
Este operador nos permite comparar n cantidad de arreglos que pasamos como parámetro para determinar si poseen los mismos elementos. Como son tratados como set, solo se compara si sus elementos únicos son los mismos, es decir, no se toman en cuenta valores repetidos ni tampoco el orden en el que se encuentren los elementos.
El método retorna true si y solo si todos los elementos de un arreglo están en el otro y además, no sobra ningún elemento, es decir, solo si tienen exactamente la misma cantidad de elementos y los mismos elementos. De lo contrario retorna false.
Ej: Si queremos saber los estudiantes que tengan notas de 16, 18 y 19 exactamente, utilizamos el siguiente comando:

db.estudiantes.aggregate([{$project : {_id:0, notas:1, tiene1618y19: {$setEquals : ['$notas',[16,18,19]]}}}])

$setIntersection:
{ $setIntersection: [ < expresión1>, < expresión2>, … ] }
Este operador toma n cantidad de arreglos que pasamos como parámetros, y retorna un arreglo con solamente los elementos que existen en todos ellos, es decir, retorna la intersección de los arreglos.
Si no existen elementos que estén en todos los arreglos, retorna un arreglo vacío.
Ej: Para realizar una intersección entre las notas de los estudiantes y el arreglo [18,19,20] hacemos los siguiente:

db.estudiantes.aggregate([{$project : {_id:0, notas:1, intercepcion181920: {$setIntersection : ['$notas',[18,19,20]]}}}])

$setUnion:
{ $setUnion: [ <expresión1>, <expresión2>, … ] }
Toma n cantidad de arreglos como parámetros y retorna un arreglo con todos los elementos diferentes que existen en cada uno de los arreglos, es decir, realiza una unión de los arreglos.
Ej: para unir el arreglo con las notas de los estudiantes con un arreglo con las notas 16 y 14 de modo que los que ya tengan esas notas no se repitan, lo haríamos de la siguiente forma:

db.estudiantes.aggregate([{$project : {_id:0, notas:1, union1416: {$setUnion : ['$notas',[14,16]]}}}])

$setDifference:
{ $setDifference: [ <expresión1>, <expresión2> ] }
Toma 2 arreglos como parámetro y retorna todos los elementos que existan en el primer arreglo que no estén en el segundo, es decir, le resta los elementos de la intersección de los arreglos al primer arreglo.
Ej: para obtener las notas de los estudiantes exceptuando las notas 12 y 19 hacemos lo siguiente:

db.estudiantes.aggregate([{$project : {_id:0, notas:1, diferencia12y19: {$setDifference : ['$notas',[12,19]]}}}])

$setInSubset:
{ $setIsSubset: [ <expresión1>, <expresión2> ] }
Toma 2 arreglos como parámetros y retorna true si el primer arreglo es un subconjunto del segundo, incluyendo cuando son iguales, es decir, si todos los elementos del primer arreglo están en el segundo. Retorna false de lo contrario.
Importante: tener en cuenta el orden en el que pasamos los elementos.
Ej: para determinar los estudiantes que tienen al menos un 15 y un 16 hacemos lo siguiente:

db.estudiantes.aggregate([{$project : {_id:0, notas:1, tiene15y16: {$setIsSubset : [[15,16], '$notas']}}}])

$anyElementTrue:
{ $anyElementTrue: [ <expresión> ] }
Este operador evalúa cada elemento del arreglo y si al menos un elemento del arreglo es true, retorna true, de lo contrario retorna false. Los elementos que no sean 0, null o undefined retornan false, todos los demás retornan true, incluyendo arreglos internos y valores numéricos diferentes de 0.
$allElementsTrue:
{ $allElementsTrue: [ <expresión> ] }
Este operador evalúa cada elemento del arreglo y si al menos un elemento del arreglo es false, retorna true, de lo contrario retorna false. Aplica el mismo criterio que $anyElementTrue.

Operadores sobre arreglos (arrays):
Estos operadores nos permiten realizar operaciones sobre arreglos. Siempre que se requiere un arreglo como parámetro, podemos en lugar, pasar una expresión, siempre y cuando esta retorne un arreglo.
$arrayElementAt
{ $arrayElemAt: [ , <índice> ] }
Nos permite retornar el valor de un elemento de un arreglo en un índice determinado.
Ej: Para retornar la segunda nota de cada estudiante lo hacemos de la siguiente forma:

db.estudiantes.aggregate([{$project: { _id:0, notas:1, segundaNota:{$arrayElemAt : ['$notas',1]}}}])

$concatArrays
{ $concatArrays: [ , , … ] }
Con este operador podemos concatenar n cantidad de arreglos que pasamos como parámetros, en el orden en que los pasamos.
Ej: Supongamos que queremos mostrar un arreglo con los valores “Este ”, “es ”, “ un”, luego los elementos del arreglo de notas del estudiante y luego los valores “arreglo”, “concatenado”, “con las notas de:” y el nombre completo del estudiante.

db.estudiantes.aggregate([{$project:{_id:0, arregloConcatenado: {$concatArrays : [['Este','es','un'],'$notas',['arreglo','concatenado con las notas de:',{$concat:['$nombre',' ','$apellido']}]]}}}])

$filter
{ $filter: { input: , as: , cond: <expresión> } }
Nos permite filtrar los elementos de un arreglo que pasamos como entrada (input), y retornarlos según cumplan una condición especifica en otro arreglo. El parametro ‘as’ lo utilizamos para hacer referencia al cada elemento del arreglo en la condición posterior utilizando $$.
Ej: para retornar solo las notas de los estudiantes que sean mayor a 16 lo hacemos de la siguiente forma:

db.estudiantes.aggregate([{$project: {_id:0, nombreCompleto: {$concat: ['$nombre',' ','$apellido']}, notasMayoresA16 : {$filter: {input : '$notas', as: 'nota', cond: { $gt : ['$$nota', 16]}}}}}])

Nótese que al comparar las notas numéricas con las alfabéticas se hace una comparación entre strings usando los caracteres ASCII y su número respectivo para determinar el mayor, y como las letras están después de los números, estas son mayores.

$in
{ $in: [ <expresión>, ] }
Retorna true o false dependiendo de si el valor que resulta de una expresión se encuentra dentro de un arreglo que también pasamos como parámetro.
Ej: Si queremos saber si algún estudiante tiene un 18 en alguna de sus notas hacemos lo siguiente:

db.estudiantes.aggregate([{$project: { _id: 0, nombre:1, notas:1, tieneAlMenosUnaCon18oMas: {$in : [18,'$notas']}}}])

$indexOfArray
{ $indexOfArray: [ , <expresión>, , ] }
Este operador realiza una búsqueda de un elemento que pasamos en una expresión en un arreglo dado, y retorna la posición en el arreglo de la primera coincidencia que exista, retorna -1 si ningún elemento coincide. Opcionalmente podemos pasar 2 valores enteros inicio y fin que representan los índices desde donde y hasta cuando se llevará a cabo la búsqueda.
Ej: Para retornar los índices de la ocurrencia de un 16 en las notas de los estudiantes lo hacemos de la siguiente forma:

db.estudiantes.aggregate([{$project: { _id: 0, nombre:1, notas:1, indiceDelPrimer16:{$indexOfArray: ['$notas',16]} }}])

$isArray
{ $isArray: [ <expresión> ] }
Determina si la expresión que pasamos como parámetro es un arreglo. Retorna true si es un arreglo o false de lo contrario
Ej: Con el siguiente comando verificamos si los valores de notas, programa y grupos son arreglos o no.

db.estudiantes.aggregate([{$project: { _id: 0, nombre:1, notas:1, notasEsArreglo: {$isArray: '$notas'}, programaEsArreglo: {$isArray : '$programa'}, gruposEsArreglo : {$isArray: '$grupos'}}}])

Como podemos ver, en el caso en que la expresión retorne null como el del estudiante que no tiene ningún grupo, también retorna false.

$map
{ $map: { input: , as: , in: <expresión> } }
Este operador nos permite aplicar operaciones mediante una expresión a cada elemento de un arreglo y retorna un arreglo con los valores modificados. Utilizamos el string que pasamos como parámetro “as” como identificador de las variables a utilizar en la expresión que pasamos al parámetro “in”.
Ej: Si queremos obtener el tipo de dato de cada nota asociada al estudiante. Lo hacemos de la siguiente forma:

db.estudiantes.aggregate([{$project: { _id:0, notas:1, mitadDeNotas: { $map: {input:'$notas', as:'nota', in: {$type: '$$nota'}} } }}])

$range
{ $range: [ , , ] }
Nos permite obtener los números existentes entre dos enteros inicio y fin que pasamos como parámetros. Opcionalmente podemos pasar otro entero “paso” que representa el salto que se dará entre cada elemento del arreglo resultante.
Ej: Para obtener los números entre la edad del estudiante y 30 con un salto de 2 hacemos lo siguiente:

db.estudiantes.aggregate([{$project: { _id:0, notas:1, rangoEntreEdadY30: { $range : ['$edad',30,2] } }}])

Como podemos ver, podemos utilizar expresiones normales siempre y cuando se resuelvan a números enteros.

$reduce
{$reduce: { input: , initialValue: <expresión>, in: <expresión> }}
Con este operador podemos iterar sobre lo elementos de un arreglo y retornar un único valor el cual obtenemos a través de la expresión que pasamos como parámetro “in”. En dicha expresión, podemos utilizar las variables $$value y $$this para hacer referencia al valor a acumulado y al elemento sobre el que se está iterando respectivamente.
Ej: Para obtener el valor acumulado de la suma de la mitad y la décima parte de la edad de un estudiante lo hacemos de la siguiente forma:

db.estudiantes.aggregate([{$project: { _id:0, notas:1, acumuladorDeNotas: { $reduce:{ input : [{$divide:['$edad',2]},{$divide : ['$edad',10]}], initialValue: 0, in : { $add : ['$$value','$$this']}}} }}])

Nótese que de la misma forma podemos pasar cualquier expresión al parámetro input siempre y cuando se resuelva a un arreglo.

$reverseArray
{ $reverseArray: }
Retorna el mismo arreglo que pasamos como parámetro pero con los elementos en el orden inverso al original.
Ej: Para obtener las notas en orden inverso al original hacemos lo siguiente:

db.estudiantes.aggregate([{$project: { _id:0, notas:1, notasInversas: {$reverseArray: '$notas'} }}])

$size
{ $size: }
Nos permite obtener la cantidad total de elementos de un arreglo.
Ej: Para saber cuántas notas tiene un estudiante:

db.estudiantes.aggregate([{$project: { _id:0, notas:1, cantidadDeNotas: {$size: '$notas'} }}])

$slice
{ $slice: [ ,,<número n> ] }
Con este operador retornamos un arreglo conformado por una cantidad “n” de elementos a la derecha de una “posición” de un arreglo específico.
En este caso el parámetro posición es opcional, por lo que si solo pasamos dos parámetros al método, serán el arreglo y el número de elementos.
Ej: Para obtener las primeras 2 notas del estudiante, hacemos lo siguiente.

db.estudiantes.aggregate([{$project: { _id:0, notas:1, primeras2Notas: {$slice: ['$notas',2]} }}])

De no pasar ninguna posición, se toma 0 como valor por defecto.

$zip
{$zip: {inputs: [ ,  … ], useLongestLength: , defaults:   }}
Este operador recibe como input n cantidad de arreglos y retorna un arreglo en donde cada elemento de dicho resultado serán los elementos en la posición n de cada uno de los arreglos, es decir, el primer elemento del arreglo resultante será un arreglo con todos los primeros elementos de los arreglos que recibe como parámetro input, el segundo elemento será un arreglo con todos los segundo elementos y así sucesivamente.
El segundo parametro es un valor booleano que con el que indicamos si la longitud del arreglo resultante estará condicionada por la longitud del arreglo más largo de los arreglos de entrada de ser true. De ser false, ignora la cantidad de elementos que excedan el valor del arreglo más corto.
Ej: Usaremos este operador para mezclar los elementos de las notas de los estudiantes y otros 2 arreglos ([1,2,3] y [‘A’,’B’,’C’]).

db.estudiantes.aggregate([{$project: { _id:0, notas:1, zipNotas: {$zip:{inputs:['$notas',[1,2,3],['A','B','C']]}} }}])])

Como podemos ver, los elementos que exceden el tamaño del arreglo más pequeño no se muestran.

Operadores de variable:
$let
{ $let: { vars: { : <expresión>, … }, in: <expresión> }}
Este operador nos permite definer n cantidad de variables obtenidas mediante expresiones para utilizarlas luego en una expresión y retorna el resultado de esta última expresión.
Las variables que definimos en el parámetro “vars” no tienen uso alguno afuera de la expresión que pasamos como parámetro “in”. Dichas variables serán identificadas por el string que pasemos como nombre, y luego podremos hacer referencia a ellas en la expresión utilizando dicho string con $$.
Ej: Vamos a definir una variable que almacenará la longitud del nombre completo del estudiante, y luego lo retornaremos de forma que diga “El nombre del estudiante x tiene n letras.”

db.estudiantes.aggregate([{$project : {_id:0, nombre:1, apellido:1, cantidadDeLetrasEnNombreCompleto: { $let : { vars : { nombreCompleto : {$concat: ['$nombre', '$apellido'] }}, in: { $concat : ["El nombre del estudiante ",'$$nombreCompleto'," tiene ", { $substr : [{$strLenCP:'$$nombreCompleto'},0,-1 ] }, " letras" ] } } } }}])

Aquí utilizamos un truquito para poder convertir un entero a un string fácilmente, que consiste un usar el método substring con parámetros 0 y -1.

Operador literal:
$literal
{ $literal: }
Este operador nos permite retornar el valor que pasamos como parámetro sin ser procesado por MongoDB. Bastante útil cuando tenemos que utilizar caracteres especiales como $, o cuando queremos pasar una operación sin que se resuelva.
Ej: Para ver el funcionamiento de este operador utilizaremos el siguiente comando:

db.estudiantes.aggregate([{$project : {_id:0, nombre: 1, literalEdad: {$literal : '$edad'}, literalFuncion:{$literal: { $add : ['$edad',2]}}}}])

Como podemos observar, en el caso de la edad, en la que con el uso del símbolo $ mongo usualmente retornaría el valor de la edad del estudiante, se retorna el valor literal que pasamos, es decir “$edad”, y en el caso de la función, también podemos ver que la suma que normalmente se llevaría a cabo con el comando add no se ejecuta sino que retorna el valor literal que pasamos, en este caso, la función add con los parámetros que recibe, sin ser procesados.

Operador de tipo de dato:
$type
{ $type: <expresión> }
Este operador nos permite obtener el tipo de dato BSON en el que se resuelve la expresión que pasamos como parámetro. Los tipos de datos posibles los podemos consultar en esta tabla:  Tipos de datos BSON
Ej: Con el siguiente comando observamos el tipo de dato de algunos de los campos que pertenecen al estudiante:

db.estudiantes.aggregate([{ $project: { _id:0, nombre: 1, tipoApellido : {$type : '$apellido'}, tipoEdad : { $type:'$edad'}, tipoNotas: {$type: '$notas'}, tipoPrograma: {$type:'$programa'}, tipoGrupo: {$type: '$grupos'} } }])

Importante notar que cuando un campo no exista en un documento retornara “missing”, diferente a que si exista el campo en el documento pero sea null, en el caso que si retornará null.

Operadores condicionales:
$cond
{ $cond: [ <expresión-booleana>, <expresión-true>, <expresión-false> ] }
Podemos utilizar este operador para realizar una condición if, es decir, si la expresión booleana que pasamos como parámetro retorna true, entonces retorna el valor que se resuelva de la expresión-true, de lo contrario, se resuelve de la expresión-false.
Ej: Si queremos ver que estudiantes son mayores de edad, de forma de que si el estudiante es mayor de edad muestre “Si” y de lo contrario “No”, hacemos lo siguiente:

db.estudiantes.aggregate([{$project: {_id:0, nombre:1, edad:1,esMayorDeEdad: {$cond:[{$gte: ['$edad',18]},'Si','No']}}}])

Las expresiones de true y false, pueden ser cualquier expresión que podamos usar en MongoDB, en este caso solo se pasó un string por conveniencia del ejemplo.

$ifNull
{ $ifNull: [ <expresión>, <expresión-reemplazo> ] }
Este operador nos permite sustituir el valor que retornaría una expresión en caso de que el resultador fuera null, indefinido o un campo inexistente por un valor que retornaría una función de reemplazo que pasamos como parámetro. Muy útil para manejar casos en los que algún campo pueda no existir en todos los documentos.
Ej: Para que en el caso de que cuando un estudiante no tenga ningún grupo nos muestre un string que diga “No pertenece a ningún grupo” hacemos lo siguiente:

db.estudiantes.aggregate([{$project: { _id:0, nombre:1, grupos: { $ifNull:['$grupos',"No pertenece a ningún grupo"]}}}])

Nuevamente, podemos utilizar cualquier tipo de expresión válida para la expresión de reemplazo.

$switch
$switch: { branches: [ { case: <expresión>, then: <expresión> },
{ case: <expresión>, then: < expresión> , … ], default: < expresión> }
Este operador nos permite hacer una condición switch, es decir, determinar un grupo de condiciones complementarias de modo que cuando se cumpla la expresión del case, se ejecuta la expresión del then respectivo, y si ninguna se cumple, se ejecuta la expresión del default.
Ej: Vamos a mostrar por cada estudiante si su edad esta entre los grupos de 10 a 20, de 20 a 30 u otro grupo diferente a estos.

db.estudiantes.aggregate([{$project : { _id:0, nombre:1, edad:1, grupoDeEdad: { $switch: { branches: [{case:{$lt:['$edad',20]} ,then:"Grupo de menores de 20"},{ case : {$lt:['$edad',30]} ,then: "Grupo menores de 30"}], default: "Pertenece a otro grupo" } } }}])], default: "Pertenece a otro grupo" } } }}])])])

Operadores sobre fechas:
$dayOfYear, $dayOfMonth, $dayOfWeek
$operador :
Con estos operadores obtenemos el día del año, el día del mes y el día de la semana correspondientes a la fecha que pasamos como parámetro. Podemos pasar una expresión como parámetro siempre y cuando se resuelva en a una fecha.

$year, $month, $week, $hour, $minute, $second, $millisecond
$operador :
Recibe el mismo parámetro que los operadores anteriores, en este caso, se encargan de retornar el año, mes, semana, hora, minuto, segundo y milisegundo de dicha fecha.
Ej: Para demostrar el funcionamiento de estos operadores, los usaremos todos en un solo comando utilizando la fecha actual (new Date()) para ver el resultado de cada uno.

db.estudiantes.aggregate([{$project : { _id:0, diaDelAnio: {$dayOfYear: new Date()}, diaDelMes:{ $dayOfMonth : new Date() }, diaDeLaSemana: {$dayOfWeek : new Date()}, anio:{$year : new Date()} , mes: {$month: new Date()}, semana : {$week : new Date()}, hora:{$hour : new Date()}, minuto:{$minute: new Date()}, segundo : {$second: new Date()}, milisegundo:{$millisecond: new Date()} }}])

El comando anterior podría ser más óptimo, pero lo dejaremos así con la finalidad de explicar el funcionamiento de los operadores.

$dateToString
{ $dateToString: { format:
, date: } }
Nos permite convertir una fecha que pasamos como parámetro a un string usando un formato específico. Para ver los posibles formatos ver aquí
Formatos para utilizar con el operador $dateToString
Ej: Para mostrar la fecha actual en formato dia/mes/anio, lo hacemos de la siguiente forma:

db.estudiantes.aggregate([{$project : { _id:0, diaMesaAnio: {$dateToString:{format: "%d/%m/%Y" ,date: new Date()}}}}])

Acumuladores:
 Los acumuladores son operadores muy útiles cuando estamos creando etapas de agrupamiento ($group) o proyección ($project). Nos permiten obtener valores numéricos de operaciones comúnmente utilizadas sobre grupos de documentos, como sumatorias, máximos o mínimos, entre otras.
Los operadores de acumulación que podemos utilizar en MongoDB son los siguientes:
$sum, $avg:
{ $sum/$avg: <expresión> } (En $group o $project))
O
{ $sum/$avg: [ <expresión1>, <expresión2> … ]  } (Solo en $project)
Este acumulador permite acumular los valores que retorne la expresión que pasamos como parámetro. En caso de que se use en el su segunda forma, retorna la sumatoria de todas las expresiones que se pasen como parámetro por cada uno de los documentos.
Ej: Para obtener la sumatoria y el promedio de todas las edades, lo hacemos de la siguiente forma.

db.estudiantes.aggregate([{$group : {_id: 'idAux', suma: {$sum: '$edad'}, promedio: {$avg:'$edad'}}}]);

NOTA: En este ejemplo como en los siguientes, como estamos utilizando un id auxiliar, toda la colección se considera un documento agrupado por este id, por lo que el operador está retornando la sumatoria o average te todo el documento conformado por la colección.

$max, $min
{ $max/$min: <expresión> } (En $group o $project))
O
{ $max/$min: [ <expresión1>, <expresión2> … ]  } (Solo en $project)
Permite obtener el valor máximo y el valor mínimo de la expresión que se evalúa para cada documento generado por el agrupamiento.
Ej: Obtengamos la edad máxima y la mínimo de la colección

db.estudiantes.aggregate([{$group : {_id: 'idAux', maximo: {$max: '$edad'}, minimo: {$min:'$edad'}}}]);

$first, $last
{ $first/$last: <expresión> } (Solo en $group))
Estos operadores solo los podemos utilizar en la etapa de $group y nos permiten obtener el primer y el último valor respectivamente. Lo ideal es utilizar estos operadores luego de una etapa $sort, para obtener valores que tengan sentido que estén en el primer o último lugar, de no hacerlo, sencillamente retorna el primer y el ultimo valor registrado en la colección.
Ej: De la siguiente forma podemos obtener el estudiante más joven y al más viejo.

db.estudiantes.aggregate([{$sort: {edad:1}},{$group : {_id: 'idAux', primero: {$first: '$edad'}, ultimo: {$last:'$edad'}}}]);












# Consultas básicas: find db.collection.find(query, projection)

### query: 
especifica condiciones o filtros 
### projection: 
especifica los campos a proyectar

Devuelve los campos indicados de los documentos que
satisfacen las condiciones

### Las secciones query y projection son opcionales

Query vacía devuelve todos los documentos (filas) 

Projection vacía devuelve todos los campos (columnas)


### Consultas sin condiciones

SQL                                     MongoDB

SELECT *
FROM inventory                       db.inventory.find()
---------------------------------------------------------------------------
SELECT id,
       item
FROM inventory                      db.inventory.find({},{item:1})
----------------------------------------------------------------------------
SELECT item
FROM inventory                       db.inventory.find({}, {item:1,_id:0})
-----------------------------------------------------------------------------


### Consultas con condiciones de filtrado

SQL                            MongoDB

SELECT item
FROM inventory
WHERE status=”A”              db.inventory.find( {status:”A”},{item:1,_id:0})

---------------------------------------------------------------------------
SELECT item
FROM inventory
WHERE status <>”A”             db.inventory.find( {status:{$ne:”A”}}, {item:1,_id:0})

------------------------------------------------------------------------------------------
SELECT *
FROM inventory
WHERE status=“A” or
      item = “paper”            db.inventory.find({$or:[{status:"A"}, {item:"paper"}]})


#### Lista completa de operadores (de comparación, lógicos, etc)
<https://docs.mongodb.com/manual/reference/operator/query>



### Consultas básicas: distinct 

db.collection.distinct(field, query, options)

#### field: 
campo sobre el cual aplica el distinct 
#### query:
especifica condiciones o filtros 
#### options:
ver documentación
.


SQL                                MongoDB

SELECT
DISTINCT("instock.qty")
FROM inventory
WHERE status=“A”                   db.inventory.distict("instock.qty",{ status: "A" });


### Aggregation framework
La idea principal es la de pipeline de ejecución.
``
ls -l | grep -i mongo
``
#### Secuencia de etapas (stages) de: 
filtrado, transformación, agrupación, ordenamiento,
y proyección.

![Ejemplo](</img/Captura de pantalla 2024-06-24 a las 19.39.06.png>)


## Aggregation Framework

![Pipeline Operators](</img/Captura de pantalla 2024-06-24 a las 19.40.58.png>)


## $sort, $limit, $skip
Ordenar documentos por uno o varios campos

- Misma sintaxis de ordenación que los cursores
- Espera el retorno del operador de canalización anterior 
- En memoria a menos que sea temprano e indexado

### Limitar y omitir el comportamiento del cursor de seguimiento

![Alt text](</img/Captura de pantalla 2024-06-24 a las 19.47.13.png>)

## Ordenar todos los documentos de la cadena


![Alt text](</img/Captura de pantalla 2024-06-24 a las 19.48.28.png>)

## Limitar la circulación de documentos

![Alt text](</img/Captura de pantalla 2024-06-24 a las 19.49.51.png>)
