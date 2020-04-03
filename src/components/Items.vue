<template>
  <div class="box">
    <h4>Name: John Wick</h4>
    <text-highlight :queries="keywords" @click="alert">{{ this.mydata["Medical record"] }}</text-highlight>
  </div>
</template>


<script>
import Vue from "vue";
import TextHighlight from "vue-text-highlight";

Vue.component("text-highlight", TextHighlight);

export default {
  name: "item",
  props: {
    mydata: Object,
    keywords: {
      type: Array,
      default: [1]
    },
    itemFunction: {
      type: Function,
      required: true
    }
  },
  data() {
    return {
      queries: ["birds", "scatt"],
      description: "Tropical birds scattered as Drake veered the Jeep",
      activeColor: "red",
      fontSize: 30
    };
  },
  methods: {
    tempfn: function() {
      // console.log(this.keywords);
    },
    alert: function(data) {
      this.itemFunction(data);
    }
  },
  mounted: function() {
    this.tempfn();
  },
  computed: {
    datawithbr: function() {
      const doc = this.mydata["Medical record"];
      return doc.replace(/(\\r)*\\n/g, "<br>");
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.box:nth-of-type(odd) {
  background-color: rgba(200, 200, 200, 0.8);
}

.box:nth-of-type(even) {
  background-color: rgba(200, 200, 200, 0.1);
}

.box {
  padding: 10px;
  margin-top: 20px;
  border: 2px solid rgba(200, 200, 200, 0.9);
}
</style>
