<template>
  <div class="d-flex" id="wrapper" style="margin-left:2%;">
    <div style="height:300px; width:11%;">
      <b-button
        id="toggleData"
        class="mb-3 bg-primary"
        type="button"
        @click="onToggleData"
      >Toggle Data</b-button>
      <div class="sidebar-heading text-center m-1">
        <b style="margin-left:2%;">Patient Information</b>
      </div>
      <div
        style="background-color: lightgray; overflow:hidden; border-radius: 5px;"
        id="patientinfo"
      ></div>
      <div style="margin-top: 20%;" class="sidebar-heading">
        <b>Filter Collection</b>
      </div>
      <div class="div_fc">
        <!-- <span
          v-for="item in diseasesFc"
          :key="item"
          style="border-radius: 5px; background-color: lightgray; "
          class="list-group-item p-1 mb-1"
        >{{ item }}</span>-->
      </div>
    </div>

    <!-- Volin Plot From here -->
    <div class="position-relative" style="margin-left:2%">
      <b-button class="bg-info" id="back-button" @click="onClickBack">Back</b-button>
      <div id="my_dataviz"></div>

      <!-- <button type="button" @click="treefun">Tree</button> -->
    </div>

    <!-- Patient Records from here -->
    <!-- <div style="margin-left:2%; width: 35%;" id="textdetails"> -->

    <div style="margin-left:2%; width: 35%;">
      <div style="background-color:rgba(0,0,0,0.1)">Records {{this.position + 1}} / {{items.length}}</div>

      <virtual-list
        :size="1000"
        :remain="1000"
        :variable="true"
        class="hello"
        style="height: 75vh;"
        ref="scroller"
        @scroll.native="myMethod"
      >
        <item-div
          v-for="(val,key,index) of items"
          :key="key"
          v-bind:mydata="val"
          v-bind:keywords="keywords"
          v-bind:class="index"
          v-bind:itemFunction="treefun"
        ></item-div>
      </virtual-list>
    </div>
    <button id="goToTopBtn" type="button" @click="onClickTop">GO TO TOP!</button>
  </div>
</template>

<script>
import * as d3 from "d3";
import { parse } from "path";
import { constants } from "fs";
import Item from "./Items";
import virtualList from "vue-virtual-scroll-list";
import { fuchsia } from "color-name";
var lvl = "FC Tuples";
var level_selected = 0;
var level_value = "";
var parent_stack = [];
var x_domain_values = [];
var selected_index = 0;
var initial_max = 0;
var medical_records = [];
var sum_stat_records;
var encounter_id = "";
var y_axix;
var y_selected_val = 0;
var margin = { top: 10, right: 30, bottom: 30, left: 40 };
var width = 460 - margin.left - margin.right;
var height = 400 - margin.top - margin.bottom;
var patients = ["final_1.csv", "final_2.csv"];
var selected_dataSet_file = 0;

export default {
  name: "LineChart",
  props: {
    data: {
      required: false,
      default: null
    }
  },
  data() {
    var res = [];
    return {
      items: res,
      position: 0,
      keywords: [],
      diseasesFc: []
    };
  },
  components: {
    "item-div": Item,
    "virtual-list": virtualList
  },

  methods: {
    onToggleData: function() {
      lvl = "FC Tuples";
      level_selected = 0;
      level_value = "";
      parent_stack = [];
      if (selected_dataSet_file) {
        selected_dataSet_file = 0;
      } else {
        selected_dataSet_file = 1;
      }
      d3.select("#my_dataviz")
        .selectAll("svg")
        .remove();
      this.updateDataFunc();
    },
    myMethod: function(event) {
      var mybutton = document.getElementById("goToTopBtn");
      if (this.position >= 1) {
        mybutton.style.display = "block";
      } else {
        mybutton.style.display = "none";
      }

      var off = 0;
      off = (this.items.length - 1) * 3.4;
      var len = (event.target.scrollHeight - off) / this.items.length;
      var pos = Math.round(event.target.scrollTop / len);
      this.position = pos;
      this.position = Math.min(pos, this.items.length - 1);
      this.findRecord();
    },
    findRecord: function() {
      //console.log("Finding rec",sum_stat_records);
      for (var i in sum_stat_records) {
        for (var rec in sum_stat_records[i].value) {
          if (rec.length) {
            for (var ind_rec in sum_stat_records[i].value[rec]) {
              if (
                sum_stat_records[i].value[rec][ind_rec]["ENCOUNTER"] ==
                this.items[this.position]["ENCOUNTER"]
              ) {
                y_selected_val = y_axix(sum_stat_records[i].value[rec].x0);
              }
            }
          }
        }
      }
      if (y_selected_val) {
        d3.select(".mouse-line")
          .attr("d", function() {
            // var y_val =  mouse[1] - 2 * margin.top;
            var d = "M" + 0 + "," + y_selected_val;
            d += " " + width + "," + y_selected_val;
            return d;
          })
          .style("opacity", "0.7");
      }
    },

    setDataValue: function(data) {
      this.items = data;
    },

    //tree================================/
    treefun: function(event) {
      console.log(event.target.textContent);

      var t = "";
      if (event.target.textContent != null) {
        t = event.target.textContent;
      }

      //tree taxonomy
      var margin = { top: 10, right: 30, bottom: 30, left: 40 };

      var margintree = { top: 30, right: 20, bottom: 30, left: 20 },
        widthtree = 1200,
        barHeight = 20,
        barWidth = (widthtree - margintree.left - margintree.right) * 0.5;

      var i = 0,
        duration = 400,
        root;

      var diagonal = d3
        .linkHorizontal()
        .x(function(d) {
          return d.y;
        })
        .y(function(d) {
          return d.x;
        });

      var svgtree = d3
        .select("#tree")
        .append("svg")
        .attr("width", widthtree) // + margintree.left + margintree.right)
        .attr("height", "8000px")
        .append("g")
        .attr(
          "transform",
          "translate(" + margintree.left + "," + margintree.top + ")"
        );
      d3.select(".blur").style("opacity", "0.1");

      d3.select(".treeparent")
        .style("display", "block")
        .style("opacity", "1");

      d3.json("level_3_trees.json", function(error, flare) {
        if (error) throw error;
        console.log(t);
        console.log(flare[t]);

        root = d3.hierarchy(flare[t]);
        root.x0 = 0;
        root.y0 = 0;
        update(root);
      });

      function update(source) {
        // Compute the flattened node list.
        var nodes = root.descendants();

        var height = Math.max(
          500,
          nodes.length * barHeight + margin.top + margin.bottom
        );

        // d.children = d._children;
        // d._children = null;

        d3.select("svg")
          .transition()
          .duration(duration)
          .attr("height", height);

        d3.select(self.frameElement)
          .transition()
          .duration(duration)
          .style("height", height + "px");

        // Compute the "layout". TODO https://github.com/d3/d3-hierarchy/issues/67
        var index = -1;
        root.eachBefore(function(n) {
          n.x = ++index * barHeight;
          n.y = n.depth * 20;
        });

        // Update the nodes…
        var node = svgtree.selectAll(".node").data(nodes, function(d) {
          return d.id || (d.id = ++i);
        });

        var nodeEnter = node
          .enter()
          .append("g")
          .attr("class", "node")
          .attr("transform", function(d) {
            return "translate(" + source.y0 + "," + source.x0 + ")";
          })
          .style("opacity", 0);

        // Enter any new nodes at the parent's previous position.
        nodeEnter
          .append("rect")
          .attr("y", -barHeight / 2)
          .attr("height", barHeight)
          .attr("width", barWidth)
          .style("fill", color)
          .on("click", click);

        nodeEnter
          .append("text")
          .attr("dy", 3.5)
          .attr("dx", 5.5)
          .text(function(d) {
            return d.data.name;
          })
          .style("font-size", "11px");

        // Transition nodes to their new position.
        nodeEnter
          .transition()
          .duration(duration)
          .attr("transform", function(d) {
            return "translate(" + d.y + "," + d.x + ")";
          })
          .style("opacity", 1);

        node
          .transition()
          .duration(duration)
          .attr("transform", function(d) {
            return "translate(" + d.y + "," + d.x + ")";
          })
          .style("opacity", 1)
          .select("rect")
          .style("fill", color);

        // Transition exiting nodes to the parent's new position.
        node
          .exit()
          .transition()
          .duration(duration)
          .attr("transform", function(d) {
            return "translate(" + source.y + "," + source.x + ")";
          })
          .style("opacity", 0)
          .remove();

        // Update the links…
        var link = svgtree.selectAll(".link").data(root.links(), function(d) {
          return d.target.id;
        });

        // Enter any new links at the parent's previous position.
        link
          .enter()
          .insert("path", "g")
          .attr("class", "link")
          .attr("d", function(d) {
            var o = { x: source.x0, y: source.y0 };
            return diagonal({ source: o, target: o });
          })
          .transition()
          .duration(duration)
          .attr("d", diagonal);

        // Transition links to their new position.
        link
          .transition()
          .duration(duration)
          .attr("d", diagonal);

        // Transition exiting nodes to the parent's new position.
        link
          .exit()
          .transition()
          .duration(duration)
          .attr("d", function(d) {
            var o = { x: source.x, y: source.y };
            return diagonal({ source: o, target: o });
          })
          .remove();

        // Stash the old positions for transition.
        root.each(function(d) {
          d.x0 = d.x;
          d.y0 = d.y;
        });
      }

      // Toggle children on click.
      function click(d) {
        if (d.children) {
          d._children = d.children;
          d.children = null;
        } else {
          d.children = d._children;
          d._children = null;
        }
        update(d);
      }

      function color(d) {
        return d._children ? "#3182bd" : d.children ? "#c6dbef" : "#fd8d3c";
      }

      //================================
    },
    //tree================================
    myMethod: function(event) {
      var mybutton = document.getElementById("goToTopBtn");
      if (this.position >= 1) {
        mybutton.style.display = "block";
      } else {
        mybutton.style.display = "none";
      }

      var off = 0;
      off = (this.items.length - 1) * 3.4;
      var len = (event.target.scrollHeight - off) / this.items.length;
      var pos = Math.round(event.target.scrollTop / len);
      this.position = pos;
      this.position = Math.min(pos, this.items.length - 1);
      this.findRecord();
    },

    tempfn: function() {
      alert("gqacdjgaef");
    },
    onClickTop: function() {
      this.$refs["scroller"].setScrollTop();
    },
    onClickBack: function() {
      if (level_selected <= 0) {
        return;
      }
      level_selected -= 1;
      if (level_selected == 0) {
        document.getElementById("back-button").style.visibility = "hidden";
      }
      console.log("Parent Stack ", parent_stack);
      level_value = parent_stack.pop();

      d3.select("#my_dataviz")
        .selectAll("svg")
        .remove();
      this.$refs["scroller"].setScrollTop();

      console.log(level_selected, level_value);
      d3.select("#my_dataviz")
        .selectAll("svg")
        .remove();

      this.updateDataFunc();
    },

    updateDataFunc: function() {
      // console.log(this.$refs["scroller"]);
      // this.$refs["scroller"].setScrollTop();
      var that = this;

      // updateDataFunc: function updateData() {

      d3.json("keywords.json", function(data) {
        that.keywords = data["array"];
        // console.log(that.keywords);
      });
      // set the dimensions and margins of the graph
      var margin = { top: 10, right: 30, bottom: 30, left: 40 },
        width = 460 - margin.left - margin.right,
        height = 400 - margin.top - margin.bottom;

      var color = d3.scaleOrdinal(d3.schemeCategory10);
      var nextLevelColors = [];
      for (var i = 0; i < 11; i++) {
        nextLevelColors.push(
          d3
            .scaleLinear()
            .domain([1, 10])
            .interpolate(d3.interpolateHcl)
            .range([d3.rgb(color(i)).brighter(1), d3.rgb(color(i)).darker(30)])
        );
      }

      // append the svg object to the body of the page
      var svg = d3
        .select("#my_dataviz")
        .append("svg")
        .attr("width", width + margin.left + margin.right)
        .attr("height", height + margin.top + margin.bottom)
        .append("g")
        .attr("transform", "translate(" + margin.left + "," + margin.top + ")");

      var svg_transition = d3.select("#my_dataviz").transition();
      var glob_sumstat;
      d3.csv(patients[selected_dataSet_file], function(data_patient) {
        var svg_transition = d3.select("#my_dataviz").transition();
        // Read the data and compute summary statistics for each specie

        var y = d3
          .scaleTime()
          .domain(
            d3.extent(data_patient, function(d) {
              // return new Date(d.start);
              let mydate = Date.parse(d.start, "mm/dd/yyyy");

              return mydate;
            })
          )
          .range([50, width]);
        y_axix = y;
        svg.append("g").call(d3.axisLeft(y));

        // Features of the histogram
        var histogram = d3
          .histogram()
          .domain(function() {
            return y.domain().map(function(d) {
              return Date.parse(d);
            });
          })
          .thresholds(y.ticks(20)) // Important: how many bins approx are going to be made? It is the 'resolution' of the violin plot
          .value(function(d) {
            return Date.parse(d.start, "mm/dd/yyyy");
          });

        var record = [];
        var sumstat = d3
          .nest() // nest function allows to group the calculation per level of a factor
          .key(function(d, i) {
            if (level_selected == 1 && d["FC Tuples"] == level_value) {
              record.push(d);
              return d["Disease Tuples"].trim();
            } else if (level_selected == 1) {
              return " ";
            }
            if (
              level_selected == 2 &&
              d["Disease Tuples"].trim() == level_value
            ) {
              record.push(d);
              return d["Sympton Tuples"].trim();
            } else if (level_selected == 2) {
              return " ";
            }

            if (d[lvl]) {
              record.push(d);
              return d[lvl];
            } else {
              return " ";
            }
          })
          .rollup(function(d) {
            // For each key..

            var input = d.map(function(g) {
              // return Date.parse(g.start, "mm/dd/yyyy");
              return g;
            }); // Keep the variable called Sepal_Length
            var bins = histogram(input); // And compute the binning on it.
            return bins;
          })
          .entries(data_patient);

        sum_stat_records = sumstat;
        that.setDataValue(record);

        x_domain_values = [];
        var del_index = -1;
        for (let index in sumstat) {
          if (sumstat[index]["key"] == " ") {
            del_index = index;
          } else {
            x_domain_values.push(sumstat[index]["key"]);
          }
        }

        if (level_selected == 0) {
          d3.select(".div_fc")
            .selectAll("*")
            .remove();

          d3.select(".div_fc")
            .selectAll("span")
            .data(x_domain_values)
            .enter()
            .append("span")
            .text(function(d) {
              return d;
            })
            .attr("class", "list-group-item p-1 mb-1 text-white rounded")
            .style("background-color", function(d, i) {
              return d3.rgb(color(i)).brighter(1);
            });
        }
        sumstat.splice(del_index, 1);

        // What is the biggest number of value in a bin? We need it cause this value will have a width of 100% of the bandwidth.
        var maxNum = 0;

        for (let i in sumstat) {
          var allBins = sumstat[i].value;
          var total_len = 0;
          var lengths = allBins.map(function(a) {
            total_len += a.length;
            return a.length;
          });
          sumstat[i]["total_len"] = total_len;
          var lengths = allBins.map(function(a) {
            return a.length;
          });
          var longuest = d3.max(lengths);
          if (longuest > maxNum) {
            maxNum = longuest;
          }
        }

        if (level_selected == 0) {
          initial_max = maxNum;
        }

        var x_top = d3
          .scaleBand()
          .range([0, width])
          .domain(x_domain_values)
          .padding(0.05); // This is important: it is the space between 2 groups. 0 means no padding. 1 is the maximum.

        var x_label_axis = svg
          .append("g")
          .attr("transform", "translate(0," + 30 + ")")
          .attr("class", "axisRed")
          .call(d3.axisTop(x_top))
          .selectAll("text")
          .attr("transform", "rotate(-25) translate(25,20)")
          .style("font-size", "small")
          .style("fill", function(d, i) {
            if (level_selected == 0) {
              return color(i);
            }
            if (level_selected == 1) {
              return nextLevelColors[selected_index](i);
            }
            return color(selected_index);
          })
          .on("click", function(d, i) {
            if (level_selected == 2) {
              return;
            }
            document.getElementById("back-button").style.visibility = "visible";
            parent_stack.push(level_value);
            if (level_selected == 0) {
              selected_index = x_domain_values.findIndex(function(val) {
                return val == d;
              });
            }
            level_selected += 1;
            level_value = d;
            // change code to color
            // console.log("Parent Stack ", parent_stack);
            d3.select("#my_dataviz")
              .selectAll("svg")
              .remove();
            that.updateDataFunc();
          });

        // The maximum width of a violin must be x.bandwidth = the width dedicated to a group
        var xNum = d3
          .scaleLinear()
          .range([0, x_top.bandwidth()])
          .domain([-initial_max, initial_max]);

        // This allows to find the closest X index of the mouse:
        var bisect = d3.bisector(function(d) {
          return d.x;
        }).left;

        var div_selected = d3
          .select("#my_dataviz")
          .append("div")
          .attr("class", "tooltip")
          .style("opacity", 0);
        // Add the shape to this svg!
        svg
          .selectAll("myViolin")
          .data(sumstat)
          .enter() // So now we are working group per group
          .append("g")
          .attr("transform", function(d) {
            return "translate(" + x_top(d.key) + " ,0)";
          }) // Translation on the right to be at the group position
          .append("path")
          .datum(function(d) {
            return d.value;
          }) // So now we are working bin per bin
          .style("stroke", "none")
          .style("fill", function(d, i) {
            if (level_selected == 0) {
              return color(i);
            }
            if (level_selected == 1) {
              return nextLevelColors[selected_index](i);
            }
            return color(selected_index);
          })
          .attr(
            "d",
            d3
              .area()
              .x0(function(d) {
                if (d.length == 0) {
                  return xNum(0.1);
                }
                return xNum(-d.length);
              })
              .x1(function(d) {
                return xNum(d.length);
              })
              .y(function(d) {
                return y(d.x0);
              })
              .curve(d3.curveCatmullRom) // This makes the line smoother to give the violin appearance. Try d3.curveStep to see the difference
          )

          .on("mousemove", function(data, i) {
            // console.log("Data", data);
            var mappedVales = data.map(x => x.x0);

            var y_scaled = y.invert(d3.mouse(this)[1]);
            // console.log("Y", y_scaled, mappedVales);
            var index = d3.bisectLeft(mappedVales, y_scaled);
            var selectedData = mappedVales[index];
            // console.log("Selected Index : ", index, selectedData);

            div_selected
              .transition()
              .duration(200)
              .style("opacity", 0.9);
            div_selected
              .html(data[index].length)
              .style("left", d3.event.pageX - 250 + "px")
              .style("top", d3.event.pageY - 110 + "px");
          })
          .on("mouseout", function(n) {
            div_selected
              .transition()
              .duration(500)
              .style("opacity", 0);
          });

        var div_selected1 = d3
          .select("#my_dataviz")
          .on("mousemove", function(data, i) {
            var mouse = d3.mouse(this);

            d3.select(".mouse-line").attr("d", function() {
              var y_val = mouse[1] - 2 * margin.top;
              var d = "M" + 0 + "," + y_selected_val;
              d += " " + width + "," + y_selected_val;
              return d;
            });
          })
          .on("mouseover", function(d) {})
          .on("mouseout", function() {
            // on mouse out hide line, circles and text
          });

        var mouseG = svg.append("g").attr("class", "mouse-over-effects");

        mouseG
          .append("path") // this is the black vertical line to follow mouse
          .attr("class", "mouse-line")
          .style("stroke", "black")
          .style("stroke-width", "1px")
          .style("opacity", "1");
        that.findRecord();

        //horizontal bar chart
        var margin_for_bar_chart = { top: 5, right: 5, bottom: 5, left: 5 },
          width_for_bar_chart = 200 - margin.left - margin.right,
          height_for_bar_chart = 450 - margin.top - margin.bottom;

        var FCs = data_patient.map(function(d) {
          return d.Filter_Collection;
        });
        var FCs_unique = [...new Set(FCs)];
        barchart(sumstat);
      }); //final_1 csv closed here

      function barchart(glob_sumstat) {
        //horizontal bar chart
        var l = 0;
        for (var a in glob_sumstat) {
          l++;
        }
        var margin_for_bar_chart = { top: 5, right: 5, bottom: 5, left: 5 },
          width_for_bar_chart = 200 - margin.left - margin.right,
          height_for_bar_chart = 25 * l + 100 - margin.top - margin.bottom;

        var svg1 = d3
          .select("#my_dataviz")
          .append("svg")
          .attr(
            "width",
            width_for_bar_chart +
              margin_for_bar_chart.left +
              margin_for_bar_chart.right
          )
          .attr(
            "height",
            height_for_bar_chart +
              margin_for_bar_chart.top +
              margin_for_bar_chart.bottom
          )
          .append("g")
          .attr(
            "transform",
            "translate(" +
              margin_for_bar_chart.left +
              "," +
              margin_for_bar_chart.top +
              ")"
          );

        var x = d3.scaleLinear().range([0, width_for_bar_chart]);
        var y = d3.scaleBand().range([height_for_bar_chart, 0]);

        glob_sumstat.sort(function(a, b) {
          return a.total_len - b.total_len;
        });

        x.domain([
          0,
          d3.max(glob_sumstat, function(d) {
            return d.total_len;
          })
        ]);
        y.domain(
          glob_sumstat.map(function(d, i) {
            return i;
          })
        ).padding(0.1);

        svg1
          .selectAll(".bar")
          .data(glob_sumstat)
          .enter()
          .append("rect")
          .attr("class", "bar")
          .attr("x", 0)
          .style("fill", "lightgray")
          .attr("height", 25)
          .attr("y", function(d, i) {
            return y(i) - 5;
          })
          .attr("width", function(d) {
            return x(d.total_len);
          });

        svg1
          .selectAll("text")
          .data(glob_sumstat)
          .enter()
          .append("text")
          .style("text-anchor", "start")
          .text(function(d) {
            return d.key;
          })
          .attr("x", 0)
          .attr("y", function(d, i) {
            return y(i) + 12;
          })
          .style("width", "20px")
          .style("font-size", "12px")
          .style("fill", function(d, i) {
            if (level_selected == 0) {
              var original_index = x_domain_values.findIndex(function(val) {
                return val == d.key;
              });
              return color(original_index);
            }
            return color(selected_index);
          });
      }
      //patient info tab dynamic property
      d3.select("#patientinfo")
        .selectAll("*")
        .remove();

      d3.csv("Patient1_info.csv", function(error, data) {
        if (error) throw error;

        data.forEach(function(d) {
          d.key = d.key.toUpperCase();
          d.val = d.val;
        });

        var pat = d3
          .select("#patientinfo")
          .selectAll("h6")
          .data(data)
          .enter()
          .append("h6")
          .style("width", "150px")
          .style("margin-top", "11%")
          .style("margin-left", "4%")
          .text(function(d) {
            return d.key + ": ";
          })
          .style("text-align", "left")
          .style("font-weight", "bold")
          .append()
          .text(function(d) {
            return d.val;
          })
          .style("font-weight", "normal")
          .style("text-align", "left");

        //patient info tab dynamic property end
      }); //patient_info csv closed
    }
  },

  mounted: function() {
    this.updateDataFunc();
  } //end of the mounted function
};
</script>

<style>
body {
  font: 10px sans-serif;
}

.axisRed .domain {
  fill: none;
  stroke: #3f3;
  opacity: 0;
  shape-rendering: crispEdges;
}

.axisRed .tick line {
  fill: none;
  stroke: #3f3;
  opacity: 0;
  shape-rendering: crispEdges;
}

div.tooltip {
  position: absolute;
  text-align: center;
  width: 60px;
  height: 28px;
  padding: 2px;
  font: 12px sans-serif;
  background: lightsteelblue;
  border: 0px;
  border-radius: 8px;
  pointer-events: none;
}

#goToTopBtn {
  display: none;
  position: fixed;
  bottom: 20px;
  right: 30px;
  z-index: 99;
  font-size: 12px;
  border: none;
  outline: none;
  background-color: rgba(255, 0, 0, 0.8);
  color: white;
  cursor: pointer;
  padding: 8px;
  border-radius: 4px;
}

.node rect {
  cursor: pointer;
  fill: #fff;
  fill-opacity: 0.5;
  stroke: #3182bd;
  stroke-width: 1.5px;
}

.node text {
  font: 10px sans-serif;
  pointer-events: none;
}

.link {
  fill: none;
  stroke: #9ecae1;
  stroke-width: 1.5px;
}

#back-button {
  position: absolute;
  left: 0px;
  visibility: hidden;
}
</style>