PAV - P4: reconocimiento y verificación del locutor
===================================================

Obtenga su copia del repositorio de la práctica accediendo a [Práctica 4](https://github.com/albino-pav/P4)
y pulsando sobre el botón `Fork` situado en la esquina superior derecha. A continuación, siga las
instrucciones de la [Práctica 2](https://github.com/albino-pav/P2) para crear una rama con el apellido de
los integrantes del grupo de prácticas, dar de alta al resto de integrantes como colaboradores del proyecto
y crear la copias locales del repositorio.

También debe descomprimir, en el directorio `PAV/P4`, el fichero [db_8mu.tgz](https://atenea.upc.edu/mod/resource/view.php?id=3654387?forcedownload=1)
con la base de datos oral que se utilizará en la parte experimental de la práctica.

Como entrega deberá realizar un *pull request* con el contenido de su copia del repositorio. Recuerde
que los ficheros entregados deberán estar en condiciones de ser ejecutados con sólo ejecutar:

~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~.sh
  make release
  run_spkid mfcc train test classerr verify verifyerr
~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~~

Recuerde que, además de los trabajos indicados en esta parte básica, también deberá realizar un proyecto
de ampliación, del cual deberá subir una memoria explicativa a Atenea y los ficheros correspondientes al
repositorio de la práctica.

A modo de memoria de la parte básica, complete, en este mismo documento y usando el formato *markdown*, los
ejercicios indicados.

## Ejercicios.

### SPTK, Sox y los scripts de extracción de características.

- Analice el script `wav2lp.sh` y explique la misión de los distintos comandos involucrados en el *pipeline*
  principal (`sox`, `$X2X`, `$FRAME`, `$WINDOW` y `$LPC`). Explique el significado de cada una de las 
  opciones empleadas y de sus valores.
> `sox`: El comando sox sirve para realizar múltiples tareas sobre un fichero de audio, como cambiar su formato, y realizar operaciones de procesado de señal (transformada o reducción de ruido por ejemplo).
> 
> `$X2X`: Programa de SPTK que permite la conversión entre distintos formatos de datos.
> 
> `$FRAME`: Sirve para dividir la señal en tramas y extraer frame a frame toda una secuencia. Concretando para nuestro caso, hemos elegido tramas de longitud 240 (-l 240) y con un periodo de 80 (-p 80).
> 
> `$WINDOW`: Enventana los datos; Multiplica las tramas por la ventana de Blackman.
> 
> `$LPC`: Mediante el método Levinson-Durbin se calculan los distintos coeficientes LPC.

- Explique el procedimiento seguido para obtener un fichero de formato *fmatrix* a partir de los ficheros de
  salida de SPTK (líneas 45 a 51 del script `wav2lp.sh`).
> Mediante fmatrix creamos una matrix de n filas, que se corresponden a las tramas de la señal, y de m columnas, que se corresponden con los coeficientes de cada trama. El número de columnas será igual al número de coeficientes del orden del predictor lineal, más uno. El número de filas se calcula inicialmente convirtiendo la señal a texto y se cuentan las filas mediante el comando UNIX wc -l.

  * ¿Por qué es más conveniente el formato *fmatrix* que el SPTK?
> Utilizando el formato fmatrix podemos transformar nuestro vector de entrada a matriz, teniendo un rápido acceso a los datos de nuestra señal. Además, su cabecera ofrece información sobre número de tramas y coeficientes calculados.

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales de predicción lineal
  (LPCC) en su fichero <code>scripts/wav2lpcc.sh</code>:
> `sox $inputfile -t raw -e signed -b 16 - | $X2X +sf | $FRAME -l 240 -p 80 | $WINDOW -l 240 -L 240 |
    $LPC -l 240 -m $lpc_order > $base.lp || exit 1`

- Escriba el *pipeline* principal usado para calcular los coeficientes cepstrales en escala Mel (MFCC) en su
  fichero <code>scripts/wav2mfcc.sh</code>:
> `sox $inputfile -t raw -e signed -b 16 - | $X2X +sf | $FRAME -l 240 -p 80 | $WINDOW -l 240 -L 240 |
    $MFCC -l 240 -s 8 -m $mfcc_order -n $melfilter_bank_order > $base.lp || exit 1`

### Extracción de características.

- Inserte una imagen mostrando la dependencia entre los coeficientes 2 y 3 de las tres parametrizaciones
  para todas las señales de un locutor.
  > !(https://cdn.discordapp.com/attachments/1081506947892777012/1117499763793862676/image.png)
  > !(https://cdn.discordapp.com/attachments/1081506947892777012/1117500271086551130/image.png)
  > !(https://cdn.discordapp.com/attachments/1081506947892777012/1117500320101183548/image.png)

  + Indique **todas** las órdenes necesarias para obtener las gráficas a partir de las señales 
    parametrizadas.
    > Inicialmente, hemos ejecutado el script run_spkid para cada una de las predicciones. Éste calcula para cada frase de cada interlocutor todas sus predicciones y las almacena:
   
    `FEAT=lp /home/rogerurbieta/PAV/bin/run_spkid lp`
    
    FEAT=lpcc /home/rogerurbieta/PAV/bin/run_spkid lpcc`
    
    FEAT=mfcc /home/rogerurbieta/PAV/bin/run_spkid mfcc`
    
    > Posteriormente guardamos en un archivo .txt los coeficientes correspondientes a rho[2] y rho[3] para después hacer las gràficas. En nuestro caso lo hacemos con el interlocutor SES013.
    `fmatrix_show work/lp/BLOCK11/SES119/*.lp | egrep '^\[' | cut -f4,5 > graphics/lp.txt
    
    fmatrix_show work/lpcc/BLOCK01/SES119/*.lpcc | egrep '^\[' | cut -f4,5 > graphics/lpcc.txt
    
    fmatrix_show work/mfcc/BLOCK01/SES119/*.mfcc | egrep '^\[' | cut -f4,5 > graphics/mfcc.txt`
    
    > Por último, mediante Matlab representamos las gráficas de cada tipo distinto de predicción:
    `foto`

  + ¿Cuál de ellas le parece que contiene más información?
    > Las gráficas más incorreladas, es decir, aquellas con puntos más separados, son las que aportan mayor información. Por lo tanto, lpcc y mfcc comprobamos que aportan más información que lp.

- Usando el programa <code>pearson</code>, obtenga los coeficientes de correlación normalizada entre los
  parámetros 2 y 3 para un locutor, y rellene la tabla siguiente con los valores obtenidos.

  |                        | LP   | LPCC | MFCC |
  |------------------------|:----:|:----:|:----:|
  | &rho;<sub>x</sub>[2,3] |  -0,506348    |  0,30836    |   0,244409   |
  
  + Compare los resultados de <code>pearson</code> con los obtenidos gráficamente.
  > Analizando los resultados en valor absoluto, el valor de rho en LP es el mayor de los tres, indicando una correlación más alta. También vemos que LPCC y MFCC están similarmente incorrelados, lo cual ya se apreciaba en la gráfica.
  
- Según la teoría, ¿qué parámetros considera adecuados para el cálculo de los coeficientes LPCC y MFCC?
> Tanto LPCC como MFCC suelen tener alrededor de 13 coeficientes. Además MFCC suele tener entre 24-40 filtros.

### Entrenamiento y visualización de los GMM.

Complete el código necesario para entrenar modelos GMM.

- Inserte una gráfica que muestre la función de densidad de probabilidad modelada por el GMM de un locutor
  para sus dos primeros coeficientes de MFCC.
> !(https://cdn.discordapp.com/attachments/1081506947892777012/1117513443289878589/image.png)

- Inserte una gráfica que permita comparar los modelos y poblaciones de dos locutores distintos (la gŕafica
  de la página 20 del enunciado puede servirle de referencia del resultado deseado). Analice la capacidad
  del modelado GMM para diferenciar las señales de uno y otro.
> Comparando los locutores SES119 y SES119:
> !(https://cdn.discordapp.com/attachments/1081506947892777012/1117514855579127818/image.png)
> Comparando los locutores SES119 y SES118:
> !(https://cdn.discordapp.com/attachments/1081506947892777012/1117514919877820566/image.png)
> Comparando los locutores SES119 y SES117:
> !(https://cdn.discordapp.com/attachments/1081506947892777012/1117515022764085298/image.png)
> Comparando los locutores SES119 y SES120:
> !(https://cdn.discordapp.com/attachments/1081506947892777012/1117515180025315378/image.png)
> 
> Obviamente, el mejor resultado lo obtenemos en la primera gráfica, donde el locutor y las muestras coinciden.

### Reconocimiento del locutor.

Complete el código necesario para realizar reconociminto del locutor y optimice sus parámetros.

- Inserte una tabla con la tasa de error obtenida en el reconocimiento de los locutores de la base de datos
  SPEECON usando su mejor sistema de reconocimiento para los parámetros LP, LPCC y MFCC.
>   |                        | LP   | LPCC | MFCC |
>   |------------------------|:----:|:----:|:----:|
>   | Tasa error |   8,79%   |  0,51%  |   1,53%   |

### Verificación del locutor.

Complete el código necesario para realizar verificación del locutor y optimice sus parámetros.

- Inserte una tabla con el *score* obtenido con su mejor sistema de verificación del locutor en la tarea
  de verificación de SPEECON. La tabla debe incluir el umbral óptimo, el número de falsas alarmas y de
  pérdidas, y el score obtenido usando la parametrización que mejor resultado le hubiera dado en la tarea
  de reconocimiento.
>   |                        | LP   | LPCC | MFCC |
>   |------------------------|:----:|:----:|:----:|
>   | Threshold |   4,894922244328  |  1,2008614748077  |   1,637487330275   |
>   |------------------------|:----:|:----:|:----:|
>   | Missed |   40,8%   |  6,4%  |   8,4%   |
>   |------------------------|:----:|:----:|:----:|
>   | False Alarm |   2,4%   |  1,3%  |   1,4%   |
>   |------------------------|:----:|:----:|:----:|
>   | Cost Detection |   62,4   |  18,1  |   21   |
 
### Test final

- Adjunte, en el repositorio de la práctica, los ficheros `class_test.log` y `verif_test.log` 
  correspondientes a la evaluación *ciega* final.

### Trabajo de ampliación.

- Recuerde enviar a Atenea un fichero en formato zip o tgz con la memoria (en formato PDF) con el trabajo 
  realizado como ampliación, así como los ficheros `class_ampl.log` y/o `verif_ampl.log`, obtenidos como 
  resultado del mismo.
