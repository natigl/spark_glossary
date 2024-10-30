# Guía Completa de Apache Spark 🚀

[![Spark Version](https://img.shields.io/badge/spark-3.5.0-orange.svg)](https://spark.apache.org/)
[![Scala Version](https://img.shields.io/badge/scala-2.12-red.svg)](https://www.scala-lang.org/)
[![License](https://img.shields.io/badge/license-MIT-blue.svg)](LICENSE)

**Una guía completa de referencia para Apache Spark**, incluyendo configuración, operaciones básicas y avanzadas, optimización y mejores prácticas.

## 📚 Tabla de Contenidos

* [Configuración e Inicialización](#configuración-e-inicialización)
* [Estructuras de Datos Básicas](#estructuras-de-datos-básicas)
* [Operaciones de Transformación](#operaciones-de-transformación)
* [Operaciones de Agregación](#operaciones-de-agregación)
* [Optimización y Performance](#optimización-y-performance)
* [Funciones Personalizadas](#funciones-personalizadas)
* [Operaciones de E/S](#operaciones-de-es)
* [Monitorización y Debugging](#monitorización-y-debugging)
* [Integración con Streaming](#integración-con-streaming)
* [Funciones de Ventana Temporales](#funciones-de-ventana-temporales)

## ⚡ Requisitos Previos

- Apache Spark 3.5.0
- Scala 2.12
- Java 8 o superior
- Maven o SBT para gestión de dependencias

## 🛠️ Instalación

```bash
# Añadir dependencia en build.sbt
libraryDependencies += "org.apache.spark" %% "spark-sql" % "3.5.0"
