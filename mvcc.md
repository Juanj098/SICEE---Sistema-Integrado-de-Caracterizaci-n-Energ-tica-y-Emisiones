# Un Breve Estudio de los Algoritmos de Concurrencia Multiversión

Dibyendu Majumdar[cite: 1]
dibyendy@mazumdar.demon.co.uk
Copyright ©2002, 2006
Revisado el 4 de noviembre de 2007

## Resumen
Este documento analiza algunos de los problemas con los métodos tradicionales de control de concurrencia mediante bloqueos, y explica cómo los algoritmos de Concurrencia Multiversión ayudan a resolver algunos de estos problemas[cite: 1]. Describe los enfoques para la concurrencia Multiversión y examina un par de implementaciones con mayor detalle.

## 1 Introducción
Los Sistemas de Gestión de Bases de Datos deben garantizar la consistencia de los datos al tiempo que permiten que múltiples transacciones lean/escriban datos de forma concurrente. Las implementaciones clásicas de SGBD (DBMS) mantienen una única versión de los datos y utilizan bloqueos para gestionar la concurrencia[cite: 1]. Ejemplos de implementaciones de bases de datos de versión única (SVDB) son IBM DB2, Microsoft SQL Server, Apache Derby y Sybase.

Para comprender cómo las implementaciones clásicas de SGBD resuelven los problemas de concurrencia con los bloqueos, es necesario primero comprender el tipo de problemas que pueden surgir.

### 1.1 Problemas de Concurrencia
*   **Lecturas sucias (Dirty reads).** El problema de la lectura sucia ocurre cuando una transacción puede leer datos que han sido modificados pero aún no confirmados por otra transacción.
*   **Actualizaciones perdidas (Lost updates).** El problema de la actualización perdida ocurre cuando una transacción sobrescribe los cambios realizados por otra transacción, porque no se da cuenta de que los datos han cambiado. Por ejemplo, si una transacción T1 lee el registro R1, seguido de lo cual una segunda transacción T2 lee el registro R1, luego la primera transacción T1 actualiza el registro R1 y confirma el cambio, después de lo cual la segunda transacción T2 actualiza R1 y también confirma el cambio. En esta situación, la actualización realizada por la primera transacción T1 al registro R1 se "pierde", porque la segunda transacción T2 nunca la ve.
*   **Lecturas no repetibles (Non-repeatable reads).** El problema de la lectura no repetible ocurre cuando una transacción encuentra que un registro que leyó antes ha sido cambiado por otra transacción.
*   **Lecturas fantasma (Phantom reads).** El problema de la lectura fantasma ocurre cuando una transacción encuentra que la misma consulta SQL devuelve un conjunto diferente de registros en diferentes momentos dentro del contexto de la transacción.

### 1.2 Modos de Bloqueo
Para resolver los diversos problemas de concurrencia, los Sistemas de Bases de Datos Clásicos utilizan bloqueos para restringir el acceso concurrente a los registros por parte de varias transacciones[cite: 1]. Se utilizan dos tipos de bloqueos:
*   **Bloqueos Compartidos (Shared Locks).** Se utilizan para proteger las lecturas de registros[cite: 1]. Los bloqueos compartidos son compatibles con otros bloqueos compartidos pero no con los bloqueos exclusivos[cite: 1]. Por lo tanto, múltiples transacciones pueden adquirir Bloqueos Compartidos en el mismo registro, pero una transacción que desee adquirir un Bloqueo Exclusivo debe esperar a que se liberen los bloqueos compartidos.
*   **Bloqueos Exclusivos (Exclusive Locks).** Se utilizan para proteger las escrituras en los registros. Solo una transacción puede mantener un bloqueo Exclusivo en un registro en cualquier momento, y además, los bloqueos Exclusivos no son compatibles con los Bloqueos Compartidos. Por lo tanto, un registro protegido por un Bloqueo Exclusivo impide tanto la lectura como la escritura por parte de otras transacciones.

Los diversos modos de bloqueo utilizados por el SGBD, también llamados niveles de Aislamiento, se detallan a continuación:
*   **Lectura Confirmada (Read Committed).** En este modo, la SVDB coloca bloqueos exclusivos con duración de confirmación en cualquier dato que escriba. Se adquieren bloqueos compartidos en los registros que se están leyendo, pero estos bloqueos se liberan tan pronto como termina la lectura. Este modo evita las lecturas sucias, pero permite que ocurran problemas de actualizaciones perdidas, lecturas no repetibles y lecturas fantasma.
*   **Estabilidad del Cursor (Cursor Stability).** Además de los bloqueos utilizados para la Lectura Confirmada, el SGBD retiene el bloqueo compartido en el "registro actual" hasta que el cursor se mueve a otro registro[cite: 1]. Este modo evita lecturas sucias y actualizaciones perdidas.
*   **Lectura Repetible (Repeatable Read).** Además de los bloqueos exclusivos utilizados para la Lectura Confirmada, el SGBD coloca bloqueos compartidos de duración de confirmación en los elementos de datos que se han leído. Este modo evita el problema de las lecturas no repetibles.
*   **Serializable.** Además de los bloqueos utilizados para la Lectura Repetible, cuando las consultas se ejecutan con un parámetro de búsqueda, el SGBD utiliza bloqueos de rango de claves para bloquear incluso datos inexistentes que satisfarían los criterios de búsqueda. Por ejemplo, si una consulta SQL ejecuta una búsqueda de ciudades que comienzan con la letra 'A', todas las ciudades que comienzan con 'A' se bloquean en modo compartido, aunque algunas de ellas no existan físicamente en la base de datos[cite: 1]. Esto evita que otras transacciones creen nuevos elementos de datos que satisfarían los criterios de búsqueda de la consulta hasta que la transacción que ejecuta la consulta confirme o aborte.

Claramente, los diferentes modos de bloqueo ofrecen diferentes niveles de consistencia de datos, intercambiando rendimiento y concurrencia por una mayor consistencia. El modo de Lectura Confirmada ofrece la mayor concurrencia pero la menor consistencia. El modo Serializado ofrece la vista más consistente de los datos, pero la concurrencia más baja debido a los bloqueos a largo plazo que mantiene una transacción operando en este modo.

## 2 Problemas con la concurrencia tradicional basada en bloqueos
Independientemente del modo de bloqueo utilizado, el problema con los sistemas SVDB es que los Escritores siempre bloquean a los Lectores[cite: 1]. Esto se debe a que todas las escrituras están protegidas por bloqueos exclusivos de duración de confirmación, que impiden a los Lectores acceder a los datos que han sido bloqueados[cite: 1]. Los Lectores deben esperar a que los Escritores confirmen o aborten sus transacciones.

En todos los modos de bloqueo distintos de la Lectura Confirmada, los Lectores también bloquean a los Escritores.

## 3 Introducción a la Concurrencia Multiversión
El objetivo de la Concurrencia Multiversión es evitar el problema de que los Escritores bloqueen a los Lectores y viceversa, haciendo uso de múltiples versiones de los datos.

El problema de que los Escritores bloqueen a los Lectores se puede evitar si los Lectores pueden obtener acceso a una versión anterior de los datos que están bloqueados por los Escritores para su modificación.

El problema de que los Lectores bloqueen a los Escritores se puede evitar asegurando que los Lectores no obtengan bloqueos sobre los datos.

La Concurrencia Multiversión permite a los Lectores operar sin adquirir ningún bloqueo, aprovechando el hecho de que si un Escritor ha actualizado un registro en particular, su versión anterior puede ser utilizada por el Lector sin esperar a que el Escritor Confirme o Aborte. En una solución de Concurrencia Multiversión, los Lectores no bloquean a los Escritores, y viceversa.

Si bien la concurrencia multiversión mejora la concurrencia de la base de datos, su impacto en la consistencia de los datos es más complejo.

## 4 Requisitos de los sistemas de Concurrencia Multiversión
Como su nombre lo indica, la concurrencia multiversión depende de múltiples versiones de datos para lograr mayores niveles de concurrencia. Típicamente, un SGBD que ofrece concurrencia multiversión (MVDB), necesita proporcionar las siguientes características:
1.  El SGBD debe ser capaz de recuperar versiones más antiguas de una fila.
2.  El SGBD debe tener un mecanismo para determinar qué versión de una fila es válida en el contexto de una transacción[cite: 1]. Por lo general, el SGBD solo considerará una versión que se confirmó antes del inicio de la transacción que está ejecutando la consulta[cite: 1]. Para determinar esto, el SGBD debe saber qué transacción creó una versión particular de una fila, y si esta transacción se confirmó antes del inicio de la transacción actual.

## 5 Desafíos en la implementación de un SGBD multiversión[cite: 1]
1.  Si se almacenan múltiples versiones en la base de datos, se requiere un mecanismo eficiente de recolección de basura para deshacerse de las versiones antiguas cuando ya no son necesarias.
2.  El SGBD debe proporcionar métodos de acceso eficientes que eviten buscar en versiones redundantes.
3.  El SGBD debe evitar búsquedas costosas al determinar el tiempo de confirmación relativo de una transacción.

## 6 Enfoques para la Concurrencia Multiversión
Existen esencialmente dos enfoques para la concurrencia multiversión. El primer enfoque es almacenar múltiples versiones de registros en la base de datos, y recolectar como basura los registros cuando ya no son necesarios[cite: 1]. Este es el enfoque adoptado por PostgreSQL y Firebird/Interbase.

El segundo enfoque es mantener solo la última versión de los datos en la base de datos, como en las implementaciones SVDB, pero reconstruir las versiones antiguas de los datos dinámicamente según sea necesario explotando la información dentro del Registro de Escritura Anticipada (Write Ahead Log). Este es el enfoque adoptado por Oracle y MySQL/InnoDB.

El resto de este documento analiza con mayor detalle las implementaciones de PostgreSQL y Oracle de la concurrencia multiversión.

## 7 Concurrencia Multiversión en PostgreSQL
PostgreSQL es la encarnación de Código Abierto de Postgres. Postgres fue desarrollado en la Universidad de California, Berkeley, por un equipo liderado por el Prof. Michael Stonebraker (famoso por INGRES). La implementación original de Postgres ofrecía una base de datos multiversión con recolección de basura[cite: 1]. Sin embargo, utilizaba el modelo tradicional de bloqueo de dos fases que conducía al fenómeno de "los lectores bloquean a los escritores".

El propósito original de múltiples versiones en la base de datos era permitir el viaje en el tiempo, y también evitar la necesidad de un Registro de Escritura Anticipada. Sin embargo, en PostgreSQL se ha eliminado el soporte para el viaje en el tiempo, y la tecnología multiversión en el Postgres original se explota para implementar un algoritmo de concurrencia Multiversión. El equipo de PostgreSQL también agregó bloqueo a nivel de fila y un Registro de Escritura Anticipada al sistema.

En PostgreSQL, cuando se actualiza una fila, se crea una nueva versión (llamada tupla) de la fila y se inserta en la tabla. A la versión anterior se le proporciona un puntero a la nueva versión. La versión anterior se marca como "expirada", pero permanece en la base de datos hasta que se recolecta como basura.

Para soportar el multiversionado, cada tupla tiene datos adicionales registrados en ella:
*   **xmin**: El ID de la transacción que insertó/actualizó la fila y creó esta tupla.
*   **xmax**: La transacción que eliminó la fila, o creó una nueva versión de esta tupla. Inicialmente, este campo es nulo.

Para rastrear el estado de las transacciones, se mantiene una tabla especial llamada PG_LOG. Dado que los IDs de Transacción se implementan usando un contador monótonamente creciente, la tabla PG_LOG puede representar el estado de la transacción como un mapa de bits. Esta tabla contiene dos bits de información de estado para cada transacción; los posibles estados son en progreso, confirmada o abortada.

PostgreSQL no deshace los cambios en las filas de la base de datos cuando una transacción se aborta, simplemente marca la transacción como abortada en PG_LOG. Una tabla de PostgreSQL por lo tanto puede contener datos de transacciones abortadas.

Se proporciona un proceso de limpieza (Vacuum cleaner) para recolectar como basura las versiones expiradas/abortadas de una fila[cite: 1]. El Limpiador también elimina las entradas del índice asociadas con las tuplas que se recolectan como basura.

Cabe señalar que en PostgreSQL, los índices no tienen información de versionado, por lo tanto, todas las versiones disponibles (tuplas) de una fila están presentes en los índices[cite: 1]. Solo observando la tupla es posible determinar si es visible para una transacción.

En PostgreSQL, una transacción no bloquea los datos cuando los lee[cite: 1]. Cada transacción ve una instantánea de la base de datos tal como existía al inicio de la transacción.

Para determinar qué versión (tupla) de una fila es visible para la transacción, se proporciona a cada transacción la siguiente información:
1.  Una lista de todas las transacciones activas/no confirmadas al inicio de la transacción actual.
2.  El ID de la transacción actual.

La visibilidad de una tupla se determina de la siguiente manera (como lo describe Bruce Momijian en [BM00]):

Las tuplas visibles deben tener un id de transacción de creación que:
*   es una transacción confirmada
*   es menor que el ID de la transacción y no estaba en proceso al inicio de la transacción, es decir, el ID no está en la lista de transacciones activas

Las tuplas visibles también deben tener un id de transacción de expiración que:
*   está en blanco o abortada o
*   es mayor que el ID de la transacción o estaba en proceso al inicio de la transacción, es decir, el ID está en la lista de transacciones activas

En palabras de Tom Lane:
Una tupla es visible si su xmin es válido y xmax no lo es. "Válido" significa "ya sea confirmada o la transacción actual".

Para evitar consultar la tabla PG_LOG repetidamente, PostgreSQL también mantiene algunos indicadores de estado en la tupla que indican si la tupla es "conocida como confirmada" o "conocida como abortada". Estos indicadores de estado son actualizados por la primera transacción que consulta la tabla PG_LOG.

## 8 Concurrencia Multiversión en Oracle
Oracle no mantiene múltiples versiones de datos en el almacenamiento permanente. En su lugar, recrea versiones más antiguas de los datos sobre la marcha a medida que se requieren.

En Oracle, un ID de transacción no es un número secuencial; en cambio, está formado por un conjunto de números que apuntan a la entrada de la transacción (ranura) en un encabezado de segmento de Reversión (Rollback). Un segmento de Reversión es un tipo especial de tabla de base de datos donde se almacenan registros "deshacer" (undo) mientras una transacción está en progreso. Múltiples transacciones pueden utilizar el mismo segmento de reversión. El bloque de encabezado del segmento de reversión se utiliza como una tabla de transacciones. Aquí se mantiene el estado de una transacción, junto con su marca de tiempo de Confirmación (llamada Número de Cambio del Sistema, o SCN por sus siglas en inglés, en Oracle).

Los segmentos de Reversión tienen la propiedad de que las nuevas transacciones pueden reutilizar el almacenamiento y las ranuras de transacciones utilizadas por transacciones más antiguas que se han confirmado o abortado. La ranura y los registros de deshacer de la transacción más antigua se reutilizan cuando no hay más espacio en el segmento de reversión para una nueva transacción. Esta facilidad de reutilización automática permite a Oracle gestionar un gran número de transacciones utilizando un conjunto finito de segmentos de reversión. Los cambios a los segmentos de Reversión se registran para que sus contenidos puedan ser recuperados en el caso de una caída del sistema.

Oracle registra el ID de Transacción que insertó o modificó una fila dentro de la página de datos. En lugar de almacenar un ID de transacción con cada fila en la página, Oracle ahorra espacio manteniendo una matriz de IDs de transacciones únicos por separado dentro de la página, y almacena solo el desplazamiento de esta matriz con la fila.

Junto con cada ID de transacción, Oracle almacena un puntero al último registro de deshacer creado por la transacción para la página. Los registros de deshacer están encadenados, de modo que Oracle puede seguir la cadena de registros de deshacer para una transacción/página, y al aplicarlos a la página, los efectos de la transacción pueden ser deshechos por completo.

No solo las filas de la tabla se almacenan de esta manera, Oracle emplea las mismas técnicas al almacenar las filas de índice.

El Número de Cambio del Sistema (SCN) se incrementa cuando una transacción se confirma.

Cuando comienza una transacción de Oracle, toma nota del SCN actual. Al leer una tabla o una página de índice, Oracle utiliza el número SCN para determinar si la página contiene los efectos de transacciones que no deberían ser visibles para la transacción actual. Solo deberían ser visibles aquellas transacciones confirmadas cuyo número SCN sea menor que el número SCN anotado por la transacción actual. Además, las Transacciones que aún no se han confirmado no deberían ser visibles. Oracle verifica el estado de confirmación de una transacción buscando el encabezado del segmento de Reversión asociado, pero, para ahorrar tiempo, la primera vez que se busca una transacción, su estado se registra en la página misma para evitar futuras búsquedas.

Si se encuentra que la página contiene los efectos de transacciones "invisibles", entonces Oracle recrea una versión más antigua de la página deshaciendo los efectos de cada una de tales transacciones. Escanea los registros de deshacer asociados con cada transacción y los aplica a la página hasta que se eliminan los efectos de esas transacciones. La nueva página creada de esta manera se utiliza luego para acceder a las tuplas dentro de ella.

Dado que Oracle aplica esta lógica tanto a los bloques de tablas como a los de índices, nunca ve tuplas que sean inválidas.

Dado que las versiones más antiguas no se almacenan en el SGBD, no hay necesidad de recolectar datos como basura.

Dado que los índices también están versionados, al escanear una relación utilizando un índice, Oracle no necesita acceder a la fila para determinar si es válida o no.

En el enfoque de Oracle, las lecturas pueden convertirse en escrituras debido a las actualizaciones del estado de una transacción dentro de la página.

Reconstruir una versión más antigua de la página es una operación costosa. Sin embargo, dado que los segmentos de Reversión son similares a las tablas ordinarias, Oracle es capaz de usar el Grupo de Búferes (Buffer Pool) para asegurar de manera efectiva que la mayor parte de los datos de deshacer se mantenga siempre en la memoria. En particular, los encabezados de los segmentos de Reversión siempre están en memoria y pueden ser accedidos directamente. Como resultado, si el Grupo de Búferes es lo suficientemente grande, Oracle puede crear versiones más antiguas de los bloques sin incurrir en mucha E/S (Entrada/Salida) de disco. Las versiones reconstruidas de una página también se almacenan en el Grupo de Búferes.

Un problema con el enfoque de Oracle es que si los segmentos de reversión no son lo suficientemente grandes, Oracle podría terminar reutilizando el espacio utilizado por las transacciones completadas/abortadas con demasiada rapidez. Esto puede significar que la información requerida para reconstruir una versión más antigua de un bloque puede no estar disponible. Las transacciones que no logran reconstruir una versión más antigua de los datos fallarán.

## Referencias

*   **[BM00]** Bruce Momijian. Aspectos internos de PostgreSQL a través de imágenes. Dic 2001.
*   **[TL01]** Tom Lane. Procesamiento de transacciones en PostgreSQL. Oct 2000.
*   **[MS87]** Michael Stonebraker. El diseño del sistema de almacenamiento Postgres. Procedimientos de la 13ª Conferencia Internacional sobre Bases de Datos Muy Grandes (Brighton, septiembre de 1987). Además, Lecturas en Sistemas de Bases de Datos, Tercera Edición, 1998. Morgan Kaufmann Publishers.
*   **[HB95]** Hal Berenson, Philip A. Bernstein, Jim Gray, Jim Melton, Elizabeth J. O'Neil, Patrick E. O'Neil: Una crítica de los niveles de aislamiento SQL ANSI. Conferencia SIGMOD 1995: 1-10.
*   **[AF04]** Alan Fekete, Elizabeth J. O'Neil, Patrick E. O'Neil: Una anomalía de transacción de solo lectura bajo el aislamiento de instantáneas. SIGMOD Record 33(3): 12-14 (2004).
*   **[JG93]** Jim Gray y Andreas Reuter. Capítulo 7: Conceptos de aislamiento. Procesamiento de transacciones: conceptos y técnicas. Morgan Kaufmann Publishers, 1993.
*   **[ZZ]** Autores desconocidos. Los métodos de acceso de Postgres. Distribución de Postgres V4.2.
*   **[DH99]** Dan Hotka. Oraclesi GIS (Cosas Internas Geeks): Aspectos internos del almacenamiento de datos físicos. Oracle Professional, septiembre de 1999.
*   **[DH00]** Dan Hotka. Oraclesi GIS (Cosas Internas Geeks): Aspectos internos del índice. Oracle Professional, noviembre de 2000.
*   **[DH01]** Dan Hotka. Oraclesi GIS (Cosas Internas Geeks): Aspectos internos del segmento de reversión. Oracle Professional, mayo de 2001.
*   **[RB99]** Roger Bamford y Kenneth Jacobs, Oracle. Patente de EE. UU. Número 5,870,758: Método y aparato para proporcionar niveles de aislamiento en un sistema de base de datos. Feb, 1999.