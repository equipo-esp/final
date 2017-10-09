# Shiny

### Diana Fabián, Tania Gómez y Mario González ###

Para esta parte del proyecto usamos el paquete de R, Shiny, el cuál es una herramienta con la que se pueden crear aplicaciones web interactivas, que permiten a los usuarios interactuar con sus datos sin tener que manipular el código. La programación reactiva enfatiza el uso de: Valores que cambian en el tiempo así como expresiones que registran esos cambios. Esta aplicación, esta formado por 2 componentes: una secuencia de comandos de interfaz de usuario (UI, archivo ui.R), que es la responsable de controlar el diseño y aspecto de la aplicación y una secuencia de comandos del servidor (server.R), que contiene las instrucciones que su equipo necesita para construir la aplicación.

## Análisis de LISA a partir de datos de los destinos turísticos más visitados de la República Mexicana ##

Se realizó un análisis de auto-correlación espacial por medio de la I de Moran, ya que queremos saber cual es el comportamiento espacial de un conjunto de polígonos, en este caso las zonas más visitadas de la República Mexicana, y explorar si presentan auto-correlación espacial global y local. 
Para la evaluación del comportamiento de las variables  se utilizó la matriz de contigüidad, con la posibilidad de seleccionar el tipo de vecindad: Tipo reina (Queen), Tipo torre (rook), Vecinos más cercanos (kNN), definiendo el número de vecinos a evaluar y por distancia, así como la posibilidad de elegir la variable de estudio: Número de cuartos disponibles  promedio (NCDP), Número de cuartos ocupados promedio (NCOP) y Porcentaje de ocupación promedio (POHP), y el año de estudio: 2013, 2014, 2015 ó 2016.

![moran](https://github.com/equipo-esp/final/blob/master/09.png)

Se realizó una gráfica de dispersión de Moran (dinámica), donde se graficaron los valores estandarizados contra una variable de retraso espacial, la cual es un promedio de las unidades vecinas (contiguas). Con el fin de observar una relación en términos de desviaciones estándar alrededor de la media.

![moran](https://github.com/equipo-esp/final/blob/master/10.png)

Se realizó también un mapa en leaflet  (dinámico) que muestra el resultado de la I de Moran, con respecto al tipo de auto-correlación, donde al posicionarte el algún polígono se despliega una etiqueta que muestra el nombre de la variable de estudio y el valor de esta para el año que se está evaluando.

![moran](https://github.com/equipo-esp/final/blob/master/11.png)

## Metodología ##
### Para la construcción del Usuario

Se requiere instalar las librerías que se van a utilizar

```
library(shiny)
library(leaflet)
library(DT)
```
Shiny UI, incluye el mínimo código necesario para la Ui.R, sidebarLayout y sidebarPanel, construyen el plano para generar una composición estándar.
```
shinyUI(fluidPage(
  
  navbarPage("",
    tabPanel("LISA dinamica",
      titlePanel("Calculo de LISA en Destinos mÃ¡s visitados de la RepÃºblica Mexicana"),
      sidebarLayout(
        sidebarPanel(
```
Se crean botones de selección para el tipo de vecindad y se crea un panel de selección para seleccionar el numero de vecinos por medio de un widget deslizador, con el numero de vecinos a elegir, se crea otro panel condicional para la selección de distancia también por medio de un widget de selección, con respecto al umbral de distancia en km.
```
          # Select stuff to work with
          radioButtons('radio', label = 'Selecciona el tipo de vecindad:',  
                       choices = list("Tipo reina (queen)" = 1, "Tipo torre (rook)" = 2, 
                                      'Vecinos mas cercanos (kNN)' = 3, 'Distancia' = 4), 
                       selected = 1), 
          conditionalPanel(
            condition = "input.radio == 3", 
            sliderInput("knn_slider", 'Selecciona el nÃºmero de vecinos', 
                        min = 1, max = 32, value = 4)
          ), 
          conditionalPanel(
            condition = "input.radio == 4", 
            sliderInput("dist_slider", "Seleciona un umbral de distancia en km", 
                        min = 450, max = 3000, step = 10, value = 450)
          ),
```

