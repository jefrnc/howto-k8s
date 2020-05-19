[![ko-fi](https://www.ko-fi.com/img/githubbutton_sm.svg)](https://ko-fi.com/josephefranco)

# Introduccion a ConfigMaps

Resumen

- [x] Algunas cosas más
- [ ] Revision de esta guia (esto se escribio de forma rapida, mientras aprendia la plataforma con lo cual pueden existir correcciones o mejoras. No dudes de hacerme un PR para ir mejorando el contenido)


## ¿Qué es un ConfigMaps?
ConfigMaps vincula archivos de configuración, argumentos de la línea de comandos, variables de entorno, números de puerto y otros artefactos de configuración a los contenedores y componentes del sistema de tus pods en entorno de ejecución. ConfigMaps te permite separar tus ajustes de tus pods y componentes, lo que ayuda a mantener tus cargas de trabajo portátiles, hace que sus ajustes sean más fáciles de cambiar y administrar, y evita codificar los datos de configuración según las especificaciones del pod.

Los ConfigMaps son útiles para almacenar y compartir información de configuración no sensible y sin cifrar. Para usar información sensible en tus clústeres, debes usar secretos.


## Cómo crear un ConfigMap

* Una manera puede ser desde un archivo
* Es escribir un objecto de configmap en vez de usarlo como un archivo


Crea un ConfigMap con el comando siguiente:

kubectl create configmap [NAME] [DATA]

[DATA] puede ser:

una ruta a un directorio que contiene uno o más archivos de configuración, indicada con la marca --from-file
pares clave-valor, cada uno especificado con marcas --from-literal
Para obtener más información sobre kubectl create, consulta la documentación de referencia.

También puedes crear un ConfigMap si defines un objeto ConfigMap en un archivo de manifiesto YAML y, luego, implementas el objeto con kubectl create -f [FILE].


 