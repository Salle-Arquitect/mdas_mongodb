Dada la base de datos de restaurantes que en la que en la teoría se describe como importar, escribe como realizar las consultas que se proponen a continuación.


# 1. ¿Cuantos restaurantes hay en la base de datos? Indica cuantos y como has obtenido el resultado.
Hay un total de 25359 en la base de datos.
He obtenido el resultado con el siguiente comando de mongo:
```mongo
> db.restaurants.count()
25359
```

# 2. Busca los restaurantes donde sirvan Tea y otras cosas.
## Investigando
```mongo
> db.restaurants.distinct("cuisine")
```

## Resultado
```mongo
> db.restaurants.find({"cuisine":/\bTea\b/}).pretty()
```

# 3. Busca los restaurantes donde sirvan Tea y otras cosas y no sean Starbucks Coffee
## Investigando
```mongo
> db.restaurants.find({"cuisine":/\bTea\b/}).count()
1216

> db.restaurants.find({"cuisine":/\bTea\b/, "name":/Starbucks Coffee/}).count()
259

1216 - 259 = 957

> db.restaurants.find({$and:[{"cuisine":/\bTea\b/},{"name":{$not:/Starbucks Coffee/}}]}).count()
957
```
Hay un total de 1216 restaurantes que sirvan Tea.
Un total de 259 restaurantes que sirvan Tea siendo de Starbucks Coffee.
Esto implica que hay un total de 957 restaurantes que sirven Tea que no son de Starbucks Coffee.
## Resultado
```mongo
> db.restaurants.find({$and:[{"cuisine":/\bTea\b/},{"name":{$not:/Starbucks Coffee/}}]}).pretty()
```

# 4. De la consulta anterior ahora debe devolver solo el name y el borough.
```mongo
> db.restaurants.find({$and:[{"cuisine":/\bTea\b/},{"name":{$not:/Starbucks Coffee/}}]}, {"name":1, "_id":0,"borough":1})
```

5.	Realiza una consulta que devuelva los restaurantes con puntuaciones mayores al 30-11-2014
```
> db.restaurants.find({ "grades.date": {$gt:ISODate("2014-03-11") } }).count()
22233
```

6.	Realiza una consulta que devuelva el restaurantes con puntuaciones mayores al 11-03-2014, tenga una grade igual a “A” y sea de comida Chinese.
7.	Realiza una consulta que devuelva el restaurante con puntuaciones mayores al 10-01-2013 y que esa puntuación tenga una grade igual a “B” y a su vez esa puntuación tenga un score de 5 y que ese restaurante sea de comida Chinese.
8.	Realiza una consulta que muestre los distintos tipos de restaurantes (cuisine) que hay (sin utilizar distinct).
9.	Realiza una consulta que muestre cuantos restaurantes hay en Manhattan por cada tipo de cocina ordenado la cantidad de restaurantes descendientemente.
10.	Realiza una consulta que muestre los 5 restaurantes con la puntuación media (score) más alta.
11.	Realiza una consulta muestra los borough que tienen entre 40 y 60 restaurantes.
12.	Cual seria el comando para insertar un nuevo restaurante, con nombre “Pa amb Tomaquet”, con cocina Mediterranea, con un grade creado hoy con puntuación 15 y grade A.
13.	¿Cuál seria el comando para cambiar todos los restaurantes de comida Italian a Mediterranea?
14.	¿Cuál seria el comando para borrar todos los restaurantes de comida Mediterranea?
15.	¿Con que comando borrarías toda la colección restaurants?
