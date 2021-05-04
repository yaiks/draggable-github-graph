<template>
  <svg id="container" viewBox="0 0 500 500">
    <g>
      <line class="link" v-for="(link, index) in links" :key="index" />
    </g>
    <g>
      <g class="node" v-for="node in nodes" :key="node.id">
        <circle class="node-circle" :class="'node-' + node.type" r="12" />
        <text class="node-text" x="15" y="3">{{ node.id }}</text>
      </g>
    </g>
  </svg>
</template>

<script>
import * as d3 from 'd3'

export default {
  name: 'Index',
  data() {
    return {
      width: 500,
      height: 500,
      nodes: [
        { id: 'FatSmaug', type: 'backend' },
        { id: 'DonCorleone', type: 'frontend' },
        { id: 'Albundy', type: 'frontend' },
        { id: 'Gandalf', type: 'backend' },
      ],
      links: [
        { source: 'Gandalf', target: 'DonCorleone' },
        { source: 'Gandalf', target: 'Albundy' },
        { source: 'Gandalf', target: 'FatSmaug' },
      ],
    }
  },
  mounted() {
    this.generateGraph()
  },
  methods: {
    generateGraph() {
      const svg = d3.select('#container')
      const node = svg.selectAll('.node').data(this.nodes).join('g')

      const link = svg.selectAll('.link').data(this.links).join('line')

      const simulation = d3
        .forceSimulation()
        .nodes(this.nodes)
        .force('charge', d3.forceManyBody())
        .force('center', d3.forceCenter(this.width / 2, this.height / 2))
        .force(
          'link',
          d3
            .forceLink(this.links)
            .id((d) => d.id)
            .distance(60)
        )
        .on('tick', tick)

      const drag = d3
        .drag()
        .on('start', dragstarted)
        .on('drag', dragged)
        .on('end', dragended)

      node.call(drag).on('click', click)

      function tick() {
        link
          .attr('x1', (d) => d.source.x)
          .attr('y1', (d) => d.source.y)
          .attr('x2', (d) => d.target.x)
          .attr('y2', (d) => d.target.y)

        node.attr('transform', (d) => `translate(${d.x},${d.y})`)
        // node.attr('cx', (d) => d.x).attr('cy', (d) => d.y)
      }

      function click(event, d) {
        delete d.fx
        delete d.fy
        simulation.alpha(1).restart()
      }

      function dragstarted(event, d) {
        if (!event.active) simulation.alphaTarget(0.3).restart()
        d.fx = d.x
        d.fy = d.y
      }

      function dragged(event, d) {
        d.fx = event.x
        d.fy = event.y
      }

      function dragended(event, d) {
        if (!event.active) simulation.alphaTarget(0)
        d.fx = null
        d.fy = null
      }
    },
  },
}
</script>

<style>
#container {
  width: 100vw;
  height: 100vh;
}

.link {
  stroke: #000;
  stroke-width: 1.5px;
}

.node-circle {
  cursor: move;
  fill: #ccc;
  stroke: #000;
  stroke-width: 1.5px;
}

.node-text {
  font-size: 12px;
}

.node-frontend {
  fill: darkslategrey;
}

.node-backend {
  fill: darkorchid;
}
</style>
