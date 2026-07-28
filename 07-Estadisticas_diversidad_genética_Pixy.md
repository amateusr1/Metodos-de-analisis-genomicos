# 07. Estadísticas de diversidad genética con Pixy

## Introducción

Una vez obtenido un conjunto de variantes de alta calidad mediante el proceso de alineamiento, llamado y filtrado de variantes, el siguiente paso consiste en cuantificar la diversidad genética presente dentro y entre las especies o poblaciones de interes. Estas estimaciones permiten describir la variabilidad genética, evaluar procesos evolutivos y demograficos, determinar el flujo génico e identificar regiones del genoma con patrones inusuales de variación.

En este curso se utilizó **Pixy**, una herramienta desarrollada para calcular estadísticas clásicas de genética de poblaciones a partir de archivos VCF que contienen tanto sitios variantes como invariantes (*all-sites VCF*). A diferencia de otros programas, Pixy produce estimaciones insesgadas incluso en presencia de datos faltantes (*missing data*), evitando la sobreestimación de parámetros como la diversidad nucleotídica (π).

Durante este ejercicio se calcularán tres de las estadísticas más utilizadas en genética de poblaciones:

- **Diversidad nucleotídica (π)**, que mide la variación genética dentro de una población.
- **Divergencia absoluta (dXY)**, que cuantifica las diferencias promedio entre dos poblaciones.
- **Índice de diferenciación genética (FST)**, que evalúa el grado de diferenciación genética entre poblaciones.

Los análisis se realizarán mediante ventanas deslizantes a lo largo del genoma, permitiendo visualizar cómo varían estos parámetros entre diferentes regiones genómicas.

---

## Objetivos

Al finalizar este capítulo el estudiante será capaz de:

- Comprender la importancia de utilizar sitios invariantes para estimar correctamente la diversidad genética.
- Preparar los archivos de entrada requeridos por Pixy.
- Definir poblaciones mediante un archivo de asignación de muestras.
- Calcular diversidad nucleotídica (π), divergencia genética (dXY) y diferenciación genética (FST).
- Interpretar los resultados obtenidos a lo largo del genoma.

---

# ¿Por qué utilizar Pixy?

Muchas herramientas calculan estadísticas de diversidad únicamente utilizando los SNPs presentes en un archivo VCF. Sin embargo, este enfoque introduce un sesgo importante, ya que ignora todas las posiciones del genoma donde no existe variación.

Por ejemplo, si un fragmento genómico posee 10 000 posiciones y solamente 120 corresponden a SNPs, calcular la diversidad utilizando únicamente esas 120 posiciones produce estimaciones artificialmente elevadas.

Pixy evita este problema utilizando archivos VCF que contienen **todos los sitios del genoma**, incluyendo tanto posiciones variables como invariantes, permitiendo obtener estimaciones más precisas de la diversidad genética.

---


