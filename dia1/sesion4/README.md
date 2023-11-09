# Sesión 4: Gestión de datos antiguos

En esta sesión aprenderemos a realizar el llamado de variantes en genomas antiguos utilizando el programa pileupCaller (https://github.com/stschiff/sequenceTools).

A diferencia de otros programas para realizar llamado de variantes (e.g. GATK), este programa puede generar llamadas "pseudo-haploides", en donde se escoge al azar una lectura por posición. Esta estrategia se utiliza porque la generación de genotipos diploides en genomas antiguos se ve dificultada por la baja profundidad de los genomas y daño, lo que aumenta la probabilidad de errores.

Los archivos a utilizar en este práctico se encuentran en el fichero:
`/home/shared/cursojnab/dia1/sesion4/`. Crear fichero `JNAB/dia1/sesion4/` en `home` y copiar archivos.  

El llamado de variantes requiere de:
- Archivos bam
- Archivos con las variantes que se quieren generar (set de datos de referencia: *.geno/*.snp/*.ind)
- Genoma de referencia (*.fa or *.fasta)

## 1: Generar archivo de input para pileupCaller

pileupCaller requiere de un archivo mpileup para hacer el llamado de variantes pseudo-haploides. Esto se puede realizar por genoma o con múltiples genomas a la vez (opción `-b`) utilizando `samtools mpileup` (https://www.htslib.org/doc/samtools-mpileup.html). Además, podemos restringir el procedimiento a un set específico de variantes utilizando la opción `-l` y un archivo con dos columnas (cromosoma posición).

- Un genoma:
  `samtools mpileup -R -B -q30 -Q20 -l db1.pos -f ref.fa sample1.bam > sample1.mpileup`
- Varios genomas:
  `samtools mpileup -R -B -q30 -Q20 -l db1.pos -f ref.fa -b bamlist.txt > genomas_antiguos.mpileup`

Explorar archivos bam:
- ¿Cuántas lecturas tiene cada archivo bam?
- ¿Qué parte del genoma está representada? (tip: mirar header de archivos: `samtools view -H sample1.bam`)
- ¿Qué programas se han utilizado en la generación de los archivos bam?

Identifica o crea los archivos necesarios para genera el archivo mpileup de los 4 bam disponibles.
- Generar archivo `*.pos` desde archivo `*.snp`:
  `awk '{print $2, $4}' <prefix>.snp > <prefix>.pos`
- Generar lista de archivos bam:
  `ls *bam > bamlist.txt`

Correr `samtools mpileup` usando `sbatch`(script `1_mpileup.sbatch`)

## 2: pileupCaller

Ejemplo:
`pileupCaller --randomHaploid --sampleNames sample1,sample2,sample3,sample4 -f southamerica_masked_subset_ID.snp -e genomas_antiguos.pileUp < genomas_antiguos.mpileup`

`sampleNames` es una lista separada por , con el nombre de los individuos. Estos serán los nombres de individuos en el nuevo archivo eigenstrat *.ind. ¿A qué columna en el archivo *.ind corresponde?
`–samplePopName` -> similar  `sampleNames`, pero aplicado a poblaciones. ¿A qué columna en el archivo *.ind corresponde?

Correr `pileupCaller` usando el script `2_pileupCaller.sbatch`

¿Qué otras opciones de llamado de variantes se pueden realizar?

## 3: Unir nuevos datos a set de referencia

Utiliza la herramienta mergeit aprendida en secciones anteriores para unir la base de datos de referencia y los nuevos genotipos pseudo-haploides.

## 4: Generar variantes para genomas completos
En esta sección utilizaremos los archivos trabajados en la sesión 3 para hacer el llamado de variantes, esta vez utilizando archivos bam de genomas completos. Repetir los pasos 1 a 3 generando los archivos necesarios.
