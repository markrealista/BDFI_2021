# Prediccion de Vuelos - BDFI 2021
Práctica de predicción de vuelos de la asignatura Big Data: Fundamentos e Infraestructura del Máster Universitario en Ingeniería de Telecomunicación de la UPM para 
el curso 2021/2022. El guión y el código de la práctica se pueden encontrar en el siguiente repositorio: https://github.com/ging/practica_big_data_2019.

**AUTORES:**
- Raúl Cabria González
- Mark Realista Quiocho

**CONSIDERACIÓN IMPORTANTE:** 
Para la correcta ejecución de la práctica se recomienda realizarla en un sistema operativo basado en Linux o en Mac OS. En caso de realizarla en Windows, algunas 
alternativas serían instalar una máquina virtual de Linux, utilizar el Subsistema de Windows para Linux disponible en Windows 10 o usar Git Bash que permite 
lanzar comandos básicos de Linux.

**NOTA:** Esta práctica se ha llevado a cabo en Windows 10 mediante la utilización de Git Bash.

## ARQUITECTURA DEL SISTEMA
A continuación, se va a realizar un breve resumen de las arquitecturas front-end y back-end del sistema de predicción de vuelos.

### ARQUITECTURA FRONT-END
El siguiente diagrama muestra la arquitectura front-end de la aplicación de predicción de retraso de vuelos. El usuario rellena un formulario con los datos de vuelo 
que se envía al servidor. En el momento en el que el usuario envía el formulario, el cliente se queda haciendo "polling" al servidor cada segundo a la espera de los resultados 
de la predicción. El servidor realiza las comprobaciones necesarias y envía un mensaje Kafka con la solicitud de predicción. Spark Stremaing se encarga de escuchar en una cola 
de Kafka las solicitudes y realizar las predicciones, las cuales se almacenan en MongoDB. Una vez que los datos están disponibles en Mongo, el cliente muestra el resultado de 
la predicción al usuario.

![image](https://user-images.githubusercontent.com/36339100/142722982-23315a49-c78b-46ec-a603-0b19079f9085.png)

### ARQUITECTURA BACK-END
El diagrama de la arquitectura back-end muestra como se entrena un modelo clasificador utilizando datos de vuelos pasados para predecir retrasos de vuelos en batch en Spark. 
Una vez que el modelo está entrenado, se guarda y, posteriomente, se lanza Zookeeper y la cola de Kafka. Spark Streaming se usa para cargar el modelo clasificador y escuchar 
las solicitudes de predicción de la cola de Kafka. Cuando llega una solicitud, Spark Streaming realiza la predicción y almacena el resultado en MongoDB, donde la aplicación 
la podrá recoger para representarla.

![image](https://user-images.githubusercontent.com/36339100/142723007-8bf81a0f-6a00-4d27-8ba4-2f1873bbbbd9.png)

## EJECUCIÓN DE LA PRÁCTICA SIN MODIFICACIONES

### Instalación de componentes
En la siguiente lista se muestran los componentes que se han utilizado para realizar la práctica:

- [JDK](https://www.oracle.com/es/java/technologies/javase/javase8-archive-downloads.html) (version 1.8)
- [Pyhton3](https://realpython.com/installing-python/) (Version 3.7) 
- [PIP](https://pip.pypa.io/en/stable/installing/)
- [SBT](https://www.scala-sbt.org/release/docs/Setup.html) 
- [MongoDB](https://docs.mongodb.com/manual/installation/) (version 4.2.17)
- [Spark](https://spark.apache.org/docs/latest/) (version 3.1.2)
- [Scala](https://www.scala-lang.org) (version 2.12)
- [Kafka y Zookeeper](https://kafka.apache.org/downloads) (version 2.12-2.8.1)

**NOTA:** La versión que se recomienda utilizar de Kafka es la 2.12-3.0.0, sin embargo, esta versión da ciertos problemas en Windows por lo que decidimos utilizar 
la versión 2.12-2.8.1.

### Descarga de datos de vuelos e instalación de librerías python
En primer lugar, clonamos el repositorio de la práctica:
```
git clone https://github.com/ging/practica_big_data_2019
```
Una vez clonado, dentro de la carpeta `practica_big_data_2019` descargamos los datos de vuelos:
```
resources/download_data.sh
```
En la misma carpeta, instalamos las librerías de python requeridas:
```
pip install -r requirements.txt
```

### Iniciar Zookeeper y Kafka
Para iniciar Zookeeper, abrimos un terminal y dentro de la carpeta de kafka descargada `kafka_2.12-2.8.1` ejecutamos lo siguiente:
```
bin/zookeeper-server-start.sh config/zookeeper.properties
```
Para iniciar Kafka, abrimos otro terminal y en la misma carpeta de kafka ejecutamos lo siguiente:
```
bin/kafka-server-start.sh config/server.properties
```
Abrimos otro terminal en la carpeta de kafka descargada y ejecutamos el siguiente comando para crear un nuevo topic:
```
bin/kafka-topics.sh \
  --create \
  --bootstrap-server localhost:9092 \
  --replication-factor 1 \
  --partitions 1 \
  --topic flight_delay_classification_request
```
Esto último debería dar como resultado:
```
Created topic "flight_delay_classification_request".
```

### Importar registros de distancia a MongoDB
Dentro de la carpeta `practica_big_data_2019`, lanzamos el siguiente comando que ejecuta el script `import_distances.sh` que importa en MongoDB 
los registros de distancias que hemos descargado previamente:
```
./resources/import_distances.sh
```
El resultado de ejecutar esto debería ser lo siguiente:
```
2021-11-19T19:24:38.814+0100    connected to: mongodb://localhost/
2021-11-19T19:24:38.966+0100    4696 document(s) imported successfully. 0 document(s) failed to import.
MongoDB shell version v4.2.17
connecting to: mongodb://127.0.0.1:27017/agile_data_science?compressors=disabled&gssapiServiceName=mongodb
Implicit session: session { "id" : UUID("caa3877b-90fd-4cad-ad8e-8ca5667b64b9") }
MongoDB server version: 4.2.17
{
        "numIndexesBefore" : 2,
        "numIndexesAfter" : 2,
        "note" : "all indexes already exist",
        "ok" : 1
}
```

### Entrenamiento del modelo con PySpark mllib
Antes que nada, es necesario crear las variables de entorno JAVA_HOME y SPARK_HOME con las rutas de las carpetas donde tengamos instalado JDK y Spark:
```
export JAVA_HOME=C:\Program Files\Java\jdk1.8.0_271
export SPARK_HOME=C:\Users\markr\apachespark\spark-3.1.2
```
A continuación, en un terminal nos situamos en la carpeta `practica_big_data_2019` y ejecutamos el script `train_spark_mllib_model.py` para entrenar 
el modelo:
```
python3 resources/train_spark_mllib_model.py .
```
Como resultado de ejecutar este comando, en el directorio `models` deberían aparecer una serie de carpetas corerspondientes a los modelos creados.

**NOTA:** es posible que en Windows el comando "python3" de error, en ese caso utilizar "python"

### Ejecución del predicctor de vuelos con spark-submit
Para ejecutar el predictor de vuelos, primero hay que cambiar el valor de la variable `base_path` que hay dentro de la clase MakePrediction.scala por la ruta de la 
carpeta donde se ha clonado el repositorio de la práctica:
```
val base_path= "C:\Users\markr\practica_big_data_2019"
```
Para ejecutar el código utilizando spark-submit, lanzamos el siguiente comando dentro de la carpeta `flight_prediction`:
```
sbt package
```
Este comando genera un archivo .jar dentro de la carpeta `target/scala-2.12` con el contenido que hay dentro de `src/main/scala`. Una vez comprobado 
que se ha creado dicho archivo .jar, ejecutamos el siguiente comando de spark-submit para arrancar el predictor de vuelos:
```
spark-submit --class es.upm.dit.ging.predictor.MakePrediction --master local --packages org.mongodb.spark:mongo-spark-connector_2.12:3.0.1,org.apache.spark:spark-sql-kafka-0-10_2.12:3.1.2 C:\Users\markr\practica_big_data_2019\flight_prediction\target\scala-2.12\flight_prediction_2.12-0.1.jar
```

### Iniciar la apliación web de predicciones
Primero declaramos la variable de entorno PROJECT_HOME con la ruta de la carpeta donde se ha clonado el repositorio:
```
export PROJECT_HOME=C:\Users\markr\practica_big_data_2019
```
Ahora, nos movemos al directorio `web` que hay dentro de la carpeta `resources` y ejecutamos el archivo `predict_flask.py`
```
cd practica_big_data_2019/resources/web
python predict_flask.py
```
Para acceder a la aplicación web de predicciones ir a http://localhost:5000/flights/delays/predict_kafka

### Comprobación de la predicciones insertadas en MongoDB
```
$ mongo
> use use agile_data_science;
>db.flight_delay_classification_response.find();
```
Debería salir algo similar a esto:
```
{ "_id" : ObjectId("6189c558ff3fa4329195c33a"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 5, "Timestamp" : ISODate("2021-11-09T00:48:22.639Z"), "FlightDate" : ISODate("2016-12-24T23:00:00Z"), "Carrier" : "AA", "UUID" : "d2f7d841-fc16-4f43-af9b-eb4f05d1be8b", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 1 }
{ "_id" : ObjectId("6189c64eff3fa4329195c33c"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 5, "Timestamp" : ISODate("2021-11-09T00:52:29.429Z"), "FlightDate" : ISODate("2016-12-24T23:00:00Z"), "Carrier" : "AA", "UUID" : "70f22f3b-325d-425a-baa7-7d3a7671d9f9", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 1 }
{ "_id" : ObjectId("6189c759ff3fa4329195c33e"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 5, "Timestamp" : ISODate("2021-11-09T00:56:56.386Z"), "FlightDate" : ISODate("2016-12-24T23:00:00Z"), "Carrier" : "AA", "UUID" : "3b4aae24-d8cb-4f2f-b9d1-02c8fe13a1d8", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 1 }
{ "_id" : ObjectId("618c12f29ee1ea1ddb1c6058"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 5, "Timestamp" : ISODate("2021-11-10T18:43:59.839Z"), "FlightDate" : ISODate("2016-12-24T23:00:00Z"), "Carrier" : "AA", "UUID" : "ea0925ac-6fb2-41d0-adac-d66f7b35d07f", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 2 }
{ "_id" : ObjectId("618c226d2df0257029674aa8"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 5, "Timestamp" : ISODate("2021-11-10T19:50:03.013Z"), "FlightDate" : ISODate("2016-12-24T23:00:00Z"), "Carrier" : "AA", "UUID" : "3a4d5bd2-1371-4f19-a720-68e5d25bc782", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 2 }
{ "_id" : ObjectId("619540e603a10c7f23223440"), "Origin" : "ATL", "DayOfWeek" : 6, "DayOfYear" : 360, "DayOfMonth" : 25, "Dest" : "SFO", "DepDelay" : 5, "Timestamp" : ISODate("2021-11-17T17:50:24.858Z"), "FlightDate" : ISODate("2016-12-24T23:00:00Z"), "Carrier" : "AA", "UUID" : "828bbbea-3947-4dbe-a89c-dea02ee7a9b1", "Distance" : 2139, "Route" : "ATL-SFO", "Prediction" : 2 }
```
