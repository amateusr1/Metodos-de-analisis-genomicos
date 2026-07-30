# 🐧 Introducción a Linux

> **Autor:** Andrey Mateus-Ruiz  
> **Curso:** Métodos de Análisis Genómicos
> **Profesor:** Ph.D. Gustavo Silva-Arias  
> **Universidad Nacional de Colombia**

---

# Descripción

Linux es el sistema operativo predominante en bioinformática y computación de alto rendimiento (High Performance Computing, HPC). La gran mayoría de herramientas utilizadas para el análisis de datos de secuenciación masiva, ensamblaje de genomas, alineamiento de lecturas y análisis de variantes se ejecutan desde una terminal mediante comandos.

Este documento reúne los comandos más utilizados durante el curso **Métodos de Análisis de Datos Genómicos** y pretende servir como una guía práctica para estudiantes que comienzan a trabajar en ambientes Linux y clústeres HPC.

Más que memorizar comandos, el objetivo es comprender la lógica de trabajo desde la línea de comandos y desarrollar flujos de análisis reproducibles.

> **Nota**
>
> Este documento se encuentra en constante construcción y actualización. La idea de este repositorio es funcionar como una bitácora técnica del curso, por lo que se irán incorporando nuevos comandos, herramientas y ejemplos conforme avancen los diferentes módulos.

---

# Objetivos

Al finalizar este módulo podrás:

- Conectarte a un servidor remoto mediante SSH.
- Navegar por el sistema de archivos de Linux.
- Crear, copiar, mover y eliminar archivos y directorios.
- Visualizar y editar archivos de texto.
- Administrar permisos.
- Trabajar con ambientes Conda.
- Utilizar módulos de software.
- Ejecutar y monitorear trabajos mediante SLURM.
- Manipular archivos FASTQ.
- Administrar espacio de almacenamiento.

---

# Conectarse al servidor

La conexión a un clúster HPC generalmente se realiza mediante **SSH (Secure Shell)**.

```bash
ssh usuario@168.176.xxx.xxx
```

o mediante un alias previamente configurado.

```bash
ssh perseus
```

Una vez conectado, verifica dónde te encuentras.

```bash
pwd
```

---

# Navegación entre directorios

Mostrar el directorio actual

```bash
pwd
```

Listar archivos

```bash
ls
```

Lista detallada

```bash
ls -lh
```

Lista ordenada por fecha

```bash
ls -lht
```

Cambiar de directorio

```bash
cd carpeta
```

Subir un nivel

```bash
cd ..
```

Ir al directorio personal

```bash
cd ~
```

Volver al directorio anterior

```bash
cd -
```

---

# Crear archivos y directorios

Crear un directorio

```bash
mkdir proyecto
```

Crear varios directorios

```bash
mkdir datos scripts resultados
```

Crear un archivo vacío

```bash
touch archivo.txt
```

---

# Visualización de archivos

Mostrar el contenido

```bash
cat archivo.txt
```

Leer archivos largos

```bash
less archivo.txt
```

Primeras líneas

```bash
head archivo.txt
```

Primeras 20 líneas

```bash
head -20 archivo.txt
```

Últimas líneas

```bash
tail archivo.txt
```

Monitorear un archivo en tiempo real

```bash
tail -f salida.out
```

---

# Edición de archivos

Abrir con Nano

```bash
nano archivo.txt
```

Guardar

```
Ctrl + O
```

Salir

```
Ctrl + X
```

---

# Copiar, mover y renombrar

Copiar archivos

```bash
cp archivo.txt copia.txt
```

Copiar directorios

```bash
cp -r carpeta respaldo
```

Mover archivos

```bash
mv archivo.txt carpeta/
```

Renombrar archivos

```bash
mv viejo.txt nuevo.txt
```

---

# Eliminar archivos

Eliminar archivo

```bash
rm archivo.txt
```

Eliminar preguntando antes

```bash
rm -i archivo.txt
```

Eliminar directorio

```bash
rm -r carpeta
```

Eliminar directorio forzadamente

```bash
rm -rf carpeta
```

> ⚠ **Advertencia:** `rm -rf` elimina permanentemente archivos y directorios. No existe papelera de reciclaje.

---

# Buscar archivos

Buscar todos los archivos FASTQ

```bash
find . -name "*.fastq.gz"
```

Buscar un archivo específico

```bash
find . -name "genome.fasta"
```

---

# Uso de comodines (*)

Todos los archivos TXT

```bash
ls *.txt
```

Todos los FASTQ

```bash
ls *.fastq.gz
```

Archivos que comienzan por SRR

```bash
ls SRR*
```

Archivos que contienen "trim"

```bash
ls *trim*
```

---

# Archivos comprimidos

Comprimir

```bash
gzip archivo.fastq
```

Descomprimir

```bash
gunzip archivo.fastq.gz
```

Visualizar un FASTQ comprimido

```bash
zcat archivo.fastq.gz | head -12
```

---

# Permisos

Dar permisos de ejecución

```bash
chmod +x script.sh
```

Ver permisos

```bash
ls -l
```

---

# Ambientes Conda

Listar ambientes

```bash
conda env list
```

Activar ambiente

```bash
conda activate nombre_del_ambiente
```

Desactivar ambiente

```bash
conda deactivate
```

---

# Módulos del clúster

Ver módulos disponibles

```bash
module avail
```

Cargar un módulo

```bash
module load envs/anaconda3
```

Ver módulos cargados

```bash
module list
```

Descargar un módulo

```bash
module unload nombre_modulo
```

---

# Gestión de trabajos (SLURM)

Ver nodos disponibles

```bash
sinfo
```

Enviar un trabajo

```bash
sbatch script.sh
```

Ver trabajos en ejecución

```bash
squeue -u usuario
```

Cancelar un trabajo

```bash
scancel JOBID
```

Consultar información de un trabajo

```bash
scontrol show job JOBID
```

---

# Monitoreo del sistema

Espacio disponible

```bash
df -h
```

Tamaño de carpetas

```bash
du -sh *
```

Uso de memoria

```bash
free -h
```

Procesos activos

```bash
top
```

Salir de `top`

```
q
```

---

# Atajos útiles

| Acción | Atajo |
|---------|--------|
| Cancelar proceso | `Ctrl + C` |
| Limpiar pantalla | `Ctrl + L` |
| Autocompletar | `Tab` |
| Historial de comandos | ↑ ↓ |
| Buscar comando anterior | `Ctrl + R` |

---
# 📂 Transferencia de archivos

Durante el análisis bioinformático es frecuente ejecutar los programas en un clúster HPC y posteriormente descargar los resultados al computador personal para su inspección. Las herramientas más utilizadas para esta tarea son **scp** y **rsync**.

## SCP (Secure Copy)

`scp` permite copiar archivos o directorios entre el computador local y un servidor utilizando el protocolo SSH.

### Descargar un archivo desde el servidor

Los comandos para subir y bajar archivos del cluster al computador local o viceversa se corren en la consola del equipo local no dentro del cluster.

```bash
scp usuario@servidor:/ruta/al/archivo .
```

Ejemplo:

```bash
scp amateusr@hercules2:/scratchsan/amateusr/curso/fastqc/multiqc_report.html .
```

El punto (`.`) indica que el archivo se guardará en el directorio actual.

Para descargarlo en una carpeta específica (Windows PowerShell):

```powershell
scp usuario@servidor:/ruta/archivo.html "C:\Users\Usuario\Downloads\"
```

### Subir un archivo al servidor

```bash
scp archivo.txt usuario@servidor:/ruta/destino/
```

### Copiar un directorio completo

```bash
scp -r carpeta usuario@servidor:/ruta/destino/
```

El parámetro `-r` copia todos los archivos y subdirectorios de forma recursiva.

---

## Rsync

`rsync` realiza la misma función que `scp`, pero es más eficiente para archivos grandes o transferencias repetidas. Solo copia la información que ha cambiado, por lo que normalmente es más rápido y permite reanudar transferencias interrumpidas. No funcionará directamente en PowerShell a menos que instale una versión de rsync. Debe ejecutarse desde WLS como por ejemplo desde Ubuntu.

### Descargar un directorio

```bash
rsync -avP usuario@servidor:/ruta/resultados/ ./resultados/
```

### Subir un directorio

```bash
rsync -avP ./resultados/ usuario@servidor:/ruta/destino/
```

### Significado de los parámetros

| Opción | Descripción |
|---------|-------------|
| `-a` | Modo archivo (conserva permisos, fechas y estructura). |
| `-v` | Muestra información detallada durante la transferencia. |
| `-P` | Muestra el progreso y permite reanudar una transferencia interrumpida. |

---

## ¿Cuál utilizar?

| Herramienta | ¿Cuándo usarla? |
|--------------|----------------|
| **scp** | Transferencias rápidas de uno o pocos archivos. |
| **rsync** | Directorios grandes, proyectos completos o cuando la transferencia puede interrumpirse. |

> **Recomendación:** Para descargar reportes como **FastQC**, **MultiQC** o archivos pequeños, `scp` suele ser suficiente. Para copiar proyectos completos de bioinformática (FASTQ, BAM, VCF, ensamblajes, etc.), `rsync` es generalmente la mejor opción por su velocidad y capacidad para reanudar transferencias.

# Comandos más utilizados

| Categoría | Comando | Función |
|------------|----------|---------|
| Navegación | `pwd` | Mostrar directorio actual |
| Navegación | `cd` | Cambiar de directorio |
| Navegación | `ls -lh` | Listar archivos |
| Archivos | `touch` | Crear archivo vacío |
| Archivos | `mkdir` | Crear directorio |
| Archivos | `cp` | Copiar |
| Archivos | `mv` | Mover o renombrar |
| Archivos | `rm` | Eliminar archivo |
| Archivos | `rm -rf` | Eliminar directorios |
| Visualización | `cat` | Mostrar contenido |
| Visualización | `less` | Leer archivos largos |
| Visualización | `head` | Primeras líneas |
| Visualización | `tail -f` | Monitorear archivos |
| Bioinformática | `zcat archivo.fastq.gz \| head -12` | Revisar FASTQ comprimidos |
| HPC | `sbatch` | Enviar un Job |
| HPC | `squeue` | Ver Jobs |
| HPC | `scancel` | Cancelar Jobs |
| HPC | `sinfo` | Ver nodos disponibles |
| Conda | `conda activate` | Activar ambiente |
| Módulos | `module avail` | Ver módulos disponibles |
| Disco | `df -h` | Espacio disponible |
| Disco | `du -sh *` | Tamaño de carpetas |

---

# Recursos recomendados

- Linux Journey — https://linuxjourney.com/

---

# Próximo módulo

En el siguiente módulo aprenderemos a descargar datos de secuenciación desde el **Sequence Read Archive (SRA)** del NCBI utilizando **SRA Toolkit**, preparando los datos para los análisis bioinformáticos posteriores.
