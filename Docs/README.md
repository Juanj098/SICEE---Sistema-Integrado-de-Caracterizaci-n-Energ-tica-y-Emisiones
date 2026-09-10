# SICEE - Sistema INtegrado de Caracterizacion Energetica y Emisiones

## Analisis del negocio

**Contexto del negocio**

El centro de Investigacion de Ingenieria (CII) de la facultad de Ingenieria de la Universidad de San Carlos de Guatemala (USAC) es el centro de servicios de investigacion y asesoria tecnica de la facultad. Su función principal es proveer servicios de alta calidad cientifico-tecnologica a los distintos sectores de la sociedad guatemalteca, actuando como puente entre la academia y la insdustria a través de ensayos, analisis y asesoria especializada.
 El CII organiza sus servicios en secciones especializadas, cada una enfocada en un area especifica de la ingenieria y atendida por profesionales con experiencia técnica en la materia: 

* Agregados, Concretos y Morteros
* Mecánica de Suelos y Asfaltos
* Metales y Productos Manufacturados
* Tecnología de la Madera
* Química Industrial
* Topografía y Catastro
* Centro de Información a la Construcción
* Ecomateriales
* Gestión de la Calidad
* Gestión Ambiental
* Estructuras
* Metrología
* Tecnología de los Materiales y Sistemas

Entre su cartera de clientes se encuentran empresas reconocidas del sector industria y de construcción guatemalteco, como Aceros de Guatemala, Cemsur, Grupo ITM y Conasa, entre otras.

**Descripcion del problema**

Actualmente, este tipo de ensayos —eficiencia térmica y emisiones— se gestionan de manera manual, principalmente mediante hojas de cálculo (Excel), lo que genera limitaciones en cuanto a trazabilidad, estandarización de resultados, generación de reportes y explotación posterior de los datos recolectados. Esto representa una oportunidad para el desarrollo de una plataforma que digitalice y sistematice dicho flujo de trabajo, mejorando la calidad, consistencia y disponibilidad de la información generada por el área de Tecnología de la Madera.

## Drivers de calidad
---

|ID        | Categoria      | SubCategoria                     | Escenario | Prioridad |
|----------|----------------|----------------------|-----------|-----------|
|**EAC-01**| <p align="center"> Usabilidad </p>    | Aprendizaje | **Fuente de estimulos:** Investigador <br> **Estímulo:** Necesita generar una prueba de ebullicion por primera vez <br> **Artefacto:** Modulo de ingreso de datos <br> **Ambiente:** Primera sesion de uso, sin capacitacion previa. <br> **Respuesta:** El investigador completa el registro guiándose por etiquetas claras y validación en tiempo real, sin necesidad de asistencia externa <br> **Medida de la respuesta:** Completa el ingreso en menos de 1 minuto y sin errores de validación en el primer intento | <p align ="center"> Alta </p> |
|**EAC-02**| <p align="center"> Eficiencia <p> | Comportamiento <br> en el tiempo | **Fuente de estimulos:** Jefe de sección <br> **Estímulo:** Revisar resultados de pruebas anteriores  <br> **Artefacto:** Módulo de consulta de historial de pruebas <br> **Ambiente:** Operacion normal del sistema <br> **Respuesta:** Mostrar el listado de pruebas anteriores, Hasta 500 registros almacenados. Ordenados por fecha de realizacion. <br> **Medida de la respuesta:** En menos de 3 segundos. | <p align="center"> Alta </p>  |
|**EAC-03**| <p align="center"> Mantenibilidad </p> | Facilidad de análisis            | **Fuente de estimulos:** Equipo de Desarrollo <br> **Estímulo:** Un componente del sistema presenta un bug <br> **Artefacto:** Código fuente del endpoint afectado <br> **Ambiente:** Sistema en produccion, con documentacion tecnica <br> **Respuesta:** El desarrollador identifica el módulo y la causa raíz del problema mediante logs, nomenclatura clara del código y documentación técnica, sin necesidad de revisar módulos no relacionados <br> **Medida de la respuesta:** Se localiza el problema en un tiempo no mayor a 30 minutos | <p align="center"> Alta </p> |
|**EAC-04**| <p align="center"> Fiabilidad </p> | Tolerancia a fallos | **Fuente de estimulos:** Investigador <br> **Estímulo:** Se solicita la generacion de un reporte pero ocurre una interrupcion a mitad del proceso.  <br> **Artefacto:** Modulo de creación de reportes. <br> **Ambiente:** Operacion normal del sistema <br> **Respuesta:** El sistema detecta interrupciones en el servidor y no genera reporte incompleto o corrupto, da la opcion de intentar volver a generarlo. <br> **Medida de la respuesta:** El sistema se recupera y da la opcion de reintentar en menos de 30 segundos. |   Alta |
|**EAC-05**| <p align="center"> Eficiencia </p> | Comportamiendo <br> de recursos  | **Fuente de estimulos:** Multiples Investigadores <br> **Estímulo:** Uso simultaneo del sistema en horario laboral <br> **Artefacto:** Servidor de aplicacion <br> **Ambiente:** Operacion normal del sistema <br> **Respuesta:** El sistema atiende varias solicitudes de forma concurrente sin degradar el rendimiento <br> **Medida de la respuesta:**  Uso de cpu y memoria se mantienen por debajo del 80% con hasta 10 usuarios concurrentes| Media |
|**EAC-06**| <p align="center"> Funcionalidad </p> | Cumplimiento |**Fuente de estimulos:** Investigador <br> **Estímulo:** Solicita la generacion de un reporte de la prueba WBT. <br> **Artefacto:** Modulo de creacion de reportes <br> **Ambiente:** Generacion de reportes sobre pruebas realizadas<br> **Respuesta:** El sistema genera el reporte respetando el formato y los campos exigidos por el estándar de la prueba WBT. <br> **Medida de la respuesta:** reporte se genera en menos de 10 minutos, cumpliendo el 100% de los campos requeridos por el formato| Media |


## Requerimientos Funcionales y No funcionales
---

**RF**


**RNF**

  