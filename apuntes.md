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

![Ejemplo](<Captura de pantalla 2024-06-24 a las 19.39.06.png>)


## Aggregation Framework

![Pipeline Operators](<Captura de pantalla 2024-06-24 a las 19.40.58.png>)


## $sort, $limit, $skip
Ordenar documentos por uno o varios campos

- Misma sintaxis de ordenación que los cursores
- Espera el retorno del operador de canalización anterior 
- En memoria a menos que sea temprano e indexado

### Limitar y omitir el comportamiento del cursor de seguimiento

![Alt text](<Captura de pantalla 2024-06-24 a las 19.47.13.png>)

## Ordenar todos los documentos de la cadena


![Alt text](<Captura de pantalla 2024-06-24 a las 19.48.28.png>)

## Limitar la circulación de documentos

![Alt text](<Captura de pantalla 2024-06-24 a las 19.49.51.png>)