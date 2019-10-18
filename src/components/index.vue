<template>
    <div>
      <el-row>
        <el-col :span="3" ref="flowTool">
          <AsideMenu @addNode="addNode"></AsideMenu>
        </el-col>
        <el-col :span="21">
          <el-row>
            <div>
                <el-button type="primary" @click="dataInfo">流程信息</el-button>
              <el-button type="primary" @click="changeLabel">设置线条</el-button>
            </div>
          </el-row>
          <el-row ref="flowContainer">
            <div id="flowContainer" class="container">
              <template v-for="node in data.nodeList" >
                <Node :id="node.id" v-show="node.show" :key="node.id" :node="node" @changeNodeSite="changeNodeSite" @editNode="editNode" @deleteNode="deleteNode"></Node>
              </template>
            </div>
          </el-row>
        </el-col>
      </el-row>
      <FlowInfo v-if="flowInfoVisable" ref="flowInfo" :data="data"></FlowInfo>
      <FlowNodeForm v-if="nodeFormVisible" ref="nodeForm"></FlowNodeForm>
    </div>
</template>

<script>
import AsideMenu from './aside-menu.vue'
import Container from './container.vue'
import {jsPlumb} from 'jsplumb'
import {getDataA} from './data_A'
import Node from './node'
import FlowInfo from './flow-info'
import FlowNodeForm from './flow-node-form'

export default {
  name: 'index',
  components: {
    AsideMenu,
    Container,
    Node,
    FlowInfo,
    FlowNodeForm
  },
  data () {
    return {
      nodeFormVisible: false,
      flowInfoVisable: false,
      jsPlumb: null,
      index: 1,
      // 是否加载完毕
      loadEasyFlowFinish: false,
      data: {},
      // 默认设置参数
      jsplumbSetting: {
        // 动态锚点 位置自适应
        Anchors: ['Top', 'TopCenter', 'TopRight', 'TopLeft', 'Right', 'RightMiddle', 'Bottom', 'BottomCenter', 'BottomRight', 'BottomLeft', 'Left', 'LeftMiddle'],
        Container: 'flowContainer',
        // 连线的样式 StateMachine Flowchart
        Connector: 'Flowchart',
        // 鼠标不能拖动删除线
        ConnectionDetachable: false,
        // 删除线的时候节不能删除
        DeleteEndpointsOnDetach: false,
        // 连线的端点
        // Endpoint: ["Dot", {radius: 5}],
        Endpoint: ['Rectangle', {height: 10, width: 10}],
        // 线端点的样式
        EndpointStyle: {fill: 'rgba(255,255,255,0)', outlineWidth: 1},
        LogEnabled: true, // 是否打开jsPlumb的内部日志记录
        // 绘制线
        PaintStyle: {stroke: 'black', strokeWidth: 3},
        // 绘制箭头
        Overlays: [['Arrow', {width: 12, length: 12, location: 1}]],
        RenderMode: 'svg'
      },
      // jsplumb连接参数
      jsplumbConnectOptions: {
        isSource: true,
        isTarget: true,
        // 动态锚点 提供了4个方向 Continuous AutoDefault
        anchor: 'Continuous'
      },
      jsplumbSourceOptions: {
        filter: '.flow-node-drag',
        filterExclude: false,
        anchor: 'Continuous',
        allowLoopback: false
      },
      jsplumbTargetOptions: {
        filter: '.flow-node-drag',
        filterExclude: false,
        anchor: 'Continuous',
        allowLoopback: false
      }

    }
  },
  methods: {
    jsPlumbInit () {
      const _this = this
      this.jsPlumb.ready(function () {
        // 导入默认配置
        _this.jsPlumb.importDefaults(_this.jsplumbSetting)
        // 会使整个jsPlumb立即重绘
        _this.jsPlumb.setSuspendDrawing(false, true)
        // 初始化节点
        _this.loadEasyFlow()

        // 当点击了连接线
        _this.jsPlumb.bind('click', function (conn, originalEvent) {
          _this.$confirm('确定删除所点击的连接线吗？', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            _this.jsPlumb.deleteConnection(conn)
          }).catch(() => {
          })
        })

        // 连线
        _this.jsPlumb.bind('connection', function (evt) {
          let from = evt.source.id
          let to = evt.target.id
          if (_this.loadEasyFlowFinish) {
            _this.data.lineList.push({
              from: from,
              to: to
            })
          }
        })

        _this.jsPlumb.bind('beforeDrop', function (evt) {
          let from = evt.sourceId
          let to = evt.targetId
          if (from === to) {
            _this.$message.error('不能连接自己')
            return false
          }
          if (_this.hasLine(from, to)) {
            _this.$message.error('不能重复连线')
            return false
          }
          _this.$message({
            message: '恭喜你，这是一条成功消息',
            type: 'success'
          })
          return true
        })

        // 删除连线
        _this.jsPlumb.bind('connectionDetached', function (evt) {
          _this.deleteLine(evt.sourceId, evt.targetId)
        })
      })
    },
    // 加载流程图
    loadEasyFlow () {
      // 初始化节点
      for (var i = 0; i < this.data.nodeList.length; i++) {
        let node = this.data.nodeList[i]
        // 设置源点，可以拖出线连接其他节点
        this.jsPlumb.makeSource(node.id, this.jsplumbSourceOptions)
        // 设置目标点，其他源点拖出的线可以连接该节点
        this.jsPlumb.makeTarget(node.id, this.jsplumbTargetOptions)
        // jsPlumb.addEndpoint(node.id)
        // 设置可拖拽
        // jsPlumb.draggable(node.id, {
        //   containment: 'parent',
        //   grid: [10, 10]
        // })

        this.jsPlumb.draggable(node.id, {
          containment: 'parent'
        })

        // jsPlumb.draggable(node.id)
      }

      // 初始化连线
      for (var j = 0; j < this.data.lineList.length; j++) {
        let line = this.data.lineList[j]
        this.jsPlumb.connect({
          source: line.from,
          target: line.to
        }, this.jsplumbConnectOptions)
      }
      this.$nextTick(function () {
        this.loadEasyFlowFinish = true
      })
    },
    addNode (evt, nodeMenu, mousePosition) {
      let width = this.$refs.flowTool.$el.clientWidth
      if (evt.originalEvent.layerX > width) {
        const index = this.index++
        let nodeId = 'node' + index
        var left = mousePosition.left
        var top = mousePosition.top
        if (mousePosition.left < 0) {
          left = evt.originalEvent.layerX - width
        }
        if (mousePosition.top < 0) {
          top = evt.originalEvent.clientY - 100
        }
        var node = {
          id: 'node' + index,
          name: nodeMenu.name,
          left: left + 'px',
          top: top + 'px',
          ico: nodeMenu.ico,
          show: true
        }
        this.data.nodeList.push(node)
        this.$nextTick(function () {
          this.jsPlumb.makeSource(nodeId, this.jsplumbSourceOptions)
          this.jsPlumb.makeTarget(nodeId, this.jsplumbTargetOptions)
          this.jsPlumb.draggable(nodeId, {
            containment: 'parent'
          })
        })
      }
    },
    dataReload (data) {
      this.easyFlowVisible = false
      this.data.nodeList = []
      this.data.lineList = []
      this.$nextTick(() => {
        // 这里模拟后台获取数据 然后加载
        // data = lodash.cloneDeep(data)
        this.easyFlowVisible = true
        this.data = data
        this.$nextTick(() => {
          this.jsPlumb = jsPlumb.getInstance()
          this.$nextTick(() => {
            this.jsPlumbInit()
          })
        })
      })
    },
    // 数据重新载入
    dataReloadA () {
      this.dataReload(getDataA())
    },
    // 改变节点位置
    changeNodeSite (data) {
      for (var i = 0; i < this.data.nodeList.length; i++) {
        let node = this.data.nodeList[i]
        if (node.id === data.nodeId) {
          node.left = data.left
          node.top = data.top
        }
      }
    },
    dataInfo () {
      this.flowInfoVisable = true
      this.$nextTick(() => {
        this.$refs.flowInfo.init()
      })
    },
    deleteLine (from, to) {
      this.data.lineList = this.data.lineList.filter(function (line) {
        return line.from !== from && line.to !== to
      })
    },
    hasLine (from, to) {
      for (var i = 0; i < this.data.lineList.length; i++) {
        let line = this.data.lineList[i]
        if (line.from === from && line.to === to) {
          return true
        }
      }
      return false
    },
    editNode (nodeId) {
      this.nodeFormVisible = true
      this.$nextTick(function () {
        this.$refs.nodeForm.init(this.data, nodeId)
      })
    },
    deleteNode (nodeId) {
      this.$confirm(`确定要删除节点${nodeId}?`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.data.nodeList = this.data.nodeList.filter((node) => {
          return node.id !== nodeId
        })
        this.$nextTick(function () {
          this.jsPlumb.removeAllEndpoints(nodeId)
        })
      }).catch()
      return true
    },
    changeLabel () {
      var lines = this.jsPlumb.getConnections({
        source: 'nodeA',
        target: 'nodeB'
      })
      lines[0].setLabel({
        label: 'admin',
        cssClass: 'labelClass a b '
      })
    }
  },
  mounted () {
    this.jsPlumb = jsPlumb.getInstance()
    this.$nextTick(() => {
      this.dataReloadA()
    })
  }
}
</script>

<style scoped>
#flowContainer{
  height:500px;
  position: relative;
  background-color:rgb(251,251,251);
}
  .labelClass {
    background-color:white;
    padding: 5px;
    opacity:0.7;
    border:1px solid #346789;
    cursor:pointer;
    -webkit-user-select:none;
    -moz-user-select:none;
    -ms-user-select:none;
    user-select:none;
  }
</style>
