<template>
  <div>
    <div id="labelManage">
      <div class="main">
        <div class="flowchart-demo" id="flowchart-demo">
          <div class="window" id="flowchartWindow1"></div>
          <div class="window" id="flowchartWindow2"></div>
          <div class="window" id="flowchartWindow3"></div>
          <div class="window" id="flowchartWindow4"></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import jsplumb from 'jsplumb'

const jsPlumb = jsplumb.jsPlumb

export default {
  name: 'HelloWorld',
  mounted () {
    console.log('==', jsplumb)
    this.jsPlumb()
  },
  methods: {
    jsPlumb () {
      jsPlumb.ready(function () {
        jsPlumb.connect({
          source: 'flowchartWindow1',
          target: 'flowchartWindow2',
          endpoint: 'Rectangle'
        })
        jsPlumb.draggable('flowchartWindow1')
        jsPlumb.draggable('flowchartWindow2')
      })
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped lang="scss">
  .flowchart-demo{
    width:800px;
    height:600px;
    border:1px solid #000;
    position:relative;
  }
  .flowchart-demo .window{
    border:1px solid #346789;
    box-shadow:2px 2px 19px #aaa;
    -o-box-shadow:2px 2px 2px 19px #aaa;
    -moz-box-shadow:2px 2px 19px #aaa;

    -moz-border-radius:0.5em;
    border-radius:0.5em;
    opacity:0.5;
    filter:alpha(opacity=80);
    text-align: center;
    position: absolute;
    background-color: #eeeeef;
    color:black;
    font-family:helvetica;
    font-size:0.9em;
    line-height:60px;
    width:60px;
    height:60px;
  }
  .flowchart-demo .window:hover{
    box-shadow:2px 2px 19px #444;
    -o-box-shadow:2px 2px 19px #444;
    -webkit-box-shadow:2px 2px 19px #444;
    -moz-box-shadow:2px 2px 19px #444;
    opacity:0.6;
    filter:alpha(opacity=60);
  }
  .flowchart-demo .active{
    border:1px dotted green;
  }
  .flowchart-demo .hover{
    border:1px dotted red;
  }
  #flowchartWindow1 {
    top:34em;
    left:5em;
  }
  #flowchartWindow2 {
    top:7em;
    left:36em;
  }
  #flowchartWindow3{
    top:27em;
    left:48em;
  }
  #flowchartWindow4{
    top:23em;
    left:22em;
  }
</style>
