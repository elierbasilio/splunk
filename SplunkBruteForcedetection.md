
**Consulta de detección de ataque de fuerza bruta en Splunk**

Los intentos de fuerza bruta van dejando un trazo de actividad que podemos identificar utilizando variables de tiempo.

Por ejemplo, en un ecosistema Windows podemos utilizar los eventos de login 4624 y 4625 para detectar los intentos de inicio de sesión exitosos y fallidos.

A continuación, vamos a observar la construcción de una consulta que puede ayudarnos a su identificación. En Splunk podemos utilizar la siguiente:

![](Aspose.Words.0544f7f7-6a49-42c7-b07c-3b8a1248070a.001.png)

Aquí, se presenta una consulta básica sobre el index con los logs de sistema seleccionado y se utiliza el tipo de evento "EventCode" 4624 para la identificación de logins exitosos. 

Después, utilizamos la barra vertical "|" seguida del comando stats para utilizar las funciones de agregación como count y así contar por los nombres de usuario "Account\_Name".  

Esta es la consulta básica que nos arrojaría todos los resultados del evento en las últimas 24 horas ya que el controlador del tiempo está definido en este rango. 

Sin embargo, hasta este momento nuestra consulta solo nos dice cuántas veces se presentó el evento en este período. Pero para un ataque de fuerza bruta tenemos que agregar rangos de tiempo de incidencia para poder detectar si hay un número inusual de intentos fallidos o exitosos en nuestro sistema. 

Así que podemos agregar lo siguiente a nuestra consulta:

![](Aspose.Words.0544f7f7-6a49-42c7-b07c-3b8a1248070a.002.png) 

Seguimos utilizando el comando stats, pero hemos agregado algunas variables adicionales:

| stats count as variable min(\_time) as variable max(\_time) as variable by Account\_Name

Para comenzar, hemos definido un nombre de variable "logins" para el resultado de las cuentas de incidencias por nombre de cuenta. La siguiente imagen muestra los elementos que afectan este comportamiento:

![](Aspose.Words.0544f7f7-6a49-42c7-b07c-3b8a1248070a.003.png)

Pero, además, hemos agregado nuevas variables de tiempo: min(\_time) y max(\_time)

![](Aspose.Words.0544f7f7-6a49-42c7-b07c-3b8a1248070a.004.png)

Estas variables las hemos definido con el nombre first y last, respectivamente, que son nombres representativos que hemos seleccionado para su identificación y así, será más fácil hacer referencia al resultado en posteriores operaciones.   


**¿Pero qué registran estas variables?**

Lo que estamos logrando con estas variables es obtener el tiempo de la primera min(\_time) y última aparición del evento max(\_time) por cada nombre de cuenta. De esta manera podemos obtener un rango.

El elemento \_time es el tiempo de registro del evento en formato de época de UNIX. Utilizamos las funciones de mínimo (min) y máximo (max) para obtener las mencionadas apariciones. 

Así que, básicamente, de esta manera, no solo se nos presenta cuantas veces sucedió el tipo de evento, sino en qué tiempo se registró la primera y última incidencia. 

Aquí es importante señalar que los resultados presentados representan los eventos registrados en las últimas 24 horas que es nuestro período inicial de consulta, aunque este valor lo podemos modificar para abarcar el tiempo de búsqueda deseado. 

![](Aspose.Words.0544f7f7-6a49-42c7-b07c-3b8a1248070a.005.png) 

**Definiendo condiciones de aparición**

Ahora, va a ser importante que podamos utilizar este rango para determinar el número de apariciones del evento en un período determinado.

Para esto, podemos utilizar el comando EVAL que, básicamente nos ayudará a la creación de variables. En este caso, utilizaremos lo siguiente:

eval duration = last – first

Con esto, estamos definiendo una nueva variable "duration" que nos permitirá presentar el resultado de la operación de resta de las variables "last" y "first" en una nueva columna, tal y como se presenta en la siguiente captura:

![](Aspose.Words.0544f7f7-6a49-42c7-b07c-3b8a1248070a.006.png)

Aquí, observamos cómo la nueva columna "duration" aparece en el resultado y muestra la diferencia entre la primera y última aparición del evento por cada una de las cuentas. 

Pero ahora será necesario agregar una condición que nos permitirá detectar si hay alguna anomalía. Tendremos que definir el rango de tiempo y la cantidad de eventos que deberían alertarnos sobre actividad sospechosa. 

En este caso, agregamos el comando "where" para definir la condición. Así que podríamos definir que, si se presentan más de 4 eventos en menos de 10 minutos (600 segundos), se presenten los resultados. Para esto, agregamos lo siguiente:

| where duration < 600 and logins > 4

Esto desplegará solo los resultados donde se presenten más de 4 incidencias en un tiempo menor de 10 minutos.  

Vamos a probar esta consulta con el evento 4625 que registra los intentos de logins fallidos y modificaré el tiempo base de consulta a "All time" para poder observar los resultados. 

![](Aspose.Words.0544f7f7-6a49-42c7-b07c-3b8a1248070a.007.png)

Ahora podemos observar los intentos fallidos de login en el sistema y debe llamar la atención la gran cantidad de intentos fallidos que se presentan, pues estos son el resultado de pruebas de un escaneo de vulnerabilidades realizadas al sistema, así que podemos observar una gran cantidad de intentos en un período muy corto de tiempo. 

Podemos ordenar los resultados por el nombre de la cuenta con más intentos fallidos agregando el comando "sort" haciendo referencia a la cuenta (en este caso asignada a la variable "failed\_logins"):

![](Aspose.Words.0544f7f7-6a49-42c7-b07c-3b8a1248070a.008.png)

En la captura se pueden observar los resultados ordenados por el parámetro definido.

**Conclusiones**

Este tipo de consulta nos permite observar los eventos con tiempo y condiciones específicas, lo que puede ser de mucha utilidad para obtener información relevante de los eventos de nuestros sistemas. Su importancia radica en que no solo estamos revisando un periodo de tiempo sino si se cumple la condición de número de incidencias en un rango de tiempo dentro de ese período. De esta manera estamos depurando información que nos permitiría definir alertas o definir acciones específicas de respuesta. 



Importancia

Buscar eventos en nuestro SIEM con base en una serie de condiciones en un periodo específico de tiempo es una de las acciones principales que debemos dominar para obtener información relevante que permita la toma de decisiones.

A continuación, vamos a revisar cómo podemos realizar una búsqueda específica de eventos y condiciones utilizando SPLUNK.

Supongamos que deseamos conocer cuántos usuarios ingresaron a los equipos en un periodo determinado de tiempo (por ejemplo 10 minutos) para monitorear sobre un posible ataque de fuerza bruta.

En SPLUNK tendríamos que comenzar a crear la siguiente sentencia:

Index=main EventCode=4625 

Aquí estamos definiendo el index que contiene los registros capturados a través de los forwarders. Además, definimos una búsqueda por eventos 4625 que representan el código en los sistemas Windows para un registro fallido (Failed login) básicamente nos dejan saber que por alguna razón, un usuario no pudo autenticarse de manera correcta (Por ejemplo, ya sea por un error en el nombre de usuario o en la contraseña utilizada)
