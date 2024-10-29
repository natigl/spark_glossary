# Guía Completa de Apache Spark 🚀
## 1. Configuración e Inicialización
### Creación de SparkSession

scala

// Configuración básica


import org.apache.spark.sql.SparkSession

val spark = SparkSession
  .builder()
  .appName("MiAplicacion")
  .master("local[*]")   # local[*] usa todos los cores disponibles
  .config("spark.executor.memory", "4g")
  .config("spark.driver.memory", "2g")
  .config("spark.sql.shuffle.partitions", "200")
  .enableHiveSupport()  # Si necesitas soporte de Hive
  .getOrCreate()


// Obtener SparkContext desde SparkSession


val sc = spark.sparkContext
