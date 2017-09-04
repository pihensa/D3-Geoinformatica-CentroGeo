# Ejercicio práctico con D3

En este último apartdo queremos usar lo aprendido de D3 para hacer un mapa.

Vamos a retomar los datos históricos de los distritos electorales de 1994 a 2012.
Los puedes descargar de [aquí](https://github.com/CentroGeo/geoinformatica/tree/master/d3/shp).

Como te podrás dar cuenta, el `.shp` es de ~4.5 MB. Vamos a generalizarlo un poco para evitar tanto detalle
en la geometría de los distritos electorales. Ve a [mapshaper](http://mapshaper.org/) y arrastra los **4**
archivos del _shape_. Luego haz clic en _Simplify_, escoge cualquier método de simplificación y luego escoge
un porcentaje (por ejemplo, 75%). Luego haz clic en _Export_ y salva el resultado como **TopoJSON**.

Inmediatamente puedes ver que el TopoJSON es de alrededor de 1.5 MB. Un ahorro de ~3 veces el tamaño original.

**TAREA:** leer qué es un TopoJSON (que resulta ser que lo desarrolló el mismo Mike Bostock).

Ahora vamos a usar este nuevo archivo para hacer un primer mapa usando D3. Hay que incluir dos librerías de JS: `D3` y `topojson`.

```html
<!DOCTYPE html>
<head>
</head>
<body>
</body>
  <script src="https://d3js.org/d3.v4.min.js"></script>
  <script src="https://unpkg.com/topojson@3"></script>
  <script>
    var features;
	
    var width = 800,
        height = 600;
  
    var projection = d3.geoMercator()
                       .scale(1450)
                       .center([-97.16, 21.411])
                       .translate([width/1.5, height/1.75]);

    var svg = d3.select("body").append("svg")
                .attr("width", width)
                .attr("height", height);
		
    d3.json('elecciones.json', function(error, datos) {
        features = topojson.feature(datos, datos.objects.elecciones);
        svg.selectAll("path")
            .data(features.features)
            .enter().append("path")
            .attr("d", d3.geoPath().projection(projection));
    });
  </script>
</html>
```
Este primer mapa colorea todos los polígonos con un mismo color predeterminado. Para colorear este mapa
de acuerdo a una de las variables, por ejemplo, el porcentaje de votos del PRD en 1994, hay que modificar 
el código para leer ese dato en particular y definir estilos para nuestra clasificación de colores. 
Esto lo hacemos con la función `d3.scaleQuantize()`. 

```html
<!DOCTYPE html>
<head>
<style>
/* Los colores para las clases */
  .q0 { fill:#fcc383; }
  .q1 { fill:#fc9f67; }
  .q2 { fill:#f4794e; }
  .q3 { fill:#e65338; }
  .q4 { fill:#ce2a1d; }
  .q5 { fill:#b30000; }
</style>
</head>
<body>
</body>
  <script src="https://d3js.org/d3.v4.min.js"></script>
  <script src="https://unpkg.com/topojson@3"></script>
  <script>
    var features;
    
    var width = 800,
        height = 600;
  
    var projection = d3.geoMercator()
                       .scale(1450)
                       .center([-97.16, 21.411])
                       .translate([width/1.5, height/1.75]);

    var svg = d3.select("body").append("svg")
                .attr("width", width)
                .attr("height", height);
        
    d3.json('elecciones.json', function(error, datos) {
        features = topojson.feature(datos, datos.objects.elecciones);
        
        var quantize = d3.scaleQuantize()
                         .domain([0, 52])
                         .range(d3.range(6).map(function(i) { return "q" + i; }));
                     
         svg.selectAll("path")
            .data(features.features)
            .enter().append("path")
            .attr("d", d3.geoPath().projection(projection))
            .attr("class", function(d){ return quantize(d.properties['PRD94']) } );
    });
  </script>
```

Pero qué pasa si queremos escoger pintar el mapa con otra variable? Es claro que ese valor de `max` que usamos
va a cambiar dependiendo de la variabele que se use... Vamos a escoger una variable y leer su máximo directamente 
de los datos dentro del callback.
```html
<!DOCTYPE html>
<head>
<style>
/*Los colores para las clases de poblaciÃ³n*/
  .q0 { fill:#fcc383; }
  .q1 { fill:#fc9f67; }
  .q2 { fill:#f4794e; }
  .q3 { fill:#e65338; }
  .q4 { fill:#ce2a1d; }
  .q5 { fill:#b30000; }
</style>
</head>
<body>
</body>
  <script src="https://d3js.org/d3.v4.min.js"></script>
  <script src="https://unpkg.com/topojson@3"></script>
  <script>
    var features;
    
    var width = 800,
        height = 600;
  
    var projection = d3.geoMercator()
                       .scale(1450)
                       .center([-97.16, 21.411])
                       .translate([width/1.5, height/1.75]);

    var svg = d3.select("body").append("svg")
                .attr("width", width)
                .attr("height", height);
        
    d3.json('elecciones.json', function(error, datos) {
        features = topojson.feature(datos, datos.objects.elecciones);
        var interes = 'PRD94';
        var max = d3.max(features.features, function(d) { return d.properties[interes]; })
        
        var quantize = d3.scaleQuantize()
                         .domain([0, max])
                         .range(d3.range(6).map(function(i) { return "q" + i; }));
                     
         svg.selectAll("path")
            .data(features.features)
            .enter().append("path")
            .attr("d", d3.geoPath().projection(projection))
            .attr("class", function(d){ return quantize(d.properties[interes]) } );
    });
  </script>
</html>
```

¿Y si queremos cambiar de manera interactiva la variable? Vamos a agregar un control al mapa que nos permita
seleccionarla. Esto lo podríamos hacer con un `<select>`o con un `<input type="radio">`.
Vamos a agregarle un `<select>` al DOM justo antes de agregar el `svg`. Para que se vea mejor, vamos a definir en
donde queremos que salga en la pantalla. Para esto, el elemeto agregado tiene una clase `variable`, cuyo estilo
está definido dentro de `<style>`.
```javascript
var select = d3.select("body")
               .append("select")
               .attr("class", "variable");
```
Hay que agregar las distintas variables que queremos poner en ese elemento de selección. Lo más facil es leer
los datos y ver qué columnas tienen y agregarlas. Para eso, dentro del _callback_ agregamos lo siguiente:
```javascript
var opciones = d3.keys(features.features[0].properties);
select.selectAll("option")
      .data(opciones)
      .enter()
      .append("option")
      .attr("value", function (d) { return d; })
      .property("selected", function(d){ return d === 'PRD94'; })
      .text(function (d) { return d; });
```
Aquí leemos las variables del primer elemento de los del JSON (puesto que todos traen las mismas columnas) con la
función `d3.keys()`. Luego agarramos el elemento `<select>` del DOM y le agregamos etiquetas de `<option>`. Hacemos 
una unión de los datos de las opciones, hacemos la selección  `enter()` y le decimos al DOM que agregue un atributo
`value`, que selecciones el que se llame `PRD94` y que le ponga un texto.

Hay que definir qué queremos que pase cuando se cambie la selección de esa variable. Para esto, usamos los 
populares _event listeners_ de JavaScript: cuando **este evento** ocurra, ejecuta _esta función_.
```javascript
select.on("change", function(d) {
    interes = d3.select(this).property("value");
    d3.select("svg").selectAll("path").remove();
    hazMapa(interes);
});
```
Aquí estamos diciendo que cuando _cambie_ la selección del `<select>`, a la variable `interes` le asignamos lo
que acabamos de seleccionar, "limpiamos" el mapa y ejecutamos la función que hace el mapa, que es la misma que
usamos anteriormente, pero como ahora estamos haciendo varios mapas, es más eficiente llamar todas esas instrucciones
como una función:
```javascript
function hazMapa(interes){

    var max = d3.max(features.features, function(d) { return d.properties[interes]; })

    var quantize = d3.scaleQuantize()
                     .domain([0, max])
                     .range(d3.range(6).map(function(i) { return "q" + i; }));
                     
    svg.selectAll("path")
       .data(features.features)
       .enter().append("path")
       .attr("d", d3.geoPath().projection(projection))
       .attr("class", function(d){ return quantize(d.properties[interes]) } );
}
```

Y para que al inciar la página haya un mapa, escribimos:
```javascript
var interes = d3.select('select').property("value");
hazMapa(interes);
```
después de haber creado las entradas del elemento `<select>`. Aquí vemos qué variable está seleccionada (`PRD94`)
y hacemos el mapa con esta variable.


Grafica con esas variables que cambie con la interactividad
Grafica que muestre promedio
Al hacer clic en estado que se actualiced la grafica con los datos de es estado

Regresar a [Selecciones, Joins y General Update Pattern](d3_selecciones.md)