USUARIOS INGRESANDO EN MÚLTIPLES EQUIPOS CON LA MISMA CUENTA

En esta consulta se intenta detectar los logins exitosos de un mismo usuario a múltiples host. Esto es importante para detectar posibles movimientos laterales y la reutilización de credenciales en diferentes equipos. 
La estructura básica es la siguiente:

index=<index_de_Logs_de_Windows> EventCode=<codigo_buscado> earliest="<rangodetiempo>"

| stats dc(ComputerName) as <variable_sumadeComputadoras> by Account_Name

| where <variable_sumadeComputadoras> > 2


Se define el index donde se encuentran los logs buscados. Seleccionamos el código de evento, en este caso, el 4624 que es el login exitoso. En el código agregamos un período de tiempo con la variable "earliest", pero también podemos hacerlo por medio del selector de tiempo.
Después utilizamos una tubería "|" para que el resultado de la búsqueda presente las siguientes características:

"stats" nos permite utilizar funciones estadísticas como dc (distinct count) que nos permitirá obtener los resultados únicos en el nombre de computadoras por cada uno de los nombres de cuentas. Como se puede observar en la expresión:
| stats dc(ComputerName) as <variable_sumadeComputadoras> by Account_Name
Esta expresión realizará la cuenta de cada una de las diferentes computadoras a las que cada nombre de cuenta ingrese.

Sin embargo, aquí debemos agregar la condición:
| where <variable_sumadeComputadoras> > 2

Que, básicamente, permite definir que solo se presenten los resultados DONDE se cumpla que la condición de conteo de nombre de computadoras únicas (ComputerName) sea mayor a 2 (este valor puede llegar a variar según las condiciones). 
Así que, para conocer si existen nombres de cuenta ingresando a diferentes equipos de nuestro domino podemos utilizar lo siguiente:

index=winevenlog EventCode=4624 earliest=-72hr

| stats dc(ComputerName) as hosts_logged by Account_Name

| where hosts_logged > 2

Aquí se define un período de tiempo de 72 horas para identificar si existen cuentas que hayan ingresado a más de 2 computadora.
Esta búsqueda puede resultar de utilidad para conocer si existen cuentas que estén ingresando a diferentes equipos y determinar si esta actividad es normal o requiere un análisis detallado. 
