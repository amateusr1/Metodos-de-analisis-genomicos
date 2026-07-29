# Estadísticas de Diversidad Genética: Fundamentos y Herramientas de Cálculo

> **Autor:** Andrey Mateus-Ruiz  
> **Curso:** Métodos de Análisis Genómicos

> **Profesor:** Ph.D. Gustavo Silva-Arias  
> **Universidad Nacional de Colombia**

> **Nota**
>
> Este documento se encuentra en constante construcción y actualización. Su objetivo es servir como una bitácora técnica y un recurso de consulta donde se documentan los comandos, scripts, metodologías y resultados desarrollados durante el curso. A medida que continúe profundizando en los temas y adquiriendo nueva experiencia, el repositorio seguirá incorporando nuevos contenidos, ejemplos y mejoras.

---

La diversidad genética es la materia prima sobre la que actúa la selección natural, la deriva génica y la adaptación. Cuantificarla no es un simple ejercicio descriptivo: permite inferir la historia demográfica de una especie, flujo génico entre poblaciones y señales de selección. En estudios de evolución, biogeografía o genética de poblaciones estas métricas son la base para contrastar hipótesis sobre rutas de dispersión, introgresión y diferenciación adaptativa local, etc.

Este documento busca introducir a las principales estadísticas de diversidad genética, la importancia de calcularlas, los programas recomendados según el tamaño y tipo de datos: DnaSP y Arlequin para conjuntos de datos pequeños (secuencias Sanger, pocos loci, microsatélites), y VCFtools/pixy junto con paquetes de R para datos masivos derivados de secuenciación de nueva generación (WGS, RADseq, etc.), y más importante aún como interpretarlas. 






# VCFTOOLS Y R TAMBIEN INCLUIR

07. Estadísticas de diversidad genética con Pixy

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


