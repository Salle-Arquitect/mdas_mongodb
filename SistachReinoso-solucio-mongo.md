Dada la base de datos de restaurantes que en la que en la teoría se describe como importar, escribe como realizar las consultas que se proponen a continuación.

# 1. ¿Cuantos restaurantes hay en la base de datos? Indica cuantos y como has obtenido el resultado.
Hay un total de 25359 en la base de datos.
He obtenido el resultado con el siguiente comando de mongo:
```mongo
db.restaurants.count()
```

# 2. Busca los restaurantes donde sirvan Tea y otras cosas.
## Investigando
```mongo
db.restaurants.distinct("cuisine")
```

## Resultado
```mongo
db.restaurants.find({ "cuisine" : /\bTea\b/ }).pretty()
```

# 3. Busca los restaurantes donde sirvan Tea y otras cosas y no sean Starbucks Coffee
## Investigando
```mongo
db.restaurants.find({ "cuisine" : /\bTea\b/ }).count()
# output: 1216

db.restaurants.find(
  {
    "cuisine" : /\bTea\b/,
    "name" : /Starbucks Coffee/
  }
).count()
# output:  259

# 1216 - 259 = 957

db.restaurants.find(
  {
    "cuisine" : /\bTea\b/,
    "name" : { "$not" : /Starbucks Coffee/ }
  }
).count()
# output 957
```
Hay un total de 1216 restaurantes que sirvan Tea.
Un total de 259 restaurantes que sirvan Tea siendo de Starbucks Coffee.
Esto implica que hay un total de 957 restaurantes que sirven Tea que no son de Starbucks Coffee.
## Resultado
```mongo
db.restaurants.find(
  {
    "cuisine" : /\bTea\b/,
    "name" : { "$not" : /Starbucks Coffee/ }
  }
).pretty()
```

# 4. De la consulta anterior ahora debe devolver solo el name y el borough.
```mongo
db.restaurants.find(
  {
    "cuisine" : /\bTea\b/,
    "name" : { "$not" : /Starbucks Coffee/ }
  },
  {
    "_id" : 0,
    "name" : 1,
    "borough" : 1
  }
).pretty()
```

# 5. Realiza una consulta que devuelva los restaurantes con puntuaciones mayores al 30-11-2014
```mongo
db.restaurants.find(
  {
    "grades.date" : { "$gt" : ISODate("2014-11-30") }
  }
).pretty()
```

# 6. Realiza una consulta que devuelva el restaurantes con puntuaciones mayores al 11-03-2014, tenga una grade igual a “A” y sea de comida Chinese.
## Investigación
```mongo
db.restaurants.find(
  {
    "grades.date" : { "$gt" : ISODate("2014-03-11") },
    "grades.grade" : "A",
    "cuisine": "Chinese"
  }
).count()
# output: 2061

db.restaurants.find(
  {
    "grades.date" : { "$gt" : ISODate("2014-03-11") },
    "grades.grade" : "A",
    "cuisine": /\bChinese\b/
  }
).count()
# output: 2116
```
Nota: se pide que sea comida `Chinese`, no que solo sea `Chinese`.

## Resultado
```mongo
db.restaurants.find(
  {
    "grades.date" : { "$gt" : ISODate("2014-03-11") },
    "grades.grade" : "A",
    "cuisine": /\bChinese\b/
  }
).pretty()
```

# 7. Realiza una consulta que devuelva el restaurante con puntuaciones mayores al 10-01-2013 y que esa puntuación tenga una grade igual a “B” y a su vez esa puntuación tenga un score de 5 y que ese restaurante sea de comida Chinese.
## Investigación
```mongo
# No funciona, ya que no cumple las dos condiciones el mismo valor
db.restaurants.findOne({"grades.date":{"$gt":ISODate("2014-01-10")}})

# Probamos con la notación elemMatch, usamos notación de tabulación para hacerlo más leíble
db.restaurants.findOne(
  {
    "grades": {
      "$elemMatch" : {
        "date" : { "$gt" : ISODate("2013-01-10") }
      }
    }
  }
)

# Ahora sí que cumple las dos condiciones de igual forma
db.restaurants.findOne(
  {
    "grades": {
      "$elemMatch" : {
        "date" : { "$gt" : ISODate("2013-01-10") },
        "grade" : "B"
      }
    }
  }
)

# Añadimos la condición del scrore de 5
db.restaurants.findOne(
  {
    "grades": {
      "$elemMatch" : {
        "date" : { "$gt" : ISODate("2013-01-10") },
        "grade" : "B",
        "score" : 5
      }
    }
  }
)

# Última condición, que sea un restaurante Chino
db.restaurants.findOne(
  {
    "grades": {
      "$elemMatch" : {
        "date" : { "$gt" : ISODate("2013-01-10") },
        "grade" : "B",
        "score" : 5
      }
    },
    "cuisine": /\bChinese\b/
  }
)
```

## Resultado
```mongo
db.restaurants.find(
  {
    "grades": {
      "$elemMatch" : {
        "date" : { "$gt" : ISODate("2013-01-10") },
        "grade" : "B",
        "score" : 5
      }
    },
    "cuisine": /\bChinese\b/
  }
).pretty()
```

# 8. Realiza una consulta que muestre los distintos tipos de restaurantes (cuisine) que hay (sin utilizar distinct).
## Investigando
```mongo
db.restaurants.distinct("cuisine").length
# output: 85

# El count aquí no funciona, en ser solo 85 elementos los he comparado a mano y coinciden.
db.restaurants.aggregate([
  { "$group" : { "_id" : "$cuisine" } },
  { "$sort" : { "_id" : 1 } }
])
```

## Resultado
```mongo
db.restaurants.aggregate([
  { "$group" : { "_id" : "$cuisine" } }
])
```

# 9. Realiza una consulta que muestre cuantos restaurantes hay en Manhattan por cada tipo de cocina ordenado la cantidad de restaurantes descendientemente.
## Investigación
```mongo
db.restaurants.findOne()

db.restaurants.distinct("borough").length
# output: 6, Manhattan no aparece duplicado.
```

## Resultado
```mongo
db.restaurants.aggregate([
  { "$match" : { "borough" : "Manhattan" } },
  {
    "$group" : {
      "_id" : "$cuisine",
      "count" : { "$sum" : 1 }
    }
  },
  { "$sort" : { "count" : -1 } }
])
```

# 10. Realiza una consulta que muestre los 5 restaurantes con la puntuación media (score) más alta.
## Investigación
```mongo
db.restaurants.findOne()
```
- muestre los 5 restaurantes: `{ "$limit" : 5 }`
- con la puntuación media (score): `"scoreAvg" : { "$avg" : "$grades.score" }`
- más alta: `{ "$sort" : { "scoreAvg" : -1 } }`

## Resultado
```mongo
db.restaurants.aggregate([
  { "$project" : {
    "scoreAvg" : { "$avg" : "$grades.score" },
    "name" : "$name"
  }},
  { "$sort" : { "scoreAvg" : -1 } },
  { "$limit" : 5 }
])
```

# 11. Realiza una consulta muestra los `borough` que tienen entre 40 y 60 restaurantes.
```mongo
db.restaurants.aggregate([
  { "$group" : {
    "_id" : "$borough",
    "count" : { "$sum" : 1 }
  }},
  { "$match" : {
    "count" : { "$gt" : 40, "$lt" : 60 }
  }}
])
```

12.	Cual seria el comando para insertar un nuevo restaurante, con nombre “Pa amb Tomaquet”, con cocina Mediterranea, con un grade creado hoy con puntuación 15 y grade A.
13.	¿Cuál seria el comando para cambiar todos los restaurantes de comida Italian a Mediterranea?
14.	¿Cuál seria el comando para borrar todos los restaurantes de comida Mediterranea?
15.	¿Con que comando borrarías toda la colección restaurants?
