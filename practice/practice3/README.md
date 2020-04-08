# ПРАКТИКА 3
## Шаблоны(Layouts)

`Layout` - это некая форма представления данных. `D3` включает всебя шаблоны, которые могут быть использованы для преобразования
содержащийся в данных информации в тот вид, в котором аттрибуты данных могут быть напрямую переданы в атрибуты `svg` элеентов для
формирования какого-либо из распространенных типов визуализаци.

### Hierarchy

[`d3.hierarchy`](https://observablehq.com/@d3/d3-hierarchy) - вложенная структура представляющая собой дерево каждый узел(`node`) имеет
родитея(за исключением корневого(`root`)), а также каждый узел имеет одного и более потомка(`child node`)(за исключением листьев(`leaves`)).
Каждый узел также хранит все данные в атрибуте `data`.

`d3.hierarchy` не может быть напрямую использован для визуализации, он лишь упрощает работу со вложенными структурами. 
Полученный в результате объект с данными может быть испоьзован в других шаблонах, таких как [`tidy tree`](https://observablehq.com/@d3/tidy-tree), [`treemap`](https://observablehq.com/@d3/treemap) or [`sunburst`](https://observablehq.com/@d3/sunburst).

Пример создания шаблона `hierarchy`:
```JavaScript
const data = {
  "name": "Eve",
  "children": [
    {
      "name": "Cain"
    },
    {
      "name": "Abel"
    },
    {
      "name": "Awan",
      "children": [
        {
          "name": "Enoch"
        }
      ]
    },
    {
      "name": "Azura"
    }
  ]
}
d3.hierarchy(data);
```
По умолчанию шаблон будет искать `d.children` в каждом узле. Для определения другого поля можно передать вторым аргументом функцию.

Данные из строчных форматов, таких как `csv` могут быть преобразованы в иерархический с помощью [`d3.stratify()`](https://observablehq.com/@d3/d3-stratify):
```JavaScript
var data = [
  {"name": "Eve",   "parent": ""},
  {"name": "Cain",  "parent": "Eve"},
  {"name": "Seth",  "parent": "Eve"},
  {"name": "Enos",  "parent": "Seth"},
  {"name": "Noam",  "parent": "Seth"},
  {"name": "Abel",  "parent": "Eve"},
  {"name": "Awan",  "parent": "Eve"},
  {"name": "Enoch", "parent": "Awan"},
  {"name": "Azura", "parent": "Eve"}
]
var root = d3.stratify()
    .id(function(d) { return d.name; })
    .parentId(function(d) { return d.parent; })
    (data);
console.log(root instanceof d3.hierarchy) // true
```

### Pie

[`d3.pie`](https://github.com/d3/d3-shape/blob/v1.3.5/README.md#pie) вычисляет атрибуты `startAngle` и `endAngel` для каждого элемента из массива данных.
```JavaScript
var data = [
  {"number":  4, "name": "Locke"},
  {"number":  8, "name": "Reyes"},
  {"number": 15, "name": "Ford"},
  {"number": 16, "name": "Jarrah"},
  {"number": 23, "name": "Shephard"},
  {"number": 42, "name": "Kwon"}
];

var arcs = d3.pie()
    .value(function(d) { return d.number; })
    (data);
// -> [{
  data:{
    name: "Locke",
    number: 4
  },
  endAngle: 6.283185307179586,
  index: 5,
  padAngle: 0,
  startAngle: 6.050474740247009,
  value: 4
}, ...
]
```
По умолчанию он формирует полную окружность, но шаблону также можно задать `startAngle` и `endAngel`.

### Pack

[`d3.pack`](https://github.com/d3/d3-hierarchy/blob/v1.1.8/README.md#pack) вычисляет `x`, `y` и `r` для каждого элемента в массиве данных
таким образом чтобы они заполнили определенное прямоуголное пространство.
```JavaScript
d3.pack()
    .size([width - 2, height - 2])
    .padding(3)
    .radius(d=>d.age)
    (employers)
```

### Force directed layout

`Force directed layout` - это динамический шаблон. Он обнавляет атрибуты каждый `tick` пока симуляция активна.
[Пример](http://bl.ocks.org/mbostock/31ce330646fa8bcb7289ff3b97aab3f5) использования `force directed layout`.

В `d3` `Force directed layout` представляет объект `d3.forceSimulation`. 
Симуляция контролируется параметром `alpha`, который стремится к `alphaMin` со скоростью `alphaDecay` в течении периода симуляции.
Все эти параметры могут быть заданы(в промежутке `[0,1]`).

В симуляцию могут быть включины ряд сил, которые будут влиять на повидение объектов:

- forceCenter - сила, приложенная к определенной точке
- forceCollide - сила, приложенная к каждому объекту с заданным радиусом
- forceManyBody - сила, приложенная к каждому объекту
- forceLink - сила, приложенная к связанным объектам
- forceX - сила, приложенная к определенной точки на оси x
- forceY - сила, приложенная к определенной точки на оси y
- Radial - сила, приложенная к определенной точке с заданным радиусом

Позиции объектов могут обновлятъся при событии `tick`:
```JavaScript
const simulation = d3.forceSimulation()
        .nodes(nodes)
        .force("charge", d3.forceManyBody())
        .force("center", d3.forceCenter(width / 2, height / 2));
        .on("tick", ticked);
        
 node = svg.append("g")
        .attr("class", "nodes")
    .selectAll("circle")
    .data(nodes)
    .enter().append("circle");
    
 function ticked() {
    node
        .attr("cx", function(d) { return d.x; })
        .attr("cy", function(d) { return d.y; });
}
```
## Генераторы

### Arc 

[`d3.arc`](https://github.com/d3/d3-shape/blob/v1.3.5/README.md#arcs) строит `svg path` основываясь на `startAngle` и `endAngle`.
```JavaScript
var data = [
  {"number":  4, "name": "Locke"},
  {"number":  8, "name": "Reyes"},
  {"number": 15, "name": "Ford"},
  {"number": 16, "name": "Jarrah"},
  {"number": 23, "name": "Shephard"},
  {"number": 42, "name": "Kwon"}
];

var pie = d3.pie().value(function(d){return d.number;});

var arc = d3.arc()
       .innerRadius(30) // it'll be donut chart
        .outerRadius(50)
        .padAngle(0.02)
        .cornerRadius(5);

svg.selectAll('path')
  .data(pie(data))
  .enter().append('path')
  .attr('d', arc) // каждый элемент будет передан в генератор
  .attr('fill', 'red')
  .style("opacity", 0.7);
```

### Line 
[`d3.line`](https://github.com/d3/d3-shape/blob/v1.3.5/README.md#lines) строит линию на входном массиве значений.

```JavaScript
const line = d3.line()
        .x(function(d) { return x(d.date) })
        .y(function(d) { return y(d.value) })
        (data);
svg.append("path")
      .attr("fill", "none")
      .attr("stroke", "steelblue") 
      .attr("stroke-width", 1.5)
      .attr("d", line)
```
### Area

[`d3.area`](https://github.com/d3/d3-shape/blob/v1.3.5/README.md#areas) строит `path` на основе четырех параметров.
```JavaScript
const area = d3.area()
        .x((d) => { return x(d.date) })
        .y0((d) => { return y(d.value1) })                        
        .y1((d) => { return y(d.value2) })
        (data)
svg.append("path")
      .attr("fill", "#cce5df")
      .attr("stroke", "#69b3a2")
      .attr("stroke-width", 1.5)
      .attr("d", area)
```
