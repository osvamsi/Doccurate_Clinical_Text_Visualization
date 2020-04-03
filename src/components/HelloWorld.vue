<template>
  <div class="d-flex" id="wrapper" style="margin-left:2%;">
    <div class="blur" style="margin-left:2%; padding:8px">
      <line-chart></line-chart>
    </div>

    <!-- <div style="margin-left:2%; width: 35%;" id="textdetails" ></div> -->
    <div id="container">
      <div
        id="treeparent"
        class="fixed-content treeparent"
        style="background-color:rgba(255, 255, 255, 0.8); display:none; position:fixed; top:0%; left:0%; width:100%"
      >
        <button style="position:fixed; right:3%; top:3%" @click="reverse">X</button>
        <div id="tree"></div>
      </div>
    </div>
  </div>
</template>



<script>
import LineChart from "./LineChart";
import * as d3 from "d3";
export default {
  name: "HelloWorld",
  components: {
    "line-chart": LineChart
  },
  props: {
    msg: String
  },
  methods: {
    tempfn: function() {
      alert("gqacdjgaef");
    },
    reverse: function() {
      location.reload();
      // d3.select(".blur").style("opacity", "1");

      // d3.select(".treeparent")
      //   .style("display", "none")
      //   .style("opacity", "1");
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.fixed-content {
  top: 0;
  bottom: 0;
  position: fixed;
  overflow-y: scroll;
  overflow-x: hidden;
}

h3 {
  margin: 40px 0 0;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
</style>
