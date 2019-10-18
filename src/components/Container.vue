<template>
    <div>
      <div id="labelManage">
      </div>
    </div>
</template>

<script>
import jsplumb from 'jsplumb'

const jsPlumb = jsplumb.jsPlumb

export default {
  name: 'Container',
  mounted () {
    this.jsPlumb()
  },
  methods: {
    jsPlumb () {
      jsPlumb.ready(function () {
      })
    }
  }
}
</script>

<style scoped>

</style>
