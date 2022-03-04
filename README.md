# R-CORE

Este sistema está basado en el núcleo de cálculo del Sistema R Plus y surge con la finalidad de  distribuir el proceso de cálculo del sistema para optimizar el tiempo de evaluación. Esta aplicación esta basada en .NET 5, por lo que puede ejecutarse en plataformas de Windows y Linux. 

A continuación se describen las principales caracteristicas, la sintaxis de los parámetros a utilizar y se añaden ejemplos para su ejecución.


# Sintaxis R-CORE
Definición de parámetros

|Descripción|Valores Permitidos|Comentarios|
|---|---|---|
|Archivo de evaluación|Archivo JSON "{file}"|Ruta del archivo JSON| 
|Tipo de Proceso |Texto - "L", "LC", "LS", "esc" |"L"=Carga de Portafolios, "LC"=Evaluación, "LS"=Generación de Resultados, "esc"=Evaluación de escenario| 
|Numero de Proceso|Numérico|1| 
|Total de Procesos|Numérico|2| 
|---|---|---|

## Archivo de evaluación

El archivo de evaluación, es un archivo en formato JSON el cual contiene una serie de propiedades las cuales corresponden a todas las funcionalidades que realiza el Sistema R Plus.

Ejemplo de archivo JSON
```sh
{
	"Perils":"S",
	"CutOffDate":"2022-01-20",
	"Portfolios": [
		{
		"FileName": "C:\\ERN\\R_CORE\\EQ_Ind.xml",
		"Type": 1
		},
		{
		"FileName": "C:\\ERN\\R_CORE\\EQ_Col.xml",
		"Type": 2
		}
  	]
}
```

### Propiedades JSON 

|Campo|Valores permitidos|Comentarios|
|---|---|---|
|Perils|"S", "SR", "H"|Amenaza -> S=Sismo, SR=Sismo (Regulatorio), H=Hidro| 
|Cutoff Date|yyyy-MM-dd|Fecha de Corte |
|Portfolios|["{Filename, Type}"]| Lista de Carteras {Type -> 1 = Independiente, 2 = Colectiva}|
|Results Path|"{path}"|Ruta donde se almacenaran los resultados|
|SystemPath|"{path}"| Ruta del Sistema |
|Tr|Cadena de texto separada por comas|Lista de periodos de retorno - > "0,50,100,200,500,1000,1500,2000,2500,4500"|
|Scenario|"{file}"|Ruta del archivo del Evento (formato zip) |
|extraFields|[]|Arreglo de campos que se extraeran de la cartera y se agregaran al archivo de primas por ubicación (shapefile)|
|---|---|---|


## Ejemplos de archivo de evaluación

- Cartera Independiente

```sh
{
	"Perils":"S",
	"CutOffDate":"2022-01-01",
	"Portfolios": [
		{
		"FileName": "C:\\ERN\\R_CORE\\Ejemplos\\EQ_Ind.xml",
		"Type": 1
		}
  	],
	"ResultsPath":"C:\\ERN\\R_CORE\\resultados",
	"SystemPath":"C:\\ERN\\R_CORE\\app\\R-Core",
    "Tr": "0,50,100,200,500,1000,1500,2000,2500,4500",
    "Scenario": null,
    "shapefiles": null,
    "extraFields": [
        "CampoX", "CampoY"
    ]
}
```

- Cartera Colectiva

```sh
{
	"Perils":"S",
	"CutOffDate":"2022-01-01",
	"Portfolios": [
		{
		"FileName": "C:\\ERN\\R_CORE\\Ejemplos\\EQ_Col.xml",
		"Type": 2
		}
  	],
	"ResultsPath":"C:\\ERN\\R_CORE\\resultados",
	"SystemPath":"C:\\ERN\\R_CORE\\app\\R-Core",
    "Tr": "0,10,20,50,100,200,250,500,1000,1500"
}
```


# Workflow

El sistema R-CORE consta de tres principales procesos, la carga de los portafolios, la evaluación y la generación de Resultados. Cada proceso realiza una operación en específico y durante la ejecución se genera un archivo de log en donde se registrará cada evento que realiza el proceso. 

## Carga de Portafolios

En este proceso, el sistema cargará cada cartera que se agrego en la lista de "Portfolios", este es el primer proceso que se debe de ejecutar ya que conviertira la cartera a un formato binario y lo almacenará en la ruta de resultados para su posterior lectura en la evaluación.

```sh
#ProcessType = L -> Carga de la cartera
#Ejemplo ejecución en Windows
R_Core.exe "C:\ERN\R_CORE\Ejemplos\testInd.json" "L" 1 1 
#Ejemplo ejecución en Linux
dotnet "R_Core.dll" "/home/hiar/ERN/Examples/testInd.json" "L" 1 1 
```

## Evaluación

Tras haber generado la cartera, el proceso de cálculo se puede distribuir en "N" número de procesos para optimizar los tiempos de la evaluación, en este ejemplo dividiremos el proceso de cálculo en 5 y  ejecutaremos simultaneamente el mismo número de procesos de la siguiente forma:

```sh
#ProcessType = LC -> Ejecución
R_Core.exe "C:\ERN\R_CORE\Ejemplos\testInd.json" "LC" 1 5
R_Core.exe "C:\ERN\R_CORE\Ejemplos\testInd.json" "LC" 2 5
R_Core.exe "C:\ERN\R_CORE\Ejemplos\testInd.json" "LC" 3 5
R_Core.exe "C:\ERN\R_CORE\Ejemplos\testInd.json" "LC" 4 5
R_Core.exe "C:\ERN\R_CORE\Ejemplos\testInd.json" "LC" 5 5
```

Cada proceso generará un par de archivos binarios con la información correspondiente a las pérdidas para su posterior acumulación en la generación de Resultados.

## Resultados

El último proceso se encarga de realizar la lectura de los archivos binarios generados durante la evaluación y "acumularlos" para poder generar todos los archivos de resultados.

```sh
#ProcessType = LS -> Generación de Resultados
R_Core.exe "C:\ERN\R_CORE\Ejemplos\testInd.json" "LS" 1 5
```


```sh
R_Core.exe "C:\ERN\R_CORE\Ejemplos\testInd.json" "L" 1 1 
#Se deberá de esperar a que finalice el proceso de carga para posteriormente ejecutar el proceso de evaluación
R_Core.exe "C:\ERN\R_CORE\Ejemplos\testInd.json" "LC" 1 5
R_Core.exe "C:\ERN\R_CORE\Ejemplos\testInd.json" "LC" 2 5
R_Core.exe "C:\ERN\R_CORE\Ejemplos\testInd.json" "LC" 3 5
R_Core.exe "C:\ERN\R_CORE\Ejemplos\testInd.json" "LC" 4 5
R_Core.exe "C:\ERN\R_CORE\Ejemplos\testInd.json" "LC" 5 5
#Se deberá de esperar a que finalicen los procesos de evaluación para ejecutar el proceso de resultados
R_Core.exe "C:\ERN\R_CORE\Ejemplos\testInd.json" "LS" 1 5
```
