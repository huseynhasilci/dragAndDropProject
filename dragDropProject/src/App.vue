<script setup>
import { VueFlow, useVueFlow } from '@vue-flow/core'
import { nextTick, watch } from 'vue'
import Sidebar from './components/Sidebar.vue';
import { Background, BackgroundVariant } from '@vue-flow/background'
import { MiniMap } from '@vue-flow/minimap'
let id = 0
const getId = () => `dndnode_${id++}`

const { findNode, onConnect, nodes, edges, addEdges, addNodes, viewport, project, vueFlowRef } = useVueFlow({
  nodes: [
    {
      id: '1',
      type: 'input',
      label: 'input node',
      position: { x: 250, y: 25 },
    },
  ],
})

const onDragOver = (event) => {
  event.preventDefault()

  if (event.dataTransfer) {
    event.dataTransfer.dropEffect = 'move'
  }
}

onConnect((params) => addEdges([params]))

const onDrop = (event) => {
  const type = event.dataTransfer?.getData('application/vueflow')

  const { left, top } = vueFlowRef.value.getBoundingClientRect()

  const position = project({
    x: event.clientX - left,
    y: event.clientY - top,
  })

  const newNode = {
    id: getId(),
    type,
    position,
    label: `${type} node`,
  }

  addNodes([newNode])

  // align node position after drop, so it's centered to the mouse
  nextTick(() => {
    const node = findNode(newNode.id)
    console.log(node.computedPosition); // burası node'un kordinatlarını veriyor z ile birlikte
    console.log(node.data); // içerisindeki değerleri verir
    node.data = "dumenden";
    console.log(node.position); // buda pozisyonu verir
    console.log(node.value);
    const stop = watch(
      () => node.dimensions,
      (dimensions) => {
        if (dimensions.width > 0 && dimensions.height > 0) {
          node.position = { x: node.position.x - node.dimensions.width / 2, y: node.position.y - node.dimensions.height / 2 }
          stop()
        }
      },
      { deep: true, flush: 'post' },
    )
  })
}
</script>

<template>
  <div class="dndflow" @drop="onDrop">
    <VueFlow @dragover="onDragOver">
      <MiniMap :pannable=true :zoomable=true />

      <Background :variant="BackgroundVariant.Lines" :bgColor="CD5C5C" />
    </VueFlow>
    
    <Sidebar />
  </div>
</template>
