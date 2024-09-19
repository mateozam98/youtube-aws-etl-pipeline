# Pipeline de ingesta de datos de YouTube con AWS Lambda y S3 para análisis de datos en AWS Athena

## Descripción

El proyecto **# youtube-aws-etl-pipeline** es una solución automatizada para la ingesta y procesamiento diario de datos desde la API de YouTube hacia AWS. Utiliza AWS Lambda para ejecutar un script en Python que extrae datos de YouTube, los formatea y los almacena como archivos CSV en Amazon S3. Luego, AWS Glue realiza el procesamiento ETL (Extracción, Transformación y Carga) para preparar los datos, los cuales se pueden consultar mediante AWS Athena para análisis más profundos. Todo el flujo está orquestado por AWS EventBridge, que programa la ejecución diaria del pipeline.

## Arquitectura del Proyecto

El pipeline está compuesto por los siguientes componentes:

1. **API de YouTube**: Se extraen datos relevantes, como estadísticas de videos y canales.
2. **AWS Lambda**: Función serverless que ejecuta un script en Python para realizar la ingesta de datos desde la API de YouTube y almacenarlos en formato CSV.
3. **Amazon S3**: Almacén de los archivos CSV que contienen los datos de YouTube ingeridos diariamente.
4. **AWS Glue**: Servicio de procesamiento ETL que limpia y transforma los datos almacenados en S3, preparándolos para análisis.
5. **AWS Athena**: Herramienta que permite realizar consultas SQL directamente sobre los datos procesados por AWS Glue en S3.
6. **AWS EventBridge**: Servicio que programa la ejecución diaria de la función Lambda para garantizar la ingesta automática de datos cada 24 horas.

## Diagrama:

![image](https://github.com/user-attachments/assets/9703703e-beec-47ae-8004-bf9e69cfc5b1)

## Características

- **Automatización Completa**: Gracias a EventBridge, el pipeline se ejecuta automáticamente cada día sin intervención manual.
- **Procesamiento ETL**: AWS Glue transforma los datos crudos en S3, facilitando su consulta y análisis.
- **Consultas SQL con Athena**: Los datos procesados se pueden consultar fácilmente usando SQL sin necesidad de moverlos fuera de S3.
- **Escalabilidad**: El uso de servicios serverless como Lambda y Glue permite escalar la solución de manera automática según el volumen de datos.

## Instalación y Configuración

1. Clona el repositorio:
   ```bash
   git clone https://github.com/mateozam98/youtube-aws-etl-pipeline.git
   cd youtube-aws-etl-pipeline
2. Despliega la función Lambda:

- Sube el código del script en Python a AWS Lambda.
- Configura las variables de entorno en Lambda, incluyendo la YouTube API Key y el nombre del bucket de S3.
- Asigna un rol IAM a la función Lambda con permisos adecuados para interactuar con S3, Glue y EventBridge.
  
3. Crea un Data Catalog en AWS Glue:

- Accede a la consola de AWS Glue y crea un nuevo Data Catalog.
- Configura un Crawler para el bucket de S3 donde se almacenan los archivos CSV y define una base de datos en el Data Catalog.

4. Configura un Data Source en AWS Athena:

- Accede a la consola de AWS Athena y crea una nueva fuente de datos usando el Data Catalog creado en AWS Glue.
- Asegúrate de que el catálogo de datos esté correctamente configurado para realizar consultas SQL sobre los datos almacenados en S3.
  
5. Configura AWS EventBridge:
  
- Crea una regla en EventBridge para ejecutar la función Lambda cada 24 horas.

## Resultados

- Función Lambda en AWS:
  
![image](https://github.com/user-attachments/assets/85fa3559-ee39-4be5-bb1c-b8f166d19846)

- Consulta en AWS Athena:

![image](https://github.com/user-attachments/assets/7dbd5b4a-bc12-40a6-a2d5-e00275b6256a)

 
## Uso

Una vez configurado, el pipeline funcionará de la siguiente manera:

- Cada día, AWS Lambda extrae datos de la API de YouTube y los almacena en S3 en formato CSV.
- AWS Glue procesa los datos para transformarlos y cargarlos en el catálogo de datos.
- Los datos pueden ser consultados usando AWS Athena para realizar análisis de los datos de YouTube mediante SQL.

Ejemplo de Consulta en Athena
(

    WITH STATS AS (
            SELECT channel_name
                   ,subscribers
                   ,lag(subscribers,1) over (order by createt_at) as prev_susc
                   ,video_count
                   ,lag(video_count,1) over (order by createt_at) as prev_videoc
                   ,total_views
                   ,lag(total_views,1) over (order by createt_at) as prev_viewsc
                   ,createt_at
            FROM "channel_stats"."youtube_stats_demo" 
            WHERE channel_name = 'Fazt_web'
            order by createt_at desc )
            
    SELECT channel_name
           ,subscribers - prev_susc as qty_susc
           ,((cast(subscribers as decimal(16,5)) / cast(prev_susc as decimal(16,5))) -1 ) *100 as grow_susc
           ,video_count - prev_videoc as qty_videos
           ,((cast(video_count as decimal(16,5)) / cast(prev_videoc as decimal(16,5))) -1 ) *100 as grow_videos
           ,total_views - prev_viewsc as qty_views
           ,((cast(total_views as decimal(16,5)) / cast(prev_viewsc as decimal(16,5))) -1 ) *100 as grow_views
           ,createt_at
    FROM STATS
    ;
Este ejemplo compara las estadísticas actuales de un canal de YouTube con las del período anterior y calcula la variación en números absolutos y en porcentaje para suscriptores, número de videos y vistas totales.24.

## Contribuciones
¡Las contribuciones son bienvenidas! Si tienes alguna idea para mejorar el proyecto, no dudes en hacer un fork del repositorio, crear una nueva rama, y enviar un pull request. Toda mejora es apreciada.
