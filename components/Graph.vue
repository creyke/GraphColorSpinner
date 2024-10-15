<template>
  <div>
    <canvas
      ref="canvas"
      :width="width"
      :height="height"
      @click="handleCanvasClick"
    ></canvas>
    <div class="controls">
      <button @click="rollDice">Roll Dice</button>
      <p v-if="rolledColor">
        Rolled Color: <span :style="{ color: rolledColor }">{{ rolledColor }}</span>
      </p>
    </div>
  </div>
</template>

<script setup>
import { ref, reactive, onMounted } from 'vue'

const canvas = ref(null)
const width = 800
const height = 600

const nodeCount = 20
const colors = ['red', 'green', 'blue', 'yellow']

// Node structure: { x, y, color }
const nodes = reactive([])

// Selected node index
const selectedNodeIndex = ref(null)

// Rolled color
const rolledColor = ref(null)

// Edge structure: { from: nodeIndex, to: nodeIndex }
const edges = []

function getRandomInt(min, max) {
  return Math.floor(Math.random() * (max - min + 1)) + min
}

function generateNodes() {
  for (let i = 0; i < nodeCount; i++) {
    const node = {
      x: getRandomInt(50, width - 50),
      y: getRandomInt(50, height - 50),
      color: colors[getRandomInt(0, colors.length - 1)],
    }
    nodes.push(node)
  }
}

function generateEdges() {
  const edgeCount = getRandomInt(nodeCount, nodeCount * 2)
  for (let i = 0; i < edgeCount; i++) {
    let from = getRandomInt(0, nodeCount - 1)
    let to = getRandomInt(0, nodeCount - 1)
    while (to === from) {
      to = getRandomInt(0, nodeCount - 1)
    }
    edges.push({ from, to })
  }
}

function draw() {
  const ctx = canvas.value.getContext('2d')
  ctx.clearRect(0, 0, width, height)
  drawEdges(ctx)
  drawNodes(ctx)
}

function drawNodes(ctx) {
  nodes.forEach((node, index) => {
    ctx.beginPath()
    ctx.arc(node.x, node.y, 10, 0, Math.PI * 2)
    ctx.fillStyle = node.color
    ctx.fill()
    ctx.lineWidth = selectedNodeIndex.value === index ? 3 : 1
    ctx.strokeStyle = selectedNodeIndex.value === index ? '#000' : '#fff'
    ctx.stroke()
    ctx.closePath()
  })
}

function drawEdges(ctx) {
  ctx.strokeStyle = '#aaa'
  edges.forEach((edge) => {
    const fromNode = nodes[edge.from]
    const toNode = nodes[edge.to]
    ctx.beginPath()
    ctx.moveTo(fromNode.x, fromNode.y)
    ctx.lineTo(toNode.x, toNode.y)
    ctx.stroke()
    ctx.closePath()
  })
}

function handleCanvasClick(event) {
  const rect = canvas.value.getBoundingClientRect()
  const x = event.clientX - rect.left
  const y = event.clientY - rect.top

  // Determine if a node was clicked
  for (let i = 0; i < nodes.length; i++) {
    const node = nodes[i]
    const distance = Math.hypot(node.x - x, node.y - y)
    if (distance <= 10) {
      selectedNodeIndex.value = i
      draw()
      break
    }
  }
}

function rollDice() {
  if (selectedNodeIndex.value === null) {
    alert('Please select a node first!')
    return
  }
  // Randomly select a color
  rolledColor.value = colors[getRandomInt(0, colors.length - 1)]
  moveToNearestNodeOfColor(rolledColor.value)
}

function moveToNearestNodeOfColor(color) {
  const currentNode = nodes[selectedNodeIndex.value]
  let nearestNode = null
  let minDistance = Infinity

  nodes.forEach((node, index) => {
    if (node.color === color && index !== selectedNodeIndex.value) {
      const distance = Math.hypot(node.x - currentNode.x, node.y - currentNode.y)
      if (distance < minDistance) {
        minDistance = distance
        nearestNode = index
      }
    }
  })

  if (nearestNode !== null) {
    selectedNodeIndex.value = nearestNode
    draw()
  } else {
    alert(`No nodes of color ${color} found.`)
  }
}

onMounted(() => {
  generateNodes()
  generateEdges()
  draw()
})
</script>

<style scoped>
canvas {
  border: 1px solid #ccc;
  display: block;
  margin: 0 auto;
}
.controls {
  text-align: center;
  margin-top: 10px;
}
button {
  padding: 10px 20px;
  font-size: 16px;
}
p {
  font-size: 18px;
}
</style>
