# Trabajo final Taller de Geoinformática
### Diana Fabián, Tania Gómez y Mario González ###

Para comenzar el trabajo se creó un archivo .shp propio, con base en los datos descargados de la Secretaría de Turismo 
referentes a ocupación y cuartos disponibles en los destinos más importantes del país. Para esto se trabajó con la información del 
INEGI que publica las áreas geoestadísticas municipales, creando los distintos polígonos para los destinos de nuestra base de datps. 
Ya que generamos el shape que pesaba 7.5 MB, procedimos a hacer el TopoJSON de 185 KB, con lo que se ahorró bastante espacio.

Ya en el html se incluyeron las librerías D3 y topojson. Cargamos los datos, definimos la proyección y el espacio de trabajo donde 
irán el mapa, los títulos y las gráficas con las siguientes líneas de código:

```

    <script src="https://d3js.org/d3.v4.min.js"></script>
    <script src="https://unpkg.com/topojson@3"></script>

    <div id="info"><h1 id="name"></h1></div>
    <div id="info2"><h4 id="edo"></h4></div>
    <div id="info3"><h4 id="tipo"></h4></div>
    <script>
      var features;

      var width = 850,
          height = 650;

      var projection = d3.geoMercator()
                         .scale(1400)
                         .center([-102.584065, 23.62755])
                         .translate([width/2, height/2]);

      var radio= d3.selectAll('input[name="level"]:checked').node().value;



      var svg = d3.select("body").append("svg")
                  .attr("width", width)
                  .attr("height", height);

      var g = svg.append("g")
                 .attr("id", "estados");


      var barSvg = d3.select("body").append("svg")
                     .attr("id", "bars")
                     .attr("height", 570)
                     .attr("width", 400);

      d3.json('union_3.json', function(error, datos) {

        features = topojson.feature(datos, datos.objects.union_3);

        g.selectAll("path")
           .data(features.features)
           .enter().append("path")
           .attr("d", d3.geoPath().projection(projection))
           .attr('class','states')

```

Con la variable ‘projection’ definimos la proyección, en la variable ‘radio’ se definen los botones que van a usarse para 
seleccionar el tipo de destino, en la variable ‘svg’ se define el espacio donde dibujaremos el mapa, en la variable ‘barSvg’ 
se define el espacio donde se van a dibujar las gráficas y finalmente se le agregan los datos del topoJSON con la variable ‘features’.

El mapa lo coloreamos de dos maneras: una vez que se coloque el cursor encima de alguno de los polígonos, este se pondrá de color 
rojo, desplegando en la parte de la izquierda la información sobre el mismo: el nombre del destino, el estado en el que se encuentra 
y el tipo de destino al que pertenece. Este es un event listener, el cual se llama mouseover (y mouseout para cuando el cursor se 
mueva fuera del polígono).

```
        g.selectAll("path")
           .data(features.features)
           .enter().append("path")
           .attr("d", d3.geoPath().projection(projection))
           .attr('class','states')

           .on('mouseover', function(d){ if (d.properties.Destino!=='Mexico'){
                d3.select(this).classed('selected',true);
                 var name = d.properties.Destino;
                 var edo = d.properties.ESTADO;
                 var tipo = d.properties.TIPO;

			           document.getElementById('name').innerHTML=name,
                 document.getElementById('edo').innerHTML=edo,
                 document.getElementById('tipo').innerHTML=tipo;}else {
                       var name ='';
                       var edo = '';
                       var tipo = '';
      			           document.getElementById('name').innerHTML=name,
                       document.getElementById('edo').innerHTML=edo,
                       document.getElementById('tipo').innerHTML=tipo;}
		             })

            .on('mouseout', function(d){ if (d.properties.Destino!=='Mexico'){
                  d3.select(this).classed('selected',false);}
                });
```

La otra forma en la que se colorea el mapa es cuando se selecciona alguno de los 4 distintos tipos de destino. 
De esta manera, se colorearán todos los destinos que pertenezcan a ese tipo, habiendo definido un color para cada uno. 
Por lo tanto, definimos en nuestro dominio el tipo de destino: “centros de playa, fronteriza, ciudades interior y grandes ciudades” 
y como rango el color que le corresponderá a cada tipo respectivamente: “azul, ocre, verde y morado”.

```
        function filtro() {
          var radio =d3.selectAll('input[name="level"]:checked').node().value;
          var anho = d3.select('#datos').node().value+d3.select("#anho").node().value;
          var data=features.features;
          var datos= data.filter(function(d){return d.properties.TIPO ==radio});//}



          var  acomodados= datos.map(function(elemento){return elemento.properties});


          hazBarras(anho, acomodados);
      //    resaltarTipo(radio,datos);
          var colores =d3.scaleOrdinal()
                         .domain(['CENTROS DE PLAYA','FRONTERIZA','CIUDADES INTERIOR','GRANDES CIUDADES'])
                         .range([ "#86fbff", "#b78916","#1d9b4d", "#7660ba"]);

          d3.selectAll('path')
            .style('fill', function(d){
              var radio=d3.selectAll('input[name="level"]:checked').node().value;
          //    console.log(radio);
                 if (d.properties.TIPO===radio) {return colores(radio) ;}});
```

Posteriormente agregamos las barras que están ligadas a los datos. La idea es que se despliegue la información de acuerdo al 
tipo de destino, a los datos (ya sean habitaciones disponibles, habitaciones ocupadas o porcentaje de ocupación) y al año que 
se desee. De esta manera tendremos 48 posibles visualizaciones de las gráficas: 4 distintos años, 4 tipos de destinos y 3 datos 
a obtener.

Entonces, definimos la función ‘hazBarras’, en la que se recorrerá de acuerdo al tipo de destino para desplegar la información 
de este únicamente. Los colores van a estar ligados a los que aparecen en el mapa con los botones de selección, y la información 
que se mostrará será el nombre del destino con el valor que tiene para el tipo de dato que está seleccionado.

```
      function hazBarras(anho, acomodados){
        var destino = []

      //  console.log(anho);

        for(x in acomodados) destino.push(acomodados[x].Destino);
        var tipo= acomodados[0].TIPO;
      //  console.log(destino);
        var colores = {"CENTROS DE PLAYA": "#86fbff", "FRONTERIZA": "#b78916", "CIUDADES INTERIOR": "#1d9b4d", "GRANDES CIUDADES": "#7660ba"};
        var datos = [];

        for (i = 0; i < destino.length ; i++){
          c = {};
          c["nombre"] = destino[i];
          c["valor"] = acomodados[i][anho];
          c["tipo"] =  tipo;
          datos.push(c);
        }

```

Para desplegar la información correctamente, es necesario calcular el máximo de cada tipo de dato para que la información se 
ajuste al tamaño definido para mostrarlo en la gráfica. Esto pasa porque tenemos un tipo de dato que muestra porcentajes, 
por lo que el máximo que podría mostrar es 100, mientras que los otros dos tipos de datos muestran número de habitaciones 
(ocupadas o disponibles), siento el valor más alto un dato que está alrededor de los 50,000. Para esto definimos nuestra 
variable ‘maximo’, y así el dominio irá del inicio al valor máximo para cada tipo de dato, con sus rangos ajustados al ancho 
de la barra. Pegamos estos datos al espacio destinado para ello en la variable ‘barSvg’ y se actualizarán entonces las barras 
de acuerdo al tipo de dato seleccionado.

```
        var barWidth = 400,
            barHeight = 15;
            // calcular el maximo de los datos que se van a graficar
        var maximo = d3.max([d3.max(acomodados,function(d){return d[anho] })]);
        var f = d3.format(".0f"); // fomrato para redondear los numeros
        //console.log(max);
        var x = d3.scaleLinear()
                  .range([0, barWidth])
                //  .domain([0, 100]);
                // re - escalar la proporcion de pixeles de acuerdo al maximo del dato que se va a graficar
                // 400 px corresponde al maximo
                  .domain([0, maximo]);

        bar = barSvg.selectAll(".bar")
                    .data(datos, function(d){ return d.nombre;});

        var barEnter = bar.enter()
            .append("g")
            .attr("class", "bar")
            .attr("transform", function(d, i) { return "translate(0," + i * barHeight + ")"; })

```

Sólo queda pegarle la información a las barras correspondiente a los nombres de destinos y el valor que corresponda para cada 
tipo de dato que se seleccione, así como los parámetros para las transiciones cuando se actualice la selección y la actualización 
de la información.

```
        barEnter.append("rect")
            .transition()
            .duration(500)
            .attr("width", function(d) {return x(d.valor);})
            .attr("height", barHeight - 1)
            .attr("fill", function(d) { return colores[d.tipo]; });

        barEnter.append("text")
                .transition()
                .duration(500)
                .attr("x",360)// alinear el valor en la posicion x=360px de los 400px
                .attr("y", barHeight / 2)
                .attr("dy", ".35em")
                .text(function(d) { return f(d.valor) });

        barEnter.append("text")
                .transition()
                .duration(500)
                .attr("x", 10 )
                .attr("y", barHeight / 2)
                .attr("dy", ".35em")
                .text(function(d) { return d.nombre });

        bar.select("rect")
           .transition()
           .duration(500)
           .attr("width", function(d) { return x(d.valor); });

        bar.select("text")
           .transition()
           .duration(500)
           .attr("x",360)// alinear el valor en la posicion x=360px de los 400px
           .text(function(d) {return f(d.valor)});

        barEnter.exit().remove();
        bar.exit().remove();
  }

```

Con las últimas dos líneas se logra remover la gráfica que se muestra una vez que se selecciona otro tipo de dato, para que 
entre la nueva información.

El código final queda de la siguiente manera:

```
<!DOCTYPE html>
<html>
  <meta charset='UTF-8'>
  <head>
    <title>Proyecto d3</title>
    <style>
    /*========================================================================*/
    /*                  FORMATO DE CONTROLES DEL FORMULARIO                   */
    /*========================================================================*/
      #encabezado{
        position: absolute;
        top: 0px;
        left: 70px;
        font:2em arial;
      }

      #titulo1{
        position: absolute;
        top: 50px;
        left: 50px;
        font:16px arial;
        color: steelblue;
      }
      #centros{
        position: absolute;
        top: 65px;
        left: 170px;
        font:13px arial;
      }
      #titulo2{
        position: absolute;
        top: 50px;
        left: 720px;
        font:16px arial;
        color: steelblue;
      }
      #datos{
        position: absolute;
        top: 65px;
        left: 770px;
      }
      #titulo3{
        position: absolute;
        top: 50px;
        left: 970px;
        font:16px arial;
        color: steelblue;
      }
      #anho{
        position: absolute;
        top: 65px;
        left: 1010px;
      }
      #titulo4{
        position: absolute;
        top: 330px;
        left: 10px;
        font:16px arial;
      }
      #about{
        position: absolute;
        top: 5px;
        left: 1000px;
        font:10px arial;
      }
      #imagen{
        position: absolute;
        top: 5px;
        left: 8px;
      }
      /*========================================================================*/
      /*                  FORMATO DE COlORES DEL MAPA                   */
      /*========================================================================*/
      h1 {
        font-family:arial;
        font-size:2em;
        color:#333;
      }
      #info {
        position:absolute;
        top: 470px;
        left: 10px;
      }

      .states {
	       fill:  #e5e5e5;/*#ccc2c3 ;/*#e5e5e5;*/
	       stroke: #9a9d9e;/*#fffeff/*#681107;*/
	       stroke-width:.5px;
       }
       .selected {
	        fill: #ff0303;/*#681107;*/
          stroke: #ff0303;/*#681107;*/
          stroke-width:2px;
        }
        #info2 {
          position:absolute;
          top: 540px;
          left: 10px;
          font:12px arial;
        }
        #info3 {
          position:absolute;
          top: 570px;
          left: 10px;
          font:12px arial;
        }

      }


        /*========================================================================*/
        /*                  FORMATO DE GRAFICA LAS BARRAS                    */
        /*========================================================================*/

        #bars {
          position: absolute;
          margin-top: 100px;
          margin-left: 10px;
        }

        .bar text {
          fill: black;
          font: 10px arial;
          text-anchor: right;
        }


    </style>
  </head>
  <body>
    <!--========================================================================-->
    <!--                  FORMATO DE CONTROLES DEL FORMULARIO                   -->
    <!--========================================================================-->

    <h1 id='encabezado'>Destinos más visitados de la República Mexicana</h1><br /> <!--titulo-->
    <h3 id='titulo1'>Tipo de Destino:</h3>
    <form id='centros' class="centros">
      <input type="radio" name="level" value="CENTROS DE PLAYA" onchange=filtro() checked="checked">Centros de Playa</input>
      <input type="radio" name="level" value="FRONTERIZA" onchange=filtro()>Ciudad Fronteriza</input>
      <input type="radio" name="level" value="CIUDADES INTERIOR" onchange=filtro()>Ciudades del Interior</input>
      <input type="radio" name="level" value="GRANDES CIUDADES" onchange=filtro()>Grandes Ciudades</input>
    </form>
    <h3 id='titulo2'>Datos:</h3>
    <select id="datos" class="select" onchange=filtro()>
      <option value="NCDP" selected=true>Habitaciones Disponibles</option>
      <option value="NCOP">Habitaciones Ocupadas</option>
      <option value="POHP">Porcentaje de Ocupación</option>
    </select>
    <h3 id='titulo3'>Año:</h3>
    <select id="anho", class="select" onchange=filtro()>
      <option value="2013">2013</option>
      <option value="2014">2014</option>
      <option value="2015">2015</option>
      <option value="2016">2016</option>
    </select>
    <h3 id='titulo4'>Información:</h3>
    <img id='imagen' src='cactus.jpg' alt="cactus feliz" style="width:48px;height:48px;">
    <a id='about' href="about.html">Acerca de</a>

    <script src="https://d3js.org/d3.v4.min.js"></script>
    <script src="https://unpkg.com/topojson@3"></script>

    <div id="info"><h1 id="name"></h1></div>
    <div id="info2"><h4 id="edo"></h4></div>
    <div id="info3"><h4 id="tipo"></h4></div>
    <script>
      var features;

      var width = 850,
          height = 650;

      var projection = d3.geoMercator()
                         .scale(1400)
                         .center([-102.584065, 23.62755])
                         .translate([width/2, height/2]);

      var radio= d3.selectAll('input[name="level"]:checked').node().value;



      var svg = d3.select("body").append("svg")
                  .attr("width", width)
                  .attr("height", height);

      var g = svg.append("g")
                 .attr("id", "estados");


      var barSvg = d3.select("body").append("svg")
                     .attr("id", "bars")
                     .attr("height", 570)
                     .attr("width", 400);

      d3.json('union_3.json', function(error, datos) {

        features = topojson.feature(datos, datos.objects.union_3);

        g.selectAll("path")
           .data(features.features)
           .enter().append("path")
           .attr("d", d3.geoPath().projection(projection))
           .attr('class','states')

           .on('mouseover', function(d){ if (d.properties.Destino!=='Mexico'){
                d3.select(this).classed('selected',true);
                 var name = d.properties.Destino;
                 var edo = d.properties.ESTADO;
                 var tipo = d.properties.TIPO;

			           document.getElementById('name').innerHTML=name,
                 document.getElementById('edo').innerHTML=edo,
                 document.getElementById('tipo').innerHTML=tipo;}else {
                       var name ='';
                       var edo = '';
                       var tipo = '';
      			           document.getElementById('name').innerHTML=name,
                       document.getElementById('edo').innerHTML=edo,
                       document.getElementById('tipo').innerHTML=tipo;}
		             })

            .on('mouseout', function(d){ if (d.properties.Destino!=='Mexico'){
                  d3.select(this).classed('selected',false);}
                });



        });

        function filtro() {
          var radio =d3.selectAll('input[name="level"]:checked').node().value;
          var anho = d3.select('#datos').node().value+d3.select("#anho").node().value;
          var data=features.features;
          var datos= data.filter(function(d){return d.properties.TIPO ==radio});//}



          var  acomodados= datos.map(function(elemento){return elemento.properties});


          hazBarras(anho, acomodados);
      //    resaltarTipo(radio,datos);
          var colores =d3.scaleOrdinal()
                         .domain(['CENTROS DE PLAYA','FRONTERIZA','CIUDADES INTERIOR','GRANDES CIUDADES'])
                         .range([ "#86fbff", "#b78916","#1d9b4d", "#7660ba"]);

          d3.selectAll('path')
            .style('fill', function(d){
              var radio=d3.selectAll('input[name="level"]:checked').node().value;
          //    console.log(radio);
                 if (d.properties.TIPO===radio) {return colores(radio) ;}});

          //var max = d3.max([d3.max(acomodados,function(d){return d[anho] })]);
          //alert(max);


}



      function hazBarras(anho, acomodados){
        var destino = []

      //  console.log(anho);

        for(x in acomodados) destino.push(acomodados[x].Destino);
        var tipo= acomodados[0].TIPO;
      //  console.log(destino);
        var colores = {"CENTROS DE PLAYA": "#86fbff", "FRONTERIZA": "#b78916", "CIUDADES INTERIOR": "#1d9b4d", "GRANDES CIUDADES": "#7660ba"};
        var datos = [];

        for (i = 0; i < destino.length ; i++){
          c = {};
          c["nombre"] = destino[i];
          c["valor"] = acomodados[i][anho];
          c["tipo"] =  tipo;
          datos.push(c);
        }


        var barWidth = 400,
            barHeight = 15;
            // calcular el maximo de los datos que se van a graficar
        var maximo = d3.max([d3.max(acomodados,function(d){return d[anho] })]);
        var f = d3.format(".0f"); // fomrato para redondear los numeros
        //console.log(max);
        var x = d3.scaleLinear()
                  .range([0, barWidth])
                //  .domain([0, 100]);
                // re - escalar la proporcion de pixeles de acuerdo al maximo del dato que se va a graficar
                // 400 px corresponde al maximo
                  .domain([0, maximo]);

        bar = barSvg.selectAll(".bar")
                    .data(datos, function(d){ return d.nombre;});

        var barEnter = bar.enter()
            .append("g")
            .attr("class", "bar")
            .attr("transform", function(d, i) { return "translate(0," + i * barHeight + ")"; })

    //    d3.select("#titulo").html("Estado:" + distrito[0].key.substr(0,2) + " - Distrito:" + distrito[0].key.substr(3));

        barEnter.append("rect")
            .transition()
            .duration(500)
            .attr("width", function(d) {return x(d.valor);})
            .attr("height", barHeight - 1)
            .attr("fill", function(d) { return colores[d.tipo]; });

        barEnter.append("text")
                .transition()
                .duration(500)
                .attr("x",360)// alinear el valor en la posicion x=360px de los 400px
                .attr("y", barHeight / 2)
                .attr("dy", ".35em")
                .text(function(d) { return f(d.valor) });

        barEnter.append("text")
                .transition()
                .duration(500)
                .attr("x", 10 )
                .attr("y", barHeight / 2)
                .attr("dy", ".35em")
                .text(function(d) { return d.nombre });

        bar.select("rect")
           .transition()
           .duration(500)
           .attr("width", function(d) { return x(d.valor); });

        bar.select("text")
           .transition()
           .duration(500)
           .attr("x",360)// alinear el valor en la posicion x=360px de los 400px
           .text(function(d) {return f(d.valor)});

        barEnter.exit().remove();
        bar.exit().remove();
  }

    </script>
  </body>
</html>
```
