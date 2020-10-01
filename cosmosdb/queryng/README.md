# Querying & indexing in Cosmos DB

## Ejercicio 1 Query Overview

La consulta de JSON con SQL permite a Azure Cosmos DB combinar las ventajas de una base de datos relacional heredada con una base de datos NoSQL. Puede utilizar muchas capacidades de consulta ricas como subconsultas o funciones de agregación, pero aún así conserva las muchas ventajas de modelar datos en una base de datos NoSQL.

La base de datos Azure Cosmos sólo soporta elementos JSON estrictos. El sistema de tipos y expresiones está restringido para tratar sólo con tipos JSON. 

## Ejercicio 2 Lanzar nuestra primera consulta 

En este laboratorio, nosotros haremos consulta sobre el contenedor "series".

Empecemos usando querys "basicas" con las cláusulas SELECT, WHERE y FROM

### Task I: Open Data Explorer
1. En el blade de Azure CosmosDB, localiza y seleccion Data Explorer/ Explorador de Datos en la parte izquierda de la pantalla 
2. En la sección de Data Explorer/ Explorador de Datos, expandemos el nodo de marvel, seleccionamos el contenedor series 
3. En el nodo de "series" seleccionamo la opcion de Items
4. Ahora vemos todos los elementos que tiene el contenedor. Observamos todas las propiedades que tiene cada documentos, incluido los arrays
![items marvel](/images/ItemsMarvel.PNG)
5. Seleccionamos New SQL Query
![NewSqlQuery](/images/NewSqlQuery.PNG)
6. Pegamos la siguiente intruccion SQL y ejecutamos la Consulta
```sql
SELECT * FROM c
Where c.startYear=2009 and c.id="7521"
```
7. Veras que la consulta devolvio un único elemento. Explora la estructura de este elemento, ya que es representativa de los elementos de la colección dentro del contenedor series, con los que trabajaremos el resto de sección.

## Ejercicio 3 
Puede elegir qué propiedades del documento proyectar en el resultado usando la notación de puntos. Si quisieras devolver sólo el id del comic podrías ejecutar la consulta de abajo.

Seleccione Nueva consulta SQL. Pegue la siguiente consulta SQL y seleccione Ejecutar consulta.
```sql
SELECT c.id
FROM c
WHERE c.description = "Custom" and c.id = "19244"
```
Aunque es menos común, tambien se puede acceder a las propiedades usando el operador [""]. Por ejemplo, SELECT c.id y SELECT c["id"] son equivalentes. Esta sintaxis es útil para escapar de una propiedad que contiene espacios, caracteres especiales o que tiene el mismo nombre que una palabra clave SQL o una palabra reservada.
```sql
SELECT c["id"]
FROM c
WHERE c["description"] = "Custom" and c["id"] = "19244"
```

## Ejercicio 4
Exploremos las cláusulas de WHERE. Puedes añadir expresiones escalares complejas incluyendo operadores aritméticos, de comparación y lógicos en la cláusula WHERE.

Ejecute la siguiente consulta seleccionando la Nueva Consulta SQL.

Pegue la siguiente consulta SQL y luego seleccione Ejecutar consulta.
```sql
SELECT 
c.title,
c.description,
c.resourceURI,
c.urls,
c.startYear,
c.endYear
FROM c
WHERE (c.description = "Custom" and c.startYear>2009)
```
Esta consulta devolverá el titulo, la descripción, los recursos, la url, en inicio del año y el final del año de la descripción Cutsom y su año de inicio de la serie sera mayor de 2009
El primer resultado sera el siguiente:
![resultEjercicio4](/images/ResultEjercicio4.PNG)

## Ejercicio 5 Proyecciones Avanzadas
La base de datos del Cosmos DB soporta varias formas de transformación en el JSON resultante. Una de las más simples es ponerle un alias a los elementos JSON usando la palabra clave AS  mientras se muestran los resultados.

Ejecutando la consulta de abajo verás que los nombres de los elementos se transforman. Además, la proyección accede sólo al primer elemento de la matriz para todos los elementos especificados por la cláusula WHERE.

Ejecute la siguiente consulta seleccionando la Nueva Consulta SQL.
```sql
SELECT c.title,
c.description,
c.creators.items[0].name AS name,
c.creators.items[0].role AS role
FROM c
WHERE c.creators.items[0].role='writer'
```

## Ejercicio 6 Clausula Order by
La base de datos de Azure Cosmos soporta la adición de una cláusula de ORDER BY para clasificar los resultados basados en una o más propiedades

Ejecute la siguiente consulta seleccionando la Nueva Consulta SQL.

Pegue la siguiente consulta SQL y luego seleccione Ejecutar consulta.
```sql
SELECT c.title,
c.description,
c.creators.items[0].name AS name,
c.creators.items[0].role AS role,
 c.creators.available
FROM c
ORDER BY c.creators.available DESC
```

## Ejercicio 7 Limitar los resultados de la busqueda
La base de datos de Azure Cosmos soporta la palabra clave TOP. TOP puede ser usada para limitar el número de valores de retorno de una consulta.

Ejecuta la consulta de abajo para ver los 20 mejores resultados.
```sql
SELECT TOP 3 c.title,
c.description,
c.creators.items[0].name AS name,
c.creators.items[0].role AS role,
 c.creators.available
FROM c
```

La cláusula OFFSET LIMIT es una cláusula opcional que se puede saltar y luego tomar algunos valores de la consulta. El recuento de OFFSET y el recuento de LIMIT son obligatorios en la cláusula OFFSET LIMIT.

```sql
SELECT c.title,
c.description,
c.creators.items[0].name AS name,
c.creators.items[0].role AS role,
 c.creators.available
FROM c
ORDER BY c.id
OFFSET 3 LIMIT 10
```
Cuando se utiliza el OFFSET LIMIT junto con una cláusula ORDER BY, el conjunto de resultados se produce haciendo saltar y tomar los valores ordenados. Si no se utiliza la cláusula ORDER BY, se producirá un orden determinístico de valores.

## Ejercicio 8 Mas filtros avanzados

Añadamos las palabras clave "IN" y "BETWEEN" en nuestras consultas. IN puede utilizarse para comprobar si un valor especificado coincide con algún elemento de una lista determinada y BETWEEN puede utilizarse para ejecutar consultas contra un rango de valores.

Ejecute la consulta siguiente:
```sql
SELECT c.title,
c.description,
c.creators.items[0].name AS name,
c.creators.items[0].role AS role
FROM c
WHERE c.creators.items[0].role in ('writer','penciller (cover)')
```

## Ejercicio 9 Mas proyecciones
La base de datos de Azure Cosmos soporta la proyección JSON en sus consultas. Proyectemos un nuevo objeto JSON con nombres de propiedades modificados.

Ejecuta la consulta de abajo para ver los resultados.
```sql
SELECT {
   "Titulo": c.title,
   "Descripción": c.description,
   "Name": c.creators.items[0].name,
   "Role": c.creators.items[0].role 
} AS c
FROM c
WHERE c.id="19244"
```














