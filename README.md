# BDFI_2021

Práctica de la asignatura Big Data Fundamentos y Arquitectura del Máster Universitario de Ingeniría de Telecomunicación (MUIT) de la UPM. Práctica basada en en el siguiente repositorio: https://github.com/ging/practica_big_data_2019

**AUTORES:**

- Raúl Cabria González

- Mark Realista Quiocho

## PRÁCTICA SIN MODIFICACIONES

### Instalación

En la siguiente lista se muestran las herramientas utilizadas para la realización de la práctica con sus respectivas versiones:

- [Intellij](https://www.jetbrains.com/help/idea/installation-guide.html) (jdk_1.8)
- [Pyhton3](https://realpython.com/installing-python/) (Version 3.7) 
- [PIP](https://pip.pypa.io/en/stable/installing/)
- [SBT](https://www.scala-sbt.org/release/docs/Setup.html) 
- [MongoDB](https://docs.mongodb.com/manual/installation/) (version 4.2)
- [Spark](https://spark.apache.org/docs/latest/) (Mandatory version 3.1.2)
- [Scala](https://www.scala-lang.org)(version 2.12)
- [Zookeeper](https://zookeeper.apache.org/releases.html)
- [Kafka](https://kafka.apache.org/quickstart) (version kafka_2.12-2.8.1)

### Descarga de datos e instalación de librerías python

En primer lugar, clonar el repositorio de la práctica y entramos dentro de la carpeta practica_big_data_2019
```
git clone https://github.com/ging/practica_big_data_2019
cd practica_big_data_2019
```
Descargamos lo datos:
```
resources/download_data.sh
```
Instalamos la librearía de python necesarias:
```
pip install -r requirements.txt
```

### Zookeeper y Kafka
Nos situamos dentro de la carpeta de kafka descargada y ejecutamos los siguientes comandos:
#### Iniciar Zookeeper
```
bin/zookeeper-server-start.sh config/zookeeper.properties
```
#### Iniciar Kafka
```
bin/kafka-server-start.sh config/server.properties
```
Abrimos otro terminal y en la misma carpeta de kafaka ejecutamos el siguiente comando para crear un topic nuevo:
```
    bin/kafka-topics.sh \
      --create \
      --bootstrap-server localhost:9092 \
      --replication-factor 1 \
      --partitions 1 \
      --topic flight_delay_classification_request
```
Resultado:
```
Created topic "flight_delay_classification_request".
```
Para ver la lista de topics creados:
```
bin/kafka-topics.sh --bootstrap-server localhost:9092 --list
```
Resultado:
```
flight_delay_classification_request
```

### MongoDB
Dentro de la carpeta practica_big_data_2019, ejecutamos el siguiente comando para importar los registro de distancia en Mongo
```
./resources/import_distances.sh
```
Resultado:
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

### Entrenar y guardar el modelo con PySpark mllib

En primer lugar es necesario crear las variables de entorno JAVA_HOME y SPARK_HOME:
cd practica_big_data_2019
export JAVA_HOME=/usr/lib/jvm/java-1.8.0-openjdk-amd64
export SPARK_HOME=/opt/spark

Ejecutamos a continuación el script train_saprk_mllib_model.py que se encarga de entrenar el modelo:
```
python3 resources/train_spark_mllib_model.py .
```
Como resultado de este comando, en la carpeta models deberían aparecer los modelos

### Ejecutar Flight Predictor con Spark-Submit

En primer lugar, es neceario cambiar el valor que tiene base_paht dentro de el archivo MakePrediction.scala por la ruta donde se ha clonado el repositorio de la práctica.
```
val base_path= "/home/user/Desktop/practica_big_data_2019"
```
Para ejecutar el código utilizando Spark-Submit ejecutamos los siguientes comandos dentro de la carpeta flight_prediction:
```
sbt clean
```
```
sbt package
```
Este último comando lo que hace es generar el archivo .jar necesario para la ejecución del código dentro de la carpeta flight_prediction\target\scala-2.12. Una vez obtenido dicho fichero .jar, ejecutamos el siguiente comando que para ejectuar el predictor de vuelos:
```
spark-submit --packages org.mongodb.spark:mongo-spark-connector_2.12:3.0.1,org.apache.spark:spark-sql-kafka-0-10_2.12:3.1.2 C:\Users\markr\practica_big_data_2019\flight_prediction\target\scala-2.12\flight_prediction_2.12-0.1
```

### Arrancar la aplicación web de predicciones

Primero declaramos la variable de entorno PROJECT_HOME con la ruta donde se ha clonado el repositorio:
```
export PROJECT_HOME=/home/user/Desktop/practica_big_data_2019
```
Ahora, nos movemos a la carpeta resoruces/web y ejecutamos el archivo predict_flask.py:
```
cd practica_big_data_2019/resources/web
python3 predict_flask.py
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
