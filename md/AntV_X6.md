
[快速入门](https://antv-x6.gitee.io/zh/docs/tutorial/getting-started)


## 安装
```shell
npm install @antv/x6 --save
```

安装完成之后，使用 import 或 require 进行引用。

```js
import { Graph } from '@antv/x6';
```


## 快速入门

使用[[vue-element-admin]]框架，添加一个模块。然后编写如下代码

```html
<template>
  <div class="app-container documentation-container">
    <div id="container" />
  </div>
</template>

<script>

import { Graph } from '@antv/x6'

export default {
  name: '',
  data() {
    return {}
  },
  mounted() {
    this.init()
  },
  methods: {
    init() {
      const graph = new Graph({
        container: document.getElementById('container'),
        width: 1000,
        height: 500,
        background: {
          color: 'rgba(184,180,180,0.2)' // 设置画布背景颜色
        },
        grid: {
          size: 10, // 网格大小 10px
          visible: true // 渲染网格背景
        }
      })
      const data = {
        // 节点
        nodes: [
          {
            id: 'node1', // String，可选，节点的唯一标识
            x: 40, // Number，必选，节点位置的 x 值
            y: 40, // Number，必选，节点位置的 y 值
            width: 80, // Number，可选，节点大小的 width 值
            height: 40, // Number，可选，节点大小的 height 值
            label: '普线' // String，节点标签
          },
          {
            id: 'node2', // String，节点的唯一标识
            x: 160, // Number，必选，节点位置的 x 值
            y: 80, // Number，必选，节点位置的 y 值
            width: 80, // Number，可选，节点大小的 width 值
            height: 40, // Number，可选，节点大小的 height 值
            label: '夜间' // String，节点标签
          },
          {
            id: 'node3', // String，节点的唯一标识
            x: 160, // Number，必选，节点位置的 x 值
            y: 180, // Number，必选，节点位置的 y 值
            width: 80, // Number，可选，节点大小的 width 值
            height: 40, // Number，可选，节点大小的 height 值
            attrs: {
              body: {
                fill: '#fbca00',
                stroke: '#000',
                strokeDasharray: '10,2'
              },
              label: {
                text: '白金',
                fill: '#333',
                fontSize: 13
              }
            }
          }
        ],
        // 边
        edges: [
          {
            source: 'node1', // String，必须，起始节点 id
            target: 'node2' // String，必须，目标节点 id
          },
          {
            source: 'node1', // String，必须，起始节点 id
            target: 'node3' // String，必须，目标节点 id
          }
        ]
      }
      graph.fromJSON(data)
    }
  }

}

</script>

<style lang="scss" scoped>
  .documentation-container {
    margin: 50px;
    display: flex;
    flex-wrap: wrap;
    justify-content: flex-start;

    .document-btn {
      flex-shrink: 0;
      display: block;
      cursor: pointer;
      background: black;
      color: white;
      height: 60px;
      padding: 0 16px;
      margin: 16px;
      line-height: 60px;
      font-size: 20px;
      text-align: center;
    }
  }
</style>


```

![[Pasted image 20210929003153.png]]

## 泛型

```js
import {Base} from '@antv/x6/src/shape/base.ts'
```


## 一些示例

当使用dnd拖拽生成控件时，默认会clone node，对于自定义node的一些metadata无法被clone，需要重写clone方法

```js
import {Cell,Edge,Graph,Node,Shape} from '@antv/x6'
class Group extends Shape.Rect{

	clone(options?:Cell.CloneOptions):this extends Node? Node:(this extends Edge ? Edge:Cell){
		const clone = super.clone()
		//对于自定义组件这样才能将metadata传递给clone后的对象
		clone.store.data = this.store.datja
		return clone;
	}
}

const metadata = {
	x:20,
	y:20,
	width:100,
	height:40,
	attrs:{
		body:{fill},
		label:{
			text:'123',
			fill:'#000',
			fontSize:16
		}
	},
	data:{
		parent:true
	}
}
const group = new Group(metadata)

```

通过查看其源码，在使用dnd拖拽控件时，当drop node后，会在当前画布上触发`node:embedded`事件，我们可以通过监听该事件，修改zindex等一些操作

```js
graph.on('node:embedded',function({e,cell,x,y,node,view,previousParent,currentParent}){
	
})
```

源码大致分析
`@antv/x6/src/addon/dnd/index.ts`的`onDragEnd`方法内
```js
protected onDragEnd(evt: JQuery.MouseUpEvent) {  
  const draggingNode = this.draggingNode  
 if (draggingNode) {  
    const e = this.normalizeEvent(evt)  
    const draggingView = this.draggingView  
 const draggingBBox = this.draggingBBox  
 const snapOffset = this.snapOffset  
 let x = draggingBBox.x  
 let y = draggingBBox.y  
  
 if (snapOffset) {  
      x += snapOffset.x  
 y += snapOffset.y  
 }  
  
    draggingNode.position(x, y, { silent: true })  
  
    const ret = this.drop(draggingNode, { x: e.clientX, y: e.clientY })  
    const callback = (node: null | Node) => {  
      if (node) {  
        this.onDropped(draggingNode)  
        if (this.targetGraph.options.embedding.enabled && draggingView) {  
          draggingView.setEventData(e, {  
            cell: node,  
            graph: this.targetGraph,  
            candidateEmbedView: this.candidateEmbedView,  
          })  
          draggingView.finalizeEmbedding(e, draggingView.getEventData<any>(e))  
        }  
      } else {  
        this.onDropInvalid()  
      }  
  
      this.candidateEmbedView = null  
 this.targetModel.stopBatch('dnd')  
    }  
  
    if (FunctionExt.isAsync(ret)) {  
      // stop dragging  
 this.undelegateDocumentEvents()  
      ret.then(callback) // eslint-disable-line  
 } else {  
      callback(ret)  
    }  
  }  
}
```

finalizeEmbedding方法内部
```js
finalizeEmbedding(e: JQuery.MouseUpEvent, data: EventData.MovingTargetNode) {  
  const cell = data.cell || this.cell  
 const graph = data.graph || this.graph  
 const view = graph.findViewByCell(cell)  
  const parent = cell.getParent()  
  const candidateView = data.candidateEmbedView  
 if (candidateView) {  
    // Candidate view is chosen to become the parent of the node.  
 candidateView.unhighlight(null, { type: 'embedding' })  
    data.candidateEmbedView = null  
 if (parent == null || parent.id !== candidateView.cell.id) {  
      candidateView.cell.insertChild(cell, undefined, { ui: true })  
    }  
  } else if (parent) {  
    parent.unembed(cell, { ui: true })  
  }  
  
  graph.model.getConnectedEdges(cell, { deep: true }).forEach((edge) => {  
    edge.updateParent({ ui: true })  
  })  
  
  const localPoint = graph.snapToGrid(e.clientX, e.clientY)  
  
  if (view) {  
	//触发了事件，我们只要订阅该事件即可
    view.notify('node:embedded', {  
      e,  
      cell,  
      x: localPoint.x,  
      y: localPoint.y,  
      node: cell,  
      view: graph.findViewByCell(cell),  
      previousParent: parent,  
      currentParent: cell.getParent(),  
    })  
  }  
}
```