# Preparando-bdfi

# Examen 2019 enero

**Pregunta 1 (1pt) - ¿Para qué sirven los ODMs?. ¿Qué ventajas tiene utilizarlos?**

Un ODM es un object-document mapper, un middleware que relaciona documentos en una bbdd con los objetos de una aplicación. Para que la aplicación almacene sus objetos en la bbdd se puede optar por realizar queries directamente sobre un driver o emplear un sw específico, en este caso un odm. Se destaca como ventaja que este sw hace de capa intermedia y suele impedir inyecciones de sql o facilita tratar formatos difíciles como el date.

**Pregunta 2 (1pt) - Teorema de CAP (o de Brewer). Enuncielo y diga qué significan la C, la A y la P**

Dado un sistema distribuido (uno que posee un estado que se encuentra distribuido entre un conjunto de nodos) se definen tres características de él: su C o Consistencia, su A o disponibilidad y su P o tolerancia a fallos que provoquen un particionamiento.

Consistencia: la posibildiad de que un estado esté actualizado en el sistema.

Disponibilidad: la posibilidad de que se pueda operar el sistema.

Tolerancia a particionamiento: la posibilidad de que ocurra un error en la red.

EL teorema de CAP dice que "en un sistema distribuido, con propiedades C, A y P, sólo se pueden asegurar dos". Es decir, si se produce un error en la red y el estado entre los nodos no se puede actualizar, o se decide optar a permitir operar el sistema (A) o a que no se opere, en favor de garantizar un estado consistente (C).

**Pregunta 3 (1pt) - ¿En qué consiste el particionamiento (o Sharding) de una Base de Datos? Enumere
2 ventajas**

Particionar una bbbdd consiste en fragmentar los datos almacenándolos en varios nodos. Se destacan dos ventajas que puede propiciar esta práctica:

- Mejorar el rendimiento siendo eficiente económicamente: pues al escalar el servicio (por ejemplo cuando el índice de los datos ya no cabe en la ram de la máquina), en vez de adquirir una máquina mejor y más cara (escalado vertical) se puede optar por realizar un shard en un clúster sobre máquinas COTS.

- Mejorar la latencia: al divdir los datos en nodos, se puede plantear acercar cierta ifnoramción al edge de ciertos usuarios para disminuir su latencia.

**Pregunta 4 (1pt) – Data Value Pyramid o pirámide de valor de los datos. Indique qué es y que tiene
cada uno de los niveles. (Si quiere puede ayudarse dibujando la pirámide en cuestión).**

La pirámida de valro del dato es una representación inspirada asemejanza de la pirámide de las necesidades de Mashlow, que describe ciertas acciones sobre el análisis de datos y como en los niveles ascendientes las acciones aportan más valor.

Actions

Predictions

Reports

Explorations

Records

En la base se encuentran la creación de pipelines y estructuras para almacenar y dar formato a los datos.

A continaución los análissi y las exploraciones.

Después, los reportes o análsis más generales.

Luego las prediciones y por último las acciones.


**Sea una base de datos MongoDB llamada “stats” que contiene un documento con información por
cada barrio de España. El siguiente JSON es un ejemplo de dichos documentos:**

```
{
 "_id": ObjectId("571786ca7fdc0dc3e0631311"),
 “CP”: "10280",
 “ciudad": “Madrid",
 “comunidad": “Comunidad de Madrid",
 “barrio": “Chamberi",
 “poblacion": 550074,
 “bibliotecas”: [
{ “nombre”: “María Zambrano”, “libros”: 30123},
{ “nombre”: “Eduardo Lúpulo”, “libros”: 12883}
 ]
}
```

```
[
 {
 "v" : 2,
 "key" : {
 "_id" : 1
 },
 "name" : "_id_",
 "ns" : "stats.zips"
 }
]
```

**Pregunta 1 (0.5 puntos) – Una vez abierto un terminal y conectado a la Mongo Shell, me conecta por
defecto a la base de datos de nombre “test”. Conéctese a la base de datos comentada anteriormente
y ejecute una query para buscar los barrios con el código postal 10280 o 10290:**

```
use test
db.stats.find({
$or:[ 
  { CP: {$eq: 10280} },
  { CP: {$eq: 10290} }
]
})
```

**Pregunta 2 (0.5 puntos) – Obtenga con un comando de la Mongo Shell los barrios que no tengan
bibliotecas (estos pueden ser los que su array bibliotecas tenga tamaño 0 o directamente no tengan
array bibliotecas).**

```
db.stats.find({
$or:[ 
  { bibliotecas: {$eq: []} },
  { bibliotecas: {$size: 0} }
]
})
```

**Pregunta 3 (0.5 puntos) – Busque con una única query los barrios que cumplan lo siguiente:** 
- **o que sean de la Comunidad de Madrid** 
- **o que no sean de la Comunidad de Madrid pero que tengan más de 30000 personas**

```
db.stats.find({
$or:[ 
  {comunidad: {$eq: Comunidad de Madrid} },
  {$and:[
    {comunidad: {$neq: Comunidad de Madrid} },
    {poblacion: {$gt: 30000} }
    ]
]
})
```

**Pregunta 4 (0.5 puntos) – Añada un campo nuevo con nombre “superpoblado” y su valor debe ser
true para los barrios que tengan igual o más a 10000 personas y false para los barrios que tengan
menos de 10000 personas.**

```
db.stats.update({
  { $gte: 10000 },
  { $set: {superpoblado: "true"} }
})

db.stats.update({
  { $lt: 10000 },
  { $set: {superpoblado: "false"} }
})
```

**Pregunta 5 (0.5 puntos) – Una aplicación que accede a esta base de datos se ha quejado que va muy
muy lenta. Con el comando db.setProfilingLevel(1, 300) conseguimos logear queries lentas y vemos
que hay miles de queries muy lentas que acceden al campo “CP”. ¿Cómo podríamos mejorar la
performance de la base de datos? Indique el comando con el que lo arreglaría (sabemos que el
campo “CP” es único, no hay dos documentos con dicho campo igual):**

Se podría mejorar la performance de la bbdd creando un índice sobre el campo CP, sabiendo que es único.

db.stats.createIndex()

```

```

**Pregunta 6 (1 punto) – Tenemos la siguiente query del aggregation framework que consta de 4 fases:**

```
db.zips.aggregate([
 { $group: {
 _id: { comunidad: "$comunidad", ciudad: "$ciudad" },
 pop: { $sum: "$poblacion" }
 }
 },
 { $sort: { pop: 1 } },
 { $group: {
 _id: "$_id.comunidad",
 biggestCity: { $last: "$_id.ciudad" },
 biggestPop: { $last: "$pop" },
 smallestCity: { $first: "$_id.ciudad" },
 smallestPop: { $first: "$pop" }
 }
 },
 { $project: {
 _id: 0,
 state: "$_id",
 biggestCity: { name: "$biggestCity", pop: "$biggestPop" },
 smallestCity: { name: "$smallestCity", pop: "$smallestPop" }
 }
 }
])
```

***Explique qué hace cada una de las fases y ponga un ejemplo de documento resultado de cada una
de las fases.***

La query consta de cuatro fases: dos groups, un sort y un project

en el primer group se agrupan los documentos con la misma comunidad y ciudad y se crea un campo pop suma de las poblaciones en la agrupación

```
{
 "_id": ObjectId("571786ca7fdc0dc3e0631317"),
 “ciudad": “Madrid",
 “comunidad": “Comunidad de Madrid",
 “pop": 3000000,
}
```

A continuación se ordena de menor a mayor en función de esta suma.

Después, para cada comunidad se realiza otra agrupación sobre el id comundiad, creanod propiedades biggestCity que almecna el idea de la última ciudad, la de mayor población, y de biggestPop, que alamacena su población; y por otro lado las propiedaded de smallestCity y smallestProp

```
{
 "_id": ObjectId("571786ca7fdc0dc3e0631317"),
 “comunidad": “Comunidad de Madrid",
 “pop": 3000000,
 "biggestCity": Madrid,
 "biggestPop": 3000000,
 "smallestCity": "Jaramillo de Odón" , //  Pueblo random que me he inventado
 "smallestPop": 3
 }
```

Finalmente, se proyectan estos campos en un documento con una forma parecida a esta:

```
{
 _id: 0,
 state: "ObjectId("571786ca7fdc0dc3e0631317")",
 biggestCity: { name: "Madrid", pop: 3000000 },
 smallestCity: { name: "Jaramillo de Odón", pop: 3 }
 }
```

**Pregunta 6 (1 punto dividido en 3 apartados) –
Una compañía financiera, dispone de un servicio RESTful Web Server para realizar operaciones CRUD
sobre clientes de la empresa. Dicha compañía utiliza mongoDB como sistema de almacenamiento de
base de datos. Actualmente, toda la información está almacenada en una única instancia de
mongoDB en la base de datos con nombre “company” y colección “clients”. Como medida para
asegurar la disponibilidad de acceso a los datos, se plantea replicar los datos en múltiples
contenedores con instancias de mongoDB. Pare ellos se muestra el siguiente esquema de despliegue:**

![Arquitectura](https://raw.githubusercontent.com/ablazleon/Preparando-bdfi/main/Screenshot%202021-01-17%20104023.png)

**El personal técnico de la empresa se ha encargado ya de definir los servicios de mongo que se
necesitan desplegar, por lo que ellos proveen un fichero docker-compose con una serie de servicios
preconfigurados para su despliegue. Sin embargo, ellos desconocen cómo se deben configurar las
instancias de mongo para que la replicación se haga efectiva. Por lo tanto, se pide:**

**A) (0.5 puntos) Asignar los valores correspondientes al fichero configure-replica.sh para que se
conecte al contenedor mongo-primary e inicie el Replica Set con los valores correspondientes
a los servicios mostrados en la figura anterior.**

**Fichero configure-replica.sh:**

```
#!/bin/bash
# wait until the databases are up and running
sleep 3
# configure the replica set
mongo --host <host> --eval "rs.initiate({
_id : '<replica-set-id>',
protocolVersion: 1,
members: [ { _id : <member_id>, host : '<member_host>'},
{ _id : <member_id>, host : '<member_host>'},
{ _id : <member_id>, host : '<member_host>'}
],
settings: {electionTimeoutMillis: 1000}
})"

```

*En nuestro año no se planteó el despleigue de mongo en replicación con contenedores sino con servicios, de forma que se debe modificar el comando ejemplo para adapatarlo a la práctica.*

```

mongo --host localhost:27001

rs.initiate({
_id : 'my-mongo-set',
members: [ 
{ _id : 0, host : 'localhost:27002', priority: 900},
{ _id : 1, host : 'localhost:27003'},
{ _id : 2, host : 'localhost:27004', priority: 0, slaveDelay: 60}
]
})
```

**C) (0.2 puntos) Suponiendo que la colección “clients” creciese considerablemente, indique
brevemente cómo se podría mejorar el despliegue para que se mejore la velocidad de acceso a los
datos.**

**Pregunta 7 (1.5 puntos en dos apartados): Sobre la aplicación realizada en la última práctica de la
asignatura, que utiliza PouchDB y CouchDB:**
**a) Dibuje y explique la arquitectura. (1 punto)**
**b) Cómo funciona la aplicación cuando no hay conexión de red, es decir cuando el terminal que
accede a la aplicación no tiene datos y hace cambios sobre la aplicación. Explique
(ayudándose si lo desea de su dibujo del apartado a) cómo funciona cuando no hay conexión
de red y qué ocurre cuando se vuelve a estar online. (0,5 puntos)**

**Pregutnas extra de mi cosecha**

**(0.5 pto) En qué se basa la seguridad de mongo**

**(0.5 pto) ¿Se puede decir que mongo sea una bbdd durable?**

**(0.5 pto) En qué consiste el paradigma PACELC? Cuál es su diferencia frente a CAP?**
