#!/bin/bash

### Este readme explica los comandos necesarios para la generacion del gif del trabajo final del ramo Datos Geoespaciales.

## Este ciclo nos sirve para generar un mapa para cada dia desde el 21 de enero al 2023 al 22 de marzo del mismo año

for x in {001..061}



do




## Definimos variables

name="figura$x" #nombre archivo de salida
region=-75/-70/-39/-35 #region mapa
j=M12 #proyeccion mercator
paleta="mby.cpt" # paleta para topografia
Sudamerica="-84/-33/-60/13" #region mapa pequeño
jc="-59/-19/0.9i" #proyeccion mapa pequeño
axs=a1f1 # intervalo entre ejes y grilla




##     MAPA


## Construimos el mapa de la region de estudio con psbasemap. -R para la region, -J la proyeccion que en este caso es Mercator12 con 12 los centimetros (tamaño). -B indica los ejes, a intervalo entre lat y lon, g para la grilla y el numero asociado para el intervalo de grilla, luego f indica el intervalo de grado entre barra de color negro y blanco. Las letras -X y -Y para posicionar el mapa dentro de la hoja (en este caso 3.5cm a la derecha y al centro en Y). -P (Portrait) retrato si no lo colocamos la imagen no se ve en el centro como lo indica el codigo. -K para mantener el archivo abierto (nunca se coloca en la ultima linea).
gmt psbasemap -R${region} -J${j} -B${axs} -X3.5 -Yc -P -K > ${name}.ps  

##    TOPOGRAFIA

##Descargamos datos de topografia de 60 arco-segundos en formato NetCDF (Network Common Data Form) el cual sirve para almacenar datos de grillas. Luego, pasamos estos datos al formato .grd (Grid Format) para trabajar en GMT, este ultimo es mas eficiente ya que almacena la informacion en un formato binario, se pueden hacer calculos de gradiente, suavisado, etc. Este formato es compatible con GMT. Con gmt grdconvert pasamos nuestra grilla .nc a .grd

#gmt grdconvert ETOPO_2022_v1_60s_N90W180_bed.nc -Gtodagrilla.grd

## CORTAMOS grilla para no ejecutar toda la topografia del planeta tierra sino que solo la zona de estudio que se muestra en nuestro mapa. -G nos da la salida -R la region donde queremos cortar todagrilla.grd y -V (verbose) que es el modo detallado. 

#gmt grdcut todagrilla.grd -Ggrilla.grd -R${region} -V 


## Luego con grdinfo podemos ver informacion sobre nuestra nueva grilla como resolucion, valores de lat y lon, altura o profundidad,etc. Donde podemos ver valores maximos y minimos de z y asi modificar la paleta que se ajuste a estos limites.

#gmt grdinfo -F grilla.grd

## Ahora pasamos la grilla a una imagen la cual superponemos al mapa, este comando grdimage asocia la grilla.grd a una paleta de colores para graficar puntos en el mapa.

#gmt grdimage grilla.grd -C${paleta} -R${region} -J${j} -B${axs} -O -K >> ${name}.ps



##     ILUMINACION

#Cambia bastante la figura con y sin iluminacion. Lo que hace este comando es generar sombra entonces se ven mucho mejor las montañas, batimetria, relieve en general. Con grdgradient colocamos nuestra grilla.grd donde elegimos un angulo de sombra -A angulo de 0 a 360. -N nos indica la direccion en que se calcula el gradiente (en este caso donde la pendiente es mayor) -G da la salida
gmt grdgradient grilla.grd -Nt1 -A360 -Gilumi.int 

#Podemos suavizar la iluminacion con el siguiente comando el cual realiza operaciones aritmeticas sobre grillas 
#gmt grdmath 0.5 ilumi.int MUL = iluminado.int (en este caso no utilizamos esta opcion)


gmt grdimage grilla.grd  -Iilumi.int -C${paleta} -R${region} -J${j} -C${paleta} -O -K >> ${name}.ps

#Podemos modificar la paleta de forma manual abriendo el archivo.cpt o con el comando gmt makecpt -Cpaleta_entrada.cpt -Tz_min/z_max/paso > paleta_salida.cpt


##   BARRA DE COLORES TOPOGRAFIA 

# Con -D podemos ver la posicion de la escala y sus dimensiones -D disthorizontal/distvertical/ancho/largo de la barra (en centimetros) si le agregamos una letra h al final obtenemos la barra pero en posicion horizontal. Con -B a2000 nos muestra la escala de la paleta cada 2000 metros, luego f500 muestra cada 500 metros marcas menores sin valor numerico asociado. De la siguiente forma el texto nos queda arriba de la barra -Ba2000f500/a2000f500:"Elevación (m)":


gmt psscale -R${region} -J${j} -D13.8c/3c/6c/0.5c  -C${paleta} -Ba2000f500:"Elevation (m)": -O -K >> ${name}.ps


##   LINEA COSTERA

#Sacamos  -G y -S ya que estos marcan el color del continente y el mar respectivamente. Y estos colores estan dados por la paleta de colores asociada a la topografia. Definimos la resolucion del mapa -Di (intermedia). En -N vemos  el tipo de fronteras que en este caso es 1 para territorios nacionales/ luego tenemos el espesor en pulgadas, y el color. En -W tenemos la linea de costa con 0.5pulgadas de espesor, de color negro. Tambien podemos agregar los rios con -I(1 a 10) e impedir ver islas de menor area a la especificada en -A.

gmt pscoast -R${region} -J${j} -B${axs} -Di -N1/0.7p,128 -W0.5p,0/0/0 -Lf-72.5/-38.7/-38/100 -Tf-72/-35.5/1/2  --HEADER_FONT_SIZE=7 -O -K  >> ${name}.ps


## ESCALA Y ROSA CARDINAL EN BLANCO Y NEGRO 

#Agregamos una escala con (-L), aparece en blanco y negro (f) en la posición longitud/latitud/con distancias validas sobre latitud -38/con un largo de 100 km. Y agregamos también una Rosa de los vientos (-T) en blanco y negro (f) ubicada en la posición longitud/latitud/tamaño 1/ y las cuatro direcciones cardinales son subdivididas otra vez en 2. El tamaño de la escritura en la Rosa de los vientos HEADER_FONT_SIZE=7.

## Colocamos la unidad de medida "km" sobre nuestra escala generada con -L ya que no pude colocarlo con el mismo codigo en pscoast.

echo "-72.59 -38.6 km" | gmt pstext -R${region}  -J${j} -V -F+f9p,Helvetica-Bold,black+jLM -O -K >> ${name}.ps

#Aqui ploteamos un circulo que indica la ubicacion de Concepcion con echo longitud latitud | gmt psxy para plotear figuras -S no indica el tipo de figura en este caso un circulo luego el espesor de 0.25 centimetros. En -W indicamos el grosor del contorno de la figura y el color. En -G seleccionamos el color del circulo.

#Concepcion
echo -73.03 -36.83 |gmt psxy -J${j} -R${region} -V -Sc0.2c -W0.2c/255/0/200 -G255/175/0 -O -K >> ${name}.ps

#Santa Juana  
echo -72.94259 -37.1737794 | gmt psxy -J${j} -R${region} -V -Sc0.2c -W0.2c/255/0/200 -G255/175/0 -O -K >> ${name}.ps

#Los Angeles
echo -72.35366 -37.46973 | gmt psxy -J${j} -R${region} -V -Sc0.2c -W0.2c/255/0/200 -G255/175/0 -O -K >> ${name}.ps

# Asociado al circulo colocamos el nombre de la ciudad con echo las coordenadas y el texto | utilizamos gmt pstext para ver el tipo de letra (Helvetica-Bold), el tamaño +f10p, el color 'black'. Con -Gcolor(R/G/B) podemos colocar un fondo para el texto
echo "-72.93 -36.83 Concepci\363n" | gmt pstext -R${region}  -J${j} -V -F+f7p,Helvetica-Bold,black+jLM -O -K >> ${name}.ps

echo "-72.87 -37.1737794 Santa Juana" | gmt pstext -R${region}  -J${j} -V -F+f7p,Helvetica-Bold,black+jLM -O -K >> ${name}.ps

echo "-72.28 -37.46973 Los Angeles" | gmt pstext -R${region}  -J${j} -V -F+f7p,Helvetica-Bold,black+jLM -O -K >> ${name}.ps


#Proyecciones
#https://pirlwww.lpl.arizona.edu/resources/guide/software/GMT/doc/html/GMT_Docs/node149.html (
#https://www.generic-mapping-tools.org/GMT.jl/stable/gallery/mapprojs/



## 	FOCOS DE INCENDIO

#En este comando ploteamos todos los puntos de brillo asociado a cierto dia donde se generaron anteriormente los archivos dia*.txt con todos los datos de brillo de ese dia. El ciclo va de 1 a 61 en las fechas de 2023-01-21 al 2023-03-22. -S nos da la opcion de elegir la forma de los puntos en este caso de cuadrados (s) y el tamaño que en este caso son 0.16 centimetros lo que se aproxima a 2km por 2km por lo que vi comparando con google earth. Igual se pudo haber hecho de mejor forma con la escala del mapa. -W0.1p nos da el ancho del contorno de cada cuadrado y -C lee la paleta de colores asociada a estos datos.
   
gmt psxy dia$x.txt -R${region} -J${j} -Ss0.16 -W0.1p -Cbrig.cpt -O -K >> ${name}.ps


#La paleta que descargue venia con colores invertidos, de negro a blanco, terminando los valores mas altos en color blanco y los bajos en negro, asi que inverti solo las columnas de los colores y coloque cada fila de mi .cpt en un txt, para luego unirlos por columnas con paste
#more fire.cpt | awk '{print $2}' | tac > efe2.txt # con tac invertimos la columna 4 de mi .cpt (lo mismo para $4)
#more fire.cpt | awk '{print $1}' > efe1.txt # aqui guardamos la primera columna de mi .cpt que son valores sin invertir (lo mismo para $3)
#paste efe*.txt > brig.cpt #unimos las cuatro columnas.


##     ESCALA FOCOS DE INCENDIO

# Modificamos -D para obtener una posicion adecuada de la escala dentro del mapa
gmt psscale -R${region} -J${j} -D1.1c/1.7c/3c/0.3c  -Cbrig.cpt -Ba20f5/a20f5:"Bright (K)": -O -K >> ${name}.ps
#Aunque lo mas correcto viendo la informacion de los datos seria colocar temperature y no bright.





##   TEXTO  FECHA


#fech va tomando las filas de fecha.txt que es nuestro archivo de todas las fechas desde 2023-01-21 al 2023-03-22 por lo tanto hay 61 datos.

fech=$(awk "NR == $x" fecha.txt) 

echo $fech

echo -73.6 -35.1 "Date: $fech" | gmt pstext  -R${region}  -J${j} -V -F+f10p,Helvetica-Bold,green+jLM  -Gwhite -O -K >> ${name}.ps

# Asi ploteamos la fecha de cada dia en la posicion indicada con longitud y latitud. Ademas de poder ajustar el tamaño de 10 pixeles, el tipo de letra Helvetica-Bold, alinear texto y el color de letra (todo en -F), en un fondo blanco -Gwhite.


##	VECTORES


# Aqui tomamos cada fila (a medida que avanza el ciclo) del vector new_*.txt de datos que contiene longitud latitud direccion en grados (azimut) y velocidad en km/h del viento. Esto para cada estacion
contu=$(awk "NR == $x" new_contulmo.txt) 
lacol=$(awk "NR == $x" new_lacolonia.txt) 
laspuen=$(awk "NR == $x" new_laspuentes.txt) 
manza=$(awk "NR == $x" new_manzanares.txt) 
yung=$(awk "NR == $x" new_yungay.txt) 

# Con la opcion -Sv le decimos que es un vector con ancho_vector/ancho_flecha/largo_flecha en centimetros
# Multiplicamos la velocidad en km/h por un factor de 0.1 porque sino el largo del vector era excesivamente grande en muchos dias, aunque creo que 0.1 tambien fue excesivamente pequeño en algunos casos.

echo "$contu" | gmt psxy  -R${region} -J${j} -Sv0.06c/0.15c/0.08c  -Gblack -O -K >> ${name}.ps

echo "$lacol" | gmt psxy  -R${region} -J${j} -Sv0.06c/0.15c/0.08c  -Gblack -O -K >> ${name}.ps

echo "$laspuen" | gmt psxy  -R${region} -J${j} -Sv0.06c/0.15c/0.08c  -Gblack -O -K >> ${name}.ps

echo "$manza" | gmt psxy  -R${region} -J${j} -Sv0.06c/0.15c/0.08c  -Gblack -O -K >> ${name}.ps

echo "$yung" | gmt psxy  -R${region} -J${j} -Sv0.06c/0.15c/0.08c  -Gblack -O -K >> ${name}.ps



# Vector de referencia

echo "-73.79 -36.5 80 1" | gmt psxy -R${region} -J${j} -SV0.06c/0.15c/0.08c -G0 -O -K >> ${name}.ps

echo "-73.9 -36.6 Wind Speed" | gmt pstext -R${region} -J${j} -V -F+f7p,Helvetica-Oblique,black+jLM -Gwhite -O -K >> ${name}.ps

echo "-73.9 -36.7 ~10 km/h" | gmt pstext -R${region} -J${j} -V -F+f7p,Helvetica-Oblique,black+jLM -Gwhite -O -K >> ${name}.ps



##     MAPA INSERTADO

#Mini map ploteamos la costa con otra region que correponde a Sudamerica y otra proyeccion las cuales definimos al comienzo del script. Seleccionamos valores de -X y -Y para localizar el minimapa dentro del principal. Luego, le damos el ancho a la linea de costa en el mini mapa y el color del continente. -Bg6 nos indica que hay grilla cada 6 grados. -JC proyección cilíndrica equidistante. ${jc} nos entrega donde se centra la proyeccion lon/lat/tamañopulgadas. -W grosor de las lineas y -G color continente.

echo -74.86 -35.23 "Study Zone" | gmt pstext -R${r}  -J${j} -P -V -F+f10p,Helvetica-Bold,white+jLM -Gblack -O -K >> ${name}.ps

gmt pscoast -R${Sudamerica} -JC${jc} -Bg6 -Di -X+0.2 -Y7.5 -A12 -W0.25p -G255/187/86 -O -K >> ${name}.ps


#Colocamos el cuadrado que identifica a la zona de estudio (Region del Bio Bio)
echo ${region} | sed 's/\// /g' | awk '{printf"%s %s\n %s %s\n %s %s\n %s %s\n %s %s\n", $1, $3, $2, $3, $2, $4, $1, $4, $1, $3}'| gmt psxy -R${Sudamerica} -JC${jc} -W0.7p -A100 -O >> ${name}.ps

#Leemos la region con lon min max y lat min max y le sacamos el slash entre medio para dejar solo numeros, donde luego...

#La secuencia "%s %s\n %s %s\n %s %s\n %s %s\n %s %s\n" define el formato de salida. Cada par de %s representa un valor que será reemplazado por las coordenadas correspondientes.

#El orden de las coordenadas en la salida será: lon_min lat_min, lon_max lat_min, lon_max lat_max, lon_min lat_max y nuevamente lon_min lat_min (para cerrar el rectángulo).

#Finalmente se plotea este rectangulo en la proyeccion y region del mapa pequeño.


#Pasamos imagen .ps a .png
gmt psconvert ${name}.ps -A -TG -V

done

#Forma de pasar imagenes.png a un gif
#convert -delay 10 -loop 1 figura*.ps focos.gif
#en este caso lo hice en matlab, no hubo caso de que me funcionara, no se si es la virtual box pero para la tarea anterior me funciono bien.

