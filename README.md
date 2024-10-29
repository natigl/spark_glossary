# Guía Completa de Apache Spark
## 1. Configuración e Inicialización
Creación de SparkSession
scalaCopy// Configuración básica
import org.apache.spark.sql.SparkSession

val spark = SparkSession
  .builder()
  .appName("MiAplicacion")
  .master("local[*]")  // local[*] usa todos los cores disponibles
  .config("spark.executor.memory", "4g")
  .config("spark.driver.memory", "2g")
  .config("spark.sql.shuffle.partitions", "200")
  .enableHiveSupport()  // Si necesitas soporte de Hive
  .getOrCreate()

// Obtener SparkContext desde SparkSession
val sc = spark.sparkContext
Configuración de Logs
scalaCopy// Configurar nivel de log
import org.apache.log4j.{Level, Logger}
Logger.getRootLogger.setLevel(Level.ERROR)
2. Creación de Estructuras de Datos Básicas
RDDs
scalaCopy// Desde colección
val rdd1 = sc.parallelize(1 to 1000, numSlices = 10)

// Desde archivo
val rdd2 = sc.textFile("path/to/file.txt")

// Desde múltiples archivos
val rdd3 = sc.wholeTextFiles("path/to/directory/*")

// Crear RDD vacío
val rdd4 = sc.emptyRDD[String]

// Desde otra fuente de datos
val rdd5 = sc.hadoopFile[K, V, F]("path")
DataFrames
scalaCopy// Desde archivo
val df1 = spark.read
  .format("csv")
  .option("header", "true")
  .option("inferSchema", "true")
  .load("path/to/file.csv")

// Desde RDD
val df2 = spark.createDataFrame(rdd, schema)

// Desde datos en memoria
val df3 = spark.createDataFrame(Seq(
  (1, "uno"), 
  (2, "dos")
)).toDF("numero", "texto")

// Desde tabla Hive
val df4 = spark.table("nombre_tabla")
Datasets
scalaCopycase class Persona(nombre: String, edad: Int)

// Desde DataFrame
val ds1 = df.as[Persona]

// Desde secuencia
val ds2 = spark.createDataset(
  Seq(Persona("Juan", 25))
)
3. Operaciones de Transformación
Transformaciones Comunes para Todas las Estructuras
scalaCopy// Filtrado
df.filter($"edad" > 25)
ds.filter(_.edad > 25)
rdd.filter(_ > 25)

// Mapeo
df.select($"nombre", $"edad" * 2)
ds.map(p => p.edad * 2)
rdd.map(_ * 2)

// Ordenación
df.orderBy($"edad".desc)
ds.orderBy($"edad".desc)
rdd.sortBy(x => x, ascending = false)
Operaciones Window
scalaCopyimport org.apache.spark.sql.expressions.Window

val windowSpec = Window
  .partitionBy("departamento")
  .orderBy("salario")
  .rowsBetween(Window.unboundedPreceding, Window.currentRow)

val dfWindow = df.withColumn("acumulado", 
  sum("salario").over(windowSpec))
Joins
scalaCopy// DataFrame/Dataset Joins
df1.join(df2, 
  df1("id") === df2("id"), 
  "inner")  // Otros tipos: left, right, outer, cross

// RDD Joins
rdd1.join(rdd2)  // Para RDDs de tipo (K,V)
4. Operaciones de Agregación
Agregaciones Básicas
scalaCopy// DataFrame
df.groupBy("departamento")
  .agg(
    avg("salario").alias("salario_promedio"),
    count("*").alias("empleados"),
    sum("ventas").alias("ventas_totales")
  )

// Dataset
ds.groupByKey(_.departamento)
  .agg(
    avg($"salario").as[Double],
    count("*").as[Long]
  )

// RDD
rdd.aggregateByKey(0)(
  (acc, value) => acc + value,
  (acc1, acc2) => acc1 + acc2
)
Funciones de Agregación Avanzadas
scalaCopyimport org.apache.spark.sql.functions._

// Percentiles
df.select(
  percentile_approx($"valor", 0.5).alias("mediana"),
  percentile_approx($"valor", array(0.25, 0.5, 0.75)).alias("cuartiles")
)

// Funciones estadísticas
df.select(
  variance("valor"),
  stddev("valor"),
  skewness("valor"),
  kurtosis("valor")
)
5. Optimización y Performance
Particionamiento
scalaCopy// Reparticionamiento
df.repartition(10)
df.repartition(col("fecha"))
df.coalesce(5)  // Reduce particiones sin shuffle completo

// Particionamiento personalizado
val customPartitioner = new HashPartitioner(10)
rdd.partitionBy(customPartitioner)
Persistencia y Caching
scalaCopy// Niveles de persistencia
import org.apache.spark.storage.StorageLevel

df.persist(StorageLevel.MEMORY_AND_DISK_SER)
ds.cache()  // Equivalente a MEMORY_AND_DISK
rdd.persist(StorageLevel.DISK_ONLY)

// Unpersist cuando ya no se necesita
df.unpersist()
Checkpointing
scalaCopy// Configurar directorio de checkpoint
sc.setCheckpointDir("path/to/checkpoint")

// Realizar checkpoint
df.checkpoint()
rdd.checkpoint()
6. Funciones Personalizadas
UDFs (User Defined Functions)
scalaCopy// UDF simple
val mayusculasUDF = udf((s: String) => s.toUpperCase)
df.withColumn("MAYUSCULAS", mayusculasUDF($"texto"))

// UDF con múltiples parámetros
val combinarUDF = udf((s1: String, s2: String) => s"$s1 - $s2")
df.withColumn("combinado", combinarUDF($"col1", $"col2"))

// UDAF (User Defined Aggregation Function)
import org.apache.spark.sql.expressions.UserDefinedAggregateFunction
// Implementación de UDAF...
7. Operaciones de E/S
Lectura de Datos
scalaCopy// Opciones comunes de lectura
val df = spark.read
  .format("parquet")  // o csv, json, orc, etc.
  .option("header", "true")
  .option("inferSchema", "true")
  .option("delimiter", ",")
  .option("encoding", "UTF-8")
  .option("dateFormat", "yyyy-MM-dd")
  .load("path/to/file")

// Lectura desde base de datos
val dfDB = spark.read
  .format("jdbc")
  .option("url", "jdbc:postgresql://localhost:5432/db")
  .option("dbtable", "schema.table")
  .option("user", "username")
  .option("password", "password")
  .load()
Escritura de Datos
scalaCopy// Opciones comunes de escritura
df.write
  .format("parquet")
  .mode("overwrite")  // append, ignore, error
  .partitionBy("fecha")
  .bucketBy(4, "id")
  .sortBy("nombre")
  .save("path/output")

// Escritura a base de datos
df.write
  .format("jdbc")
  .mode("append")
  .option("url", "jdbc:postgresql://localhost:5432/db")
  .option("dbtable", "schema.table")
  .option("user", "username")
  .option("password", "password")
  .save()
8. Monitorización y Debugging
Planes de Ejecución
scalaCopy// Ver plan lógico
df.explain()

// Ver plan físico
df.explain(true)

// Obtener plan como string
df.queryExecution.explainString()
Métricas y Estadísticas
scalaCopy// Estadísticas de DataFrame
df.describe().show()

// Información de particionamiento
df.rdd.getNumPartitions

// Contar registros por partición
df.mapPartitions(iter => Iterator(iter.size)).collect()
9. Integración con Streaming
Structured Streaming
scalaCopy// Lectura de stream
val streamingDF = spark.readStream
  .format("kafka")
  .option("kafka.bootstrap.servers", "host:port")
  .option("subscribe", "topic")
  .load()

// Procesamiento y escritura
val query = streamingDF
  .writeStream
  .outputMode("append")
  .format("parquet")
  .start()
10. Funciones de Ventana Temporales
Time Windows
scalaCopyimport org.apache.spark.sql.functions._

// Ventana deslizante
df.groupBy(
  window($"timestamp", "1 hour", "15 minutes"),
  $"usuario"
)

// Ventana de sesión
df.groupBy(
  session_window($"timestamp", "30 minutes"),
  $"usuario"
)
