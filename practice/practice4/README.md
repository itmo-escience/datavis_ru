# PRACTICE 4

## Карты

Существует два типа карт на которых мы может визуализировать данные. Выбор того или иного типа зависит 
от необходимого нам уровня детализации:

- Карты данных - карты с низким уровнем детализации, которые зачастую включают только контуры географических регионов.

- Карты улиц - карты, содержащие несколько уровней детализации вплоть до зданий.

### GeoJSON

Большенство JavaScript-библиотек(включая `D3`) спроектированы для работы с пространственными данными в формате GeoJSON.
GeoJSON основан на JavaScript Object Notation (JSON). Он определяет несколько типов JSON-объектов и способы их комбинирования
для представления информации о пространственных объектах. Геометрия в GeoJSON представлена в виде точек или наборов точек.
Ниже
Пример GeoJSON-объекта:

```JavaScript
{
  "type": "FeatureCollection",
  "features": [
    {
      "type": "Feature",
      "geometry": {
        "type": "Point",
        "coordinates": [
          -80.87088507656375,
          35.21515162500578
        ]
      },
      "properties": {
        "name": "ABBOTT NEIGHBORHOOD PARK",
        "address": "1300  SPRUCE ST"
      }
    },
    {
      "type": "Feature",
      "geometry": {
        "type": "Polygon",
        "coordinates": [
          [
            [
              -80.72487831115721,
              35.26545403190955
            ],
            [
              -80.72135925292969,
              35.26727607954368
            ]
          ]
        ]
      },
      "properties": {
        "name": "Plaza Road Park"
      }
    }
  ]
}
```
[Тут](https://codepen.io/anon/pen/boKBXB?editors=0010) можно посмотреть примеры всех типов GeoJSON-объектов.

### Проекции

Координаты в GeoJSON (`[широта, долгота]`) - это сферические координаты. Поэтому, прежде всего нам нужно спроецировать их 
на нужную нам форму.

`D3` содержит набор наиболее распространенных проекций. Полный список проекций в `D3` можно посмотреть [тут](https://github.com/d3/d3-geo-projection#projections).

Пример скрипта для отрисовки геометрии в svg:
```JavaScript
const width = 800;
const height = 600;

const svg = d3.select("svg").attr('width', width).attr('height', height);

        const projection = d3.geoAlbersUsa()  // Проекция США, смещающая Гавайи и Аляску
            .translate([width / 2, height / 2]) // центрируем проекцию в центре svg
            .scale([700]);  // определяем зум

        // создаем geoPath-генератор с использованием проекции
        const path = d3.geoPath()
            .projection(projection);

        d3.json("https://raw.githubusercontent.com/avt00/dvcourse/master/us-states.json").then(function (json) {
            
            svg.append('g').selectAll("path")
                .data(json.features)
                .enter()
                .append("path")
                .attr("d", path); // передаем каждый объект в geoPath-генератор
            
            // Можно также добавить параллели и меридианы из встроенного d3.geoGraticule
            // let graticule = d3.geoGraticule();
            //d3.select("#mapLayer").append('path').datum(graticule).attr('class', "grat").attr('d', path).attr('fill', 'none');
        });
```
[Тут](https://codepen.io/anon/pen/MEXJQP?editors=1000) можно посмотреть что получилось.

Другие фигуры можно также привычным методом наносить на карту не забывая использовать проекцию:
```JavaScript
d3.csv("https://raw.githubusercontent.com/avt00/dvcourse/master/us-cities.csv", function (data) {
      svg.append('g').selectAll("circle")
          .data(data)
          .enter()
          .append("circle")
          .attr("cx", d =>  projection([d.lon, d.lat])[0] )
          .attr("cy", d => projection([d.lon, d.lat])[1] )
          .attr("r", d =>  Math.sqrt(parseInt(d.population) * 0.00004) )
          .style("fill", "steelblue")
          .style("opacity", 0.8);
});
```

### Карты улиц

`D3` не самый удобный инструмент для использования с Картами улиц. Существует несколько API, предоставляющих полный функционал для 
работы с Картами улиц, включая добавление слоев с данными. Наиболее популярные из них: [Mapbox](https://www.mapbox.com/maps/) и
[leaflet](https://leafletjs.com/).

Однако, при использовании `leaflet` существует способ наложить поверх `svg`-слой. В данном случая нужно использовать проекцию `leaflet` для 
преобразования координат:
```JavaScript
// создадим объект leaflet внутри #map
var map = new L.Map("map", {center: [37.8, -96.9], zoom: 4})
    .addLayer(new L.TileLayer("http://{s}.tile.openstreetmap.org/{z}/{x}/{y}.png"));
    
// добавим повер слой svg
const svg = d3.select(map.getPanes().overlayPane).append("svg")
const g = svg.append("g").attr("class", "leaflet-zoom-hide");

d3.json("us-states.json").then(collection) {

  // определим функцию для проекции leaflet
  function projectPoint(x, y) {
    var point = map.latLngToLayerPoint(new L.LatLng(y, x));
    this.stream.point(point.x, point.y);
  }
  
  // создадим генератор пути
  const transform = d3.geoTransform({point: projectPoint}),
  const path = d3.geoPath().projection(transform);
  
  // и отрисуем
  var feature = g.selectAll("path")
    .data(collection.features)
    .enter().append("path")
    .attr("d", path);
});
```

