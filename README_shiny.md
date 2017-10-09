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
      titlePanel("Calculo de LISA en Destinos más visitados de la República Mexicana"),
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
            sliderInput("knn_slider", 'Selecciona el número de vecinos', 
                        min = 1, max = 32, value = 4)
          ), 
          conditionalPanel(
            condition = "input.radio == 4", 
            sliderInput("dist_slider", "Seleciona un umbral de distancia en km", 
                        min = 450, max = 3000, step = 10, value = 450)
          ),
```
Se crean botones de selección para seleccionar la variable de estudio.
```
          # Select stuff to work with
          radioButtons('radio_pp', label = 'Selecciona:',  
                      choices = list("NCDP", "NCOP", "POHP"), 
                     selected = "NCDP"),
```
Se crea un widget deslizador por las diferentes opciones de años de estudio donde en valor minimo es 2013 y el máximo 2016.
```
          # Select year
          sliderInput("yearLISA",
                      "AÃ±os:",
                      min = 2013,
                      max = 2016,
                      value = 2013,
                      #step=6,
                      sep = "",
                      animate=TRUE
          )
        ),
```
Por último se usa la función *Output para construir el tipo de objeto que se requiere colocar en el Ui, y pasa la función *Output a una cadena de caracteres correspondientes al nombre del objeto en el Server.R.
```
        mainPanel(
          plotOutput("moranPlot", height = 400, brush = brushOpts(id = "plot_brush")),
          DT::dataTableOutput("table", width = '100%'),
          leafletOutput("LISAmap", width = '100%')
        )
      )
    )
  )
))
```

### Para la construcción del Servidor

Se requiere instalar las librerías que se van a utilizar
```
library(shiny)
library(rgdal)
library(sp)
library(spdep)
library(ggplot2)
library(GISTools)
library(RColorBrewer)
library(leaflet)
library(DT)
library(dplyr)
```
Se leen los datos de entrada y se da la instrucción del análisis de LISA por medio de vecindad tipo Queen y Rock.
```
# Leer shp 
edos <- readOGR("../data", "union_3", verbose = FALSE, stringsAsFactors = FALSE, GDAL1_integer64_policy = T)

# Los vecinos de todos los estados QUEEN
edos.nbq <- poly2nb(edos, queen = T)#, row.names = as.character(edos$ADMIN_NAME))
edos.nbq.w <- nb2listw(edos.nbq)
# Los vecinos de todos los estados ROOK
edos.nbr <- poly2nb(edos, queen = F)#, row.names = as.character(edos$ADMIN_NAME))
edos.nbr.w <- nb2listw(edos.nbr)
# Las coordenadas de los centroides de los estados
coords <- coordinates(edos)

# Define server logic required to draw stuff
 shinyServer(function(input, output) {
```
Se crea la representación gráfica por medio de un mapa y gráfica del análisis de los indicadores locales de asociación espacial. 
Se realiza la dinámica LISA, con respecto al año y la variable de estudio.
```
 # TAB 4: Dynamic LISA
  ############################
  
  yearLISA <- reactive({ 
    as.character(input$yearLISA) 
  })

  radio_pp <- reactive({ 
    as.character(input$radio_pp) 
  })
  
  selected <- reactive({
   
    var <- switch(paste(input$radio_pp,input$yearLISA,sep=""),
                 "NCDP2013" = edos$NCDP2013,
                 "NCDP2014" = edos$NCDP2014,
                 "NCDP2015" = edos$NCDP2015,
                 "NCDP2016" = edos$NCDP2014,
                 "NCOP2013" = edos$NCOP2013,
                 "NCOP2014" = edos$NCOP2014,
                 "NCOP2015" = edos$NCOP2015,
                 "NCOP2015" = edos$NCOP2016,
                 "POHP2013" = edos$POHP2013,
                 "POHP2014" = edos$POHP2014,
                 "POHP2015" = edos$POHP2015,
                 "POHP2016" = edos$POHP2016)
```
Se escribe la opción para poder seleccionar los tipos de vecindad que se utilizará para el análisis LISA, donde se considera distancia, vecinos más cercanos.
```
    # Distance-based spatial weights
    knn <- reactive({
      if (input$radio == 3) {
        k <- knearneigh(coordinates(edos), k = input$knn_slider, longlat = TRUE)
        return(k$nn)
      } else {
        return(NULL)
      }
    })
    
    dist <- reactive({
      if (input$radio == 4) {
        return(dnearneigh(coordinates(edos), 0, input$dist_slider, longlat = TRUE))
      } else {
        return(NULL)
      }
    })
```
Se realizan las condiciones para poder realizar el mapa del análisis LISA de acuerdo con el tipo de vecindad seleccionado.
```
    # LISA Map
    edo_weights <- reactive({
      if (input$radio == 1) {
        #return(nb2listw(include.self(poly2nb(edos))))
        return(nb2listw(poly2nb(edos)))
      } else if (input$radio == 2) {
        #return(nb2listw(include.self(poly2nb(edos, queen = FALSE))))
        return(nb2listw(poly2nb(edos, queen = FALSE)))
      } else if (input$radio == 3) {
        k <- knearneigh(coordinates(edos), k = input$knn_slider, longlat = TRUE)
        #return(nb2listw(include.self(knn2nb(k))))
        return(nb2listw(knn2nb(k)))
      } else if (input$radio == 4) {
        d <- dnearneigh(coordinates(edos), 0, input$dist_slider, longlat = TRUE)
        #return(nb2listw(include.self(d)))
        return(nb2listw(d))
      }
    })
```
Se definen las variables estandarizadas, se catalogan los valores de acuerdo al cuadrante en el que se encuentren en la gráfica de dispersión de Moran.
```
    
    edos$var <- var
    edos$var_lag <- lag.listw(edo_weights(), var)
    
    varStd <- (var - mean(var))/sd(var)
    lagStd <- lag.listw(edo_weights(), scale(var))
    
    mean <- mean(var)
    lag_mean <- mean(edos$var_lag) 
    
    global_moran <- moran.test(var, edo_weights())
    statistic <- (global_moran$estimate)
    statistic <- round(statistic, 2)
    lisa <- localmoran(var, edo_weights())
    
    edos$cuadrante <- c(rep(0,length(var)))
    significance <- 0.05
    vec <- ifelse(lisa[,5] < significance, 1,0)
    
    edos$cuadrante[var >= mean & edos$var_lag >= lag_mean]  <- 1
    edos$cuadrante[var < mean & edos$var_lag < lag_mean]  <- 2
    edos$cuadrante[var < mean & edos$var_lag >= lag_mean]  <- 3
    edos$cuadrante[var >= mean & edos$var_lag < lag_mean]  <- 4
    edos$cuadrante.data <- edos$cuadrante*vec
    
    edos$cuadrante.col[edos$cuadrante.data==1] <- "High-High"
    edos$cuadrante.col[edos$cuadrante.data==2] <- "Low-Low"
    edos$cuadrante.col[edos$cuadrante.data==3] <- "Low-High"
    edos$cuadrante.col[edos$cuadrante.data==4] <- "High-Low"
    edos$cuadrante.col[edos$cuadrante.data==0] <- "Non-sig"
    
    edos$fill <- factor(edos$cuadrante.data+1)
    edos$var_mean <- mean
    edos$var_lag_mean <- lag_mean
    edos$statistic <- statistic[1]
    
    edos.sel <- subset(edos, select = c(OBJECTID, CVE_MUN, var, var_lag, cuadrante, cuadrante.data,
                                        cuadrante.col, fill, var_mean, var_lag_mean, statistic))
  })
```
Por ultimo se definen los colores para los cúmulos y significancia tanto de la grafica de dispersión, como del mapa, y se da la instrucción de impresión de ambos.
```
  output$moranPlot <- renderPlot({
    cColors <- c(rgb(0.74, 0.74, 0.74, alpha = 0.2), rgb(1, 0, 0, alpha = 0.75),
                 rgb(0, 0, 1, alpha = 0.75), rgb(0.58, 0.58, 1, alpha = 0.75), rgb(1, 0.58, 0.58, alpha = 0.75))
    ggplot(selected()@data, aes(x=var, y=var_lag)) +
      geom_point(aes(fill = selected()$fill), colour="black", size = 3, shape = 21)+
      scale_fill_manual(name="",
                        values = c("1" = cColors[1], "2" = cColors[2], "3" = cColors[3], "4" = cColors[4], "5" =cColors[5]),
                        labels=c("Non-sig",
                                   paste0("High-High (", sum(selected()$cuadrante.data==1), ")"),  
                                 paste0("Low-Low (", sum(selected()$cuadrante.data==2), ")"),
                                 paste0("Low-High (", sum(selected()$cuadrante.data==3), ")"),
                                 paste0("High-Low (", sum(selected()$cuadrante.data==4), ")"))) +
      geom_vline(xintercept = unique(selected()$var_mean), colour = "grey", linetype = "longdash") +
      geom_hline(yintercept = unique(selected()$var_lag_mean), colour = "grey", linetype = "longdash") +
      stat_smooth(method="lm", se=FALSE, colour = "black", size = 0.5) +
      xlab(paste("\n",input$radio_pp," por aÃ±o (", input$yearLISA,")")) +
      ylab("\nRetraso espacial de ocupaciÃ³n por zona turÃ­stica") +
      theme_bw() +
      ggtitle(paste0("I de Moran: ", unique(selected()$statistic),"\n")) +
      theme(plot.title = element_text(color = "darkorchid")) 
  })
  
   output$LISAmap <- renderLeaflet({
     cColors <- c(rgb(0.74, 0.74, 0.74), rgb(1, 0, 0),
                  rgb(0, 0, 1), rgb(0.58, 0.58, 1), rgb(1, 0.58, 0.58))
     factpal <- colorFactor(cColors, 
                            domain = c("0", "1", "2", "3", "4"))
     
     popup <- paste(input$radio_pp,":", round(selected()$var,2), "en", input$yearLISA)
     leaflet(edos) %>%
       addProviderTiles('CartoDB.Positron') %>%
       addPolygons(data = selected(), fillColor = ~factpal(cuadrante.data), 
                   fillOpacity = 0.7, color = "black", weight = 1, label = popup) %>%
       addLegend(position = "topright", colors = cColors,
                 labels = c("Non-sig", "High-High", "Low-Low", "Low-High", "High-Low"), opacity = 0.7)
   })
   
   brushed <- eventReactive(input$plot_brush, {
     brushedPoints(selected(), input$plot_brush)
   })
  
  output$table <- DT::renderDataTable({
    tbl <- brushed() %>%
      as.data.frame() %>%
      select(Estado = radio_pp, Homicidios = var, Tipo = cuadrante.col)
  },
  rownames = FALSE, options = list(pageLength = 5, dom = 'tip', autoWidth = FALSE))
 
  observe({
    req(brushed())
    popup <- paste(brushed()$radio_pp, ":",round(brushed()$var,digits=2), "en", input$yearLISA)
    leafletProxy('LISAmap') %>%
    clearGroup(group = 'brushed') %>%
    addPolygons(data = brushed(), fill = "#9cf", color = '#036', weight = 1.5,
                opacity = 0.5, group = 'brushed', label = popup)
  })
  
})
```
