### En este readme se explica como fue el tratamiento de datos para el trabajo final del ramo Datos Geoespaciales. 


## 	FOCOS DE INCENDIO


## Cambiamos las comas del archivo .txt de incendios por espacio para diferenciar entre columnas

#more ArchiveJ1_VIIRS_C2_South_America_VJ114IMGTDL_NRT_2023021_081.txt | sed 's/,/ /g' > sincomas.txt

## Sacamos la primera fila de texto

#more sincomas.txt | awk 'NR>1{print $0}' > data.txt

## Obtenemos los datos que estan entre los limites geograficos de la Region del Bio Bio

#more data.txt  | awk '{if ( $1*1 > -38.77 && $1*1 <= -36.5 && $2*1 > -73.57 && $2*1 <= -70.28 ) print $0}' > biobiodata.txt

Asi, tenemos todos los datos obtenidos por el satelite dentro de la Region del Bio-Bio


## Generamos un archivo de fechas para luego ingresar esas fechas en un grep y que busque los datos asociados a esas fechas, ademas nos sirve para plotear las fechas como texto en el mapa.


start_date="2023-01-21" #fecha de inicio
end_date="2023-03-23"  #fecha de termino

#Este ciclo nos dice que mientras current_date sea distinta a la de termino, coloque la fecha en fecha.txt donde cada vez se va sumando un dia en el formato de fecha indicado hasta llegar a la fecha final, por eso se coloca hasta el dia 23 de marzo y no el ultimo dia que es el 22, si colocamos hasta el 22 no tomaria este dia.

current_date="$start_date"
while [[ "$current_date" != "$end_date" ]]; do
    echo "$current_date" >> fecha.txt
    current_date=$(date -d "$current_date + 1 day" +%Y-%m-%d)
done

#Con esto tenemos nuestro archivo de fechas (para el periodo de verano del año 2023).


## Como ya tenemos nuestro archivo de fecha.txt creado, lo que hacemos es un ciclo que vaya leyendo cada fecha de este archivo y que busque todos los datos para un cierto dia en el archivo de la region del biobio llamado biobiodata.txt. Por ejemplo para la fecha 2023-01-21 guarda todos los datos de esa fecha en el archivo dia001.txt y asi hasta la fecha de termino completando un total de 61 archivos hasta dia061.txt

archivo="fecha.txt"
data=biobiodata.txt
sum=000

while IFS= read -r fecha; do
    echo "$fecha"
    sum=$((sum+001)) #hacemos un contador 
    echo "$sum"
    x=$(printf "%03d" $sum) # lo dejamos en formato de tres digitos para que no haya ploblema con la lectrura del orden de los archivos.
    grep "$fecha" "$data" | awk '{print $2, $1, $3}' >> dia$x.txt #buscamos la fecha en los datos del biobio y guardamos la longitud latitud y brillo asociados a esa fecha.
done < "$archivo"


Tenemos los focos listos para pasar a los vectores

## DATOS DE VIENTO

Primero descargamos lo datos de direccon y velocidad del viento desde agrometeorologia.cl, vemos desde que fila hasta que fila de nuestro archivo se encuentran los datos que nos interesan, en este caso, entre la linea 6 y 68, luego las comas se cambian por espacios y las comillas por nada, para finalmente guardar solo las columnas de direccion y velocidad de viento.

more contulmo.csv | awk '68>NR && NR>6 {print $0}' | sed 's/,/ /g' | sed 's/"//g' | awk '{print $3, $2}' > contulmo.txt

y lo mismo para las otras estaciones.

Despues, cada valor de direccion y velocidad debe tener asociado la ubicacion de la estacion, por lo tanto hacemos un archivo donde se repita 61 veces la longitud y latitud de la estacion

# Contulmo -73.23 -38.01
# Yungay   -72.01 -37.14
# La Colonia -72.59 -37.28
#Las puentes -73.43 -37.30
#Manzanares -72.54 -37.77

echo "-73.23 -38.01" > contu.txt
for ((i=1; i<61; i++)); do echo "-73.23 -38.01" >> contu.txt; done

echo "-72.01 -37.14" > yunga.txt
for ((i=1; i<61; i++)); do echo "-72.01 -37.14" >> yunga.txt; done


echo "-72.59 -37.28" > lacolo.txt
for ((i=1; i<61; i++)); do echo "-72.59 -37.28" >> lacolo.txt; done

echo "-73.43 -37.30" > laspuen.txt
for ((i=1; i<61; i++)); do echo "-73.43 -37.30" >> laspuen.txt; done

echo "-72.54 -37.77" > manza.txt
for ((i=1; i<61; i++)); do echo "-72.54 -37.77" >> manza.txt; done


Ahora unimos nuestro archivo de lat y lon con sus respectivos archivos de direccion y velocidad(km/h)
los unimos por columnas con paste

paste contu.txt contulmo.txt > vec_contulmo.txt #Ejemplo para contulmo

y asi para las demas estaciones...
luego de esto tenemos nuestros archivos de viento listos para plotear.




