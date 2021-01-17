# Preparando-bdfi

# Examen 2019 enero

**Pregunta 1 (1pt) - ¿Para qué sirven los ODMs?. ¿Qué ventajas tiene utilizarlos?**

**Pregunta 2 (1pt) - Teorema de CAP (o de Brewer). Enuncielo y diga qué significan la C, la A y la P**

**Pregunta 3 (1pt) - ¿En qué consiste el particionamiento (o Sharding) de una Base de Datos? Enumere
2 ventajas**

**Pregunta 4 (1pt) – Data Value Pyramid o pirámide de valor de los datos. Indique qué es y que tiene
cada uno de los niveles. (Si quiere puede ayudarse dibujando la pirámide en cuestión).**

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

**Pregunta 2 (0.5 puntos) – Obtenga con un comando de la Mongo Shell los barrios que no tengan
bibliotecas (estos pueden ser los que su array bibliotecas tenga tamaño 0 o directamente no tengan
array bibliotecas).**

**Pregunta 3 (0.5 puntos) – Busque con una única query los barrios que cumplan lo siguiente:** 
- **o que sean de la Comunidad de Madrid** 
- **o que no sean de la Comunidad de Madrid pero que tengan más de 30000 personas**


**Pregunta 4 (0.5 puntos) – Añada un campo nuevo con nombre “superpoblado” y su valor debe ser
true para los barrios que tengan igual o más a 10000 personas y false para los barrios que tengan
menos de 10000 personas.**

**Pregunta 5 (0.5 puntos) – Una aplicación que accede a esta base de datos se ha quejado que va muy
muy lenta. Con el comando db.setProfilingLevel(1, 300) conseguimos logear queries lentas y vemos
que hay miles de queries muy lentas que acceden al campo “CP”. ¿Cómo podríamos mejorar la
performance de la base de datos? Indique el comando con el que lo arreglaría (sabemos que el
campo “CP” es único, no hay dos documentos con dicho campo igual):**

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


