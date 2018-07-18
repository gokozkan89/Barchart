/*<!DOCTYPE html>
<meta charset="utf-8">
<style>
  svg {
    font: 10px sans-serif;
  }
.subBar {
    fill: gray;
    opacity: 0.5;
}
.bar {
  fill: orange;
}
.bar:hover {
  fill: orangered ;
}
.bar2{
  opacity: 0.3;
}
  .axis path,
  .axis line {
    fill: none;
    stroke: #000;
    shape-rendering: crispEdges;
  }

  .brush .extent {
    stroke: #fff;
    fill: steelblue;
    fill-opacity: .25;
    shape-rendering: crispEdges;
  }
  .d3-tip {
  line-height: 1;
  font-weight: bold;
  padding: 12px;
  background: rgba(0, 0, 0, 0.8);
  color: #fff;
  border-radius: 2px;
}

/* Creates a small triangle extender for the tooltip */
.d3-tip:after {
  box-sizing: border-box;
  display: inline;
  font-size: 10px;
  width: 100%;
  line-height: 1;
  color: rgba(0, 0, 0, 0.8);
  content: "\25BC";
  position: absolute;
  text-align: center;
}

/* Style northward tooltips differently */
.d3-tip.n:after {
  margin: -1px 0 0 0;
  top: 100%;
  left: 0;
}
</style>

<body>
  <script src="http://d3js.org/d3.v5.min.js"></script>
  <script src="https://cdnjs.cloudflare.com/ajax/libs/d3-tip/0.9.1/d3-tip.js"></script>
  <script>

    var data = [];

    for (var i = 0; i < 500; i++) {

      data[i] = Math.floor(Math.random() * 600);

    }

    var margin = { top: 20, right: 20, bottom: 90, left: 50 },
      margin2 = { top: 230, right: 20, bottom: 30, left: 50 },
      width = 600 - margin.left - margin.right,
      height = 300 - margin.top - margin.bottom,
      height2 = 300 - margin2.top - margin2.bottom;
      var tip = d3.tip()
      .attr('class', 'd3-tip')
      .offset([-10, 0])
      .html(function(d) {
        return "<strong>Price:</strong> <span style='color:red'>" + d + "</span>";
      })


    var svg = d3.select("body").append("svg")
      .attr("width", width + margin.left + margin.right)
      .attr("height", height + margin.top + margin.bottom);

    var focus = svg.append("g")
      .attr("transform", "translate(" + margin.left + "," + margin.top + ")");
    var context = svg.append("g")
      .attr("transform", "translate(" + margin2.left + "," + margin2.top + ")");
    svg.call(tip);
    var dataset = data;
    var maxHeight = d3.max(dataset, function (d) { return d });
    var minHeight = d3.min(dataset, function (d) { return d })


    var yScale = d3.scaleLinear().range([0, height]).domain([maxHeight, 0]);

    var xScale = d3.scaleBand().range([0, width]).padding(0.1);
    xScale.domain(dataset.map(function (d, i) { return i }));

    var yScale2 = d3.scaleLinear().range([0, height2]).domain([maxHeight, 0]);

    var xScale2 = d3.scaleBand().range([0, width]).padding(0.1);
    xScale2.domain(xScale.domain());

    var yAxis = d3.axisLeft(yScale)
    var yAxisGroup = focus.append("g").call(yAxis);
    var xAxis = d3.axisBottom(xScale).tickValues([])
    var xAxisGroup = focus.append("g").call(xAxis).attr("transform", "translate(0," + height + ")");

    var xAxis2 = d3.axisBottom(xScale2).tickValues([]);
    var xAxisGroup2 = context.append("g").call(xAxis2).attr("transform", "translate(0," + height2 + ")");

    var bars1 = focus.selectAll("rect").data(dataset).enter().append("rect").classed("bar",true).on('mouseover', tip.show)
      .on('mouseout', tip.hide)
;
    bars1.attr("x", function (d, i) {
      return xScale(i);
    })
      .attr("y", function (d) {
        return yScale(d);
      })
      .attr("width", xScale.bandwidth())
      .attr("height", function (d) {
        return height - yScale(d);
      });
    bars1.attr("fill", function (d) {
      return "steelblue";
    });

    var bars2 = context.selectAll("rect").data(dataset).enter().append("rect").classed("bar2",true);
    bars2.attr("x", function (d, i) {
      return xScale2(i);
    })
      .attr("y", function (d) {
        return yScale2(d);
      })
      .attr("width", xScale2.bandwidth())
      .attr("height", function (d) {
        return height2 - yScale2(d);
      });
    bars2.attr("fill", function (d) {
      return "steelblue";
    });

    var brush = d3.brushX()
      .extent([[0, 0], [width, height2]])
      .on("brush", brushed)
      .on("end", brushend);

    context.append("g")
      .attr("class", "brush")
      .call(brush)
      .call(brush.move, xScale2.range());

    updateScale(brushArea = null);

    function brushed() {
      if (!d3.event.sourceEvent) return;
      if (!d3.event.selection) return;
      if (d3.event.sourceEvent && d3.event.sourceEvent.type === "zoom") return;
      var brushArea = d3.event.selection;

      updateScale(brushArea);


    }

    function brushend() {
      if (!d3.event.sourceEvent) return;
      if (!d3.event.selection) return;
      if (d3.event.sourceEvent && d3.event.sourceEvent.type === "zoom") return; 
      var newInput = [];
      var brushArea = d3.event.selection;
      if (brushArea === null) brushArea = xScale.range();


      xScale2.domain().forEach(function (d) {
        var pos = xScale2(d) + xScale2.bandwidth() / 2;
        if (pos >= brushArea[0] && pos <= brushArea[1]) {
          newInput.push(d);
        }
      });

      var increment = 0;
      var left = xScale2(d3.min(newInput));
      var right = xScale2(d3.max(newInput)) + xScale2.bandwidth();

      d3.select(this).transition().call(d3.event.target.move, [left, right]);
    }

    function updateScale(brushArea) {
      var newInput = [];

      if (brushArea === null)
        brushArea = xScale2.range()

      xScale2.domain().forEach(function (d) {
        var pos = xScale2(d);
        if (pos >= brushArea[0] && pos <= brushArea[1]) {
          newInput.push(d);
        }
      });

      xScale.domain(newInput);

      bars1.attr("x", function (d, i) {
        return xScale(i)
      })
        .attr("y", function (d) {
          return yScale(d);
        })
        .attr("width", xScale.bandwidth())
        .attr("height", function (d, i) {
          if (xScale.domain().indexOf(i) === -1) {
            return 0;
          }
          else
            return height - yScale(d);
        })
      var tickScale = d3.scalePow().range([newInput.length / 4, 0]).domain([newInput.length, 0]).exponent(.5)
      var brushValue = brushArea[1] - brushArea[0];
      if (brushValue === 0) {
        brushValue = width;
      }

      var tickValueMultiplier = Math.ceil(Math.abs(tickScale(brushValue)));
      var filteredTickValues = newInput.filter(function (d, i) { return i % tickValueMultiplier === 0 }).map(function (d) { return d })
      xAxisGroup.call(xAxis.tickValues(filteredTickValues));
    }

  </script>*/
