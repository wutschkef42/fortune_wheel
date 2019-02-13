<template>
  <div id="app">
    <div id="chart"></div>
    <div id="entry_info"><h1></h1></div>
    <div id="entry_img"></div>
  </div>
</template>

<script>

export default {
  name: 'app',
  mounted: function() {
    var padding = {top:20, right:40, bottom:0, left:0},
        w = 500 - padding.left - padding.right,
        h = 500 - padding.top  - padding.bottom,
        r = Math.min(w, h)/2,
        rotation = 0,
        oldrotation = 0,
        picked = 100000,
        oldpick = [],
        color = d3.scale.category20();

      var data = [
          {label: 'Eau de Cologne', category: 'citrus', color: '#ffef11', description: 'Eau de Cologne - Citrus', img_src: './assets/stub.png', value: 1},
          {label: 'Bel Resmro', category: 'green', color: '#31c44e', description: 'Bel Resmo - Floral Green', img_src: './assets/stub.png', value: 1},
          {label: 'La Pausa', category: 'green', color: '#31c44e', description: 'La Pausa - Floral Woody Powdery', img_src: './assets/stub.png', value: 1},
          {label: 'No 22', category: 'floral', color: '#dd33c9', description: 'No 22 - Floral Powdery', img_src: './assets/stub.png', value: 1},
          {label: 'Gardenra', category: 'floral', color: '#dd33c9', description: 'Gardenra - Floral', img_src: './assets/stub.png', value: 1},
          {label: 'Beige', category: 'floral', color: '#dd33c9', description: 'Beige - Floral Powdery', img_src: './assets/stub.png', value: 1},
          {label: 'Jersey', category: 'aromatic', color: '#b033dd', description: 'Jersey - Aromatic Floral Powdery', img_src: './assets/stub.png', value: 1},
          {label: 'Boy Chanel', category: 'aromatic', color: '#b033dd', description: 'Boy Chanel - Fougère Oriental', img_src: './assets/stub.png', value: 1},
          {label: 'Bois des Res', category: 'woody', color: '#b57601', description: 'Bois des Res - Woody Oriental', img_src: './assets/stub.png', value: 1},
          {label: '31 Rue Cambon', category: 'woody', color: '#b57601', description: '31 Rue Cambon - Chypre', img_src: './assets/stub.png', value: 1},
          {label: 'No 18', category: 'woody', color: '#b57601', description: 'No 18 - Woody Fruity Oriental', img_src: './assets/stub.png', value: 1},
          {label: 'Sycomore', category: 'woody', color: '#b57601', description: 'Sycomore - Woody', img_src: './assets/stub.png', value: 1},
          {label: '1932', category: 'woody', color: '#b57601', description: '1932 - Woody Floral Fruity', img_src: './assets/stub.png', value: 1},
          {label: 'Misia', category: 'oriental', color: '#ff9811', description: 'Misia - Soft Oriental Floral Rosy', img_src: './assets/stub.png', value: 1},
          {label: 'Coromandel', category: 'oriental', color: '#ff9811', description: 'Coromandel - Oriental Woody', img_src: './assets/stub.png', value: 1},
          {label: 'Cuire De Russie', category: 'oriental', color: '#ff9811', description: 'Cuire de Russie - Oriental Leather', img_src: './assets/stub.png', value: 1},
          {label: '1957', category: 'oriental', color: '#ff9811', description: '1957 - Soft Oriental', img_src: './assets/stub.png', value: 1},
      ];
      
      var svg = d3.select('#chart')
          .append("svg")
          .data([data])
          .attr("width",  w + padding.left + padding.right)
          .attr("height", h + padding.top + padding.bottom)
            
      var container = svg.append("g")
          .attr("class", "chartholder")
          .attr("transform", "translate(" + (w/2 + padding.left) + "," + (h/2 + padding.top) + ")");

      var vis = container
          .append("g");
          
      var pie = d3.layout.pie().sort(null).value(function(d){return 1;});

      var arc = d3.svg.arc()
          .outerRadius(r)
          .padAngle(.04)
          .padRadius(80)
                      
      var arcs = vis.selectAll("g.slice")
          .data(pie)
          .enter()
          .append("g")
          .attr("class", "slice");
          
      arcs.append("path")
          .attr("fill", function(d, i){ console.log(data[i]);return data[i].color; })
          .attr("d", function (d) { return arc(d); });

      arcs.append("text").attr("transform", function(d){
              d.innerRadius = 0;
              d.outerRadius = r;
              d.angle = (d.startAngle + d.endAngle)/2;
              return "rotate(" + (d.angle * 180 / Math.PI - 90) + ")translate(" + (d.outerRadius -10) +")";
          })
          .attr("text-anchor", "end")
          .text( function(d, i) {
              return data[i].label;
          });

      container.on("click", spin);

      function spin(d){     
          container.on("click", null);
          //all slices have been seen, all done
          // console.log("OldPick: " + oldpick.length, "Data length: " + data.length);
          if(oldpick.length == data.length){
              container.on("click", null);
              return;
          }

          var  ps = 360/data.length,
                pieslice = Math.round(1440/data.length),
                rng = Math.floor((Math.random() * 1440) + 360);
              
          rotation = (Math.round(rng / ps) * ps);          
          picked = Math.round(data.length - (rotation % 360)/ps);
          picked = picked >= data.length ? (picked % data.length) : picked;

          if(oldpick.indexOf(picked) !== -1){
              d3.select(this).call(spin);
              return;
          } else {
              oldpick.push(picked);
          }

          rotation += 90 - Math.round(ps/2);

          vis.transition()
              .duration(3000)
              .attrTween("transform", rotTween)
              .each("end", function(){

                  //mark entry_info as seen
                  d3.select(".slice:nth-child(" + (picked + 1) + ") path")
                      .attr("fill",'#111');

                  //populate entry_info
                  d3.select("#entry_info h1")
                      .text(data[picked].description);
                  
                      d3.select('#chart').append('svg').attr('id', 'entry_image')
                      .append('image')
                      .attr('xlink:href', data[picked].img_src)
                      .attr('width', 300)
                      .attr('height', 150)

                  oldrotation = rotation;
                  container.on("click", spin);
              });
      }

      //make arrow
      svg.append("g")
          .attr("transform", "translate(" + (w + padding.left + padding.right) + "," + ((h/2)+padding.top) + ")")
          .append("path")
          .attr("d", "M-" + (r*.15) + ",0L0," + (r*.05) + "L0,-" + (r*.05) + "Z")
          .style({"fill":"black"});

      //draw spin circle
      container.append("circle")
          .attr("cx", 0)
          .attr("cy", 0)
          .attr("r", 60)
          .style({"fill":"white","cursor":"pointer"});

      //spin text
      container.append("text")
          .attr("x", 0)
          .attr("y", 15)
          .attr("text-anchor", "middle")
          .text("SPIN")
          .style({"font-weight":"bold", "font-size":"30px"});
      
      function rotTween(to) {
        var i = d3.interpolate(oldrotation % 360, rotation);
        return function(t) {
          return "rotate(" + i(t) + ")";
        };
      }
      
      function getRandomNumbers(){
          var array = new Uint16Array(1000);
          var scale = d3.scale.linear().range([360, 1440]).domain([0, 100000]);

          if(window.hasOwnProperty("crypto") && typeof window.crypto.getRandomValues === "function"){
              window.crypto.getRandomValues(array);
          } else {
              for(var i=0; i < 1000; i++){
                  array[i] = Math.floor(Math.random() * 100000) + 1;
              }
          }
          return array;
      }
  },
}
</script>

<style>
#app {
  font-family: 'Avenir', Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
 text{
        font-family:Helvetica, Arial, sans-serif;
        font-size:16px;
        color: #fff !important;
        pointer-events:none;
    }
    #chart{
        position:absolute;
        width:500px;
        height:500px;
        top:0;
        left:0;
    }
    #entry_info{
        position: absolute;
        width:400px;
        height:500px;
        top:20px;
        left:620px;
    }
    #entry_info h1{
        font-size: 28px;
        font-weight: bold;
        font-family: "Helvetica Neue", Helvetica, Arial, sans-serif;
        position: absolute;
        padding: 0;
        margin: 0;
        top:50%;
        -webkit-transform:translate(0,-50%);
                transform:translate(0,-50%);
    }

    #entry_image {
        position: absolute;
        top: 190px;
        left: 420px;
        z-index: 100;
    }
</style>
