# Sesión 6: Práctico Diversidad y Estructura poblacional
# F-statistics

En esta sesión aprenderemos a estimar la afinidad genética y mestizaje entre poblaciones con los índices f3 y D implementados en ADMIXTOOLS.

Trabajaremos con los archivos generados en sesiones anteriores. En particular, necesitaremos:
- Archivos en formato eigenstrat: *.geno, *.ind, *.snp
- Archivo con parámetros del análisis
- Archivo `popfilename`: archivo *.txt con lista de tests


## Outgroup-f3: caso especial de f3

La configuración del test f3 es la siguiente:

  f3(A,B; C)

En donde C es la población objetivo, mientras que A y B son las potenciales fuentes de mestizaje. El estadístico f3 puede tomar valores positivos o negativos, en donde valores negativos sugieren que la población objetivo (C) tiene evidencia de mestizaje desde A y B.

Un caso especial de f3 es el outgroup-f3, en donde en lugar de evaluar evidencia de mestizaje en C, evaluaremos la deriva genética compartida entre A y B, desde la separación con un outgroup (C). En este caso, asumiendo que C sea un outgroup a A y B, sólo obtendremos valores positivos. Mientras mayor sea el valor del estadístico, mayor es la deriva genética compartida entre A y B (o mayor afinidad genética). Los valores de z-score permiten evaluar la significancia del estadístico. Usualmente valores mayores 3 son considerados significativos.  

Para estimar el outgroup-f3 utilizaremos el módulo qp3pop de ADMIXTOOLS (https://github.com/DReichLab/AdmixTools/blob/master/README.3PopTest
). Los parámetros del análisis son especificados en un archivo de texto que debe tener, al menos, las siguientes líneas:

```

genotypename:   prefix.geno
snpname:   prefix.snp
indivname:   prefix.ind
popfilename:  example.txt
inbreed: NO

```

El archivo especificado en popfilename debe tener 3 poblaciones por línea, siendo la tercera posición ocupada por el outgroup. Por ejemplo: `popA popB Mbuti`. Este archivo puede tener tantas líneas como tests a evaluar.

El nombre de la "población" está determinado por la tercera columna del archivo *.ind.  

### Aplicación en Patagonia
Realizamos una seria de tests con el objetivo de evaluar las afinidades genéticas en la región de Patagonia. En primer lugar, evaluaremos la afinidad entre individuos tempranos (Holoceno Medio) y el resto de los grupos antiguos y modernos disponibles en el set de datos.

- Primero, usaremos R para reemplazar los ID de las poblaciones en el archivo *.ind:
Ejecutar R y realizar reemplazo de IDs de la siguiente forma:

```
d = read.table("prefix.ind")
r = read.table("metadata_curso_jnab2023_fn.csv", header = T, sep =",")
d$V3 = r$popID[match(d$V1, r$indivID)]
sum(is.na(d$V3))

```
Si el ouput del último comando (`sum`) es cero, continuar con

`write.table(d, "prefix_popID.ind", col.names = F, row.names = F, quote = F)`

- Segundo, elaboraremos una lista de los test a realizar:
  - Extraer lista única de poblaciones: 
  `cut -d" " -f3 prefix_popID.ind | sort | uniq > unique.poplist`
  - Crear archivo de texto con los IDs de los individuos tempranos (3º columna archivo *.ind) usando `nano target.poplist`:
```
  Chile_PuntaSantaAna_7300BP
  Chile_WesternArchipelago_Ayayema_4700BP
  Argentina_NorthTierradelFuego_LaArcillosa2_5800BP
```
  - Generar lista de comparaciones:
  
  ```
  for i in `cat target.poplist`;do for j in `cat unique.poplist`;do echo $i $j Mbuti;done;done | awk '$1 != $2' > PatagoniaMH.X.Mbuti.txt

  ```

- Tercero: correr qp3Pop usando sbatch (script `1_outgroupf3.sbatch`) y utilizando como `popfilename` el último archivo generado. En el archivo de parámetro, recuerda reemplazar el `indivname` por el archivo creado en R.

¿Qué otros test se pueden realizar?


### Análisis opcional:
En caso de haber realizado el paso 4 de la sesión 4, evaluaremos la afinidad genética de este individuo con todas las poblaciones o grupos del archivo eigenstrat. Para ello:
- Crear archivo poplistname:
  - Crear archivo de 3 columnas: 
  `awk '{print "genoma_antiguoX",$1,"Mbuti"}' unique.poplist`

- Modificar archivo `ejemplo.par` con los nombres correspondientes

- Correr qp3pop usando sbatch (modificar `1_outgroupf3.sbatch` según corresponda)

### Figuras

Usar script `plots_outgf3.R` en Rstudio para realizar plots.


## D-statistic

De forma similar al f3, este análisis también es un test formal de admixture pero basado en 4 grupos o poblaciones distintas. Este estadístico puede tener valores positivos o negativos. La significancia de los valores se evaluará mediante los valores de z-score (valor absoluto mayor a 3)  

Para estimar esta estadístico utilizaremos el módulo qpDstat de ADMIXTOOLS (https://github.com/DReichLab/AdmixTools/blob/master/README.Dstatistics
). Los parámetros del análisis son especificados en un archivo de texto que debe tener, al menos, las siguientes líneas:

```
genotypename: prefix.geno
snpname: prefix.snp
indivname: prefix.ind
printsd: YES
popfilename: example.txt

```

El archivo popfilename debe tener 4 poblaciones por línea. El orden dependerá del tipo de pregunta o test que se quiere evaluar. Por ejemplo, si queremos evaluar si un set de poblaciones "W" y "X" están igualmente relacionadas a "Y", seguiremos la siguiente anotación:

```
W1 X1 Y Z
W2 X1 Y Z
W3 X1 Y Z
W1 X2 Y Z
W2 X2 Y Z
W3 X2 Y Z
```

En la posición "W" hay 3 poblaciones distintas; en la posición "X", dos poblaciones distintas, mientras que "Y" y "Z" (outgroup) son fijas. Por otro lado, si queremos evaluar si existe evidencia de mestizaje o si un set de poblaciones "Y" están mas cerca de "W" o "X":

```
W X Y1 Z
W X Y2 Z
W X Y3 Z

```

En este caso, en la posición "Y" hay un set distinto de poblaciones, mientras el resto es fijo. En ambos ejemplos, la posición "Z" siempre estará ocupada por el outgroup. Considerando este último ejemplo, si el estadístico no es distinto de cero, "Y" está igualmente relacionado a "W" y "X". Valores significativamente positivos (z-score > 3), sugieren que existe flujo génico entre "W" e "Y", mientras que valores significativamente negativos (< -3) indicaran flujo génico entre "X" e "Y".

### Aplicación en Patagonia

Define la configuración del siguiente test:

- Evaluar si existe algún grupo o población ("W" y "X") mas cercana a alguno de los individuos del Holoceno Temprano de Patagonia ("Y").

Crea el archivo de parámetros y sbatch siguiendo los ejemplos de la sección anterior y otras sesiones. 


