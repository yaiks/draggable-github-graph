<template>
  <svg id="container" viewBox="0 0 500 500">
    <g class="container-group">
      <g>
        <line class="link" v-for="(link, index) in links" :key="index" />
      </g>
      <g>
        <g class="node" v-for="node in nodes" :key="node.id">
          <circle :data-id="node.id" :data-size="node.size" class="node-circle" :class="'node-' + node.type" r="12" />
          <text class="node-text" :x="node.size === 'big' ? 12 : 8" y="3">{{ node.id }}</text>
        </g>
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
        { id: 'DonCorleone', type: 'frontend', size: 'regular' },
        { id: 'Albundy', type: 'frontend', size: 'regular' },
        { id: 'BilboBaggins', type: 'frontend', size: 'regular' },
        { id: 'FatSmaug', type: 'backend', size: 'big' },
        { id: 'Gandalf', type: 'backend', size: 'big' },
        { id: 'Odin', type: 'backend', size: 'regular' },
        { id: 'ListingsAPI', type: 'backend', size: 'regular' },
        { id: 'MrBurns', type: 'backend', size: 'regular' },
        { id: 'AccountAPI', type: 'backend', size: 'regular' },
        { id: 'Wally', type: 'backend', size: 'regular' },
        { id: 'CallOfCthulhu', type: 'backend', size: 'regular' },
        { id: 'Chirrin', type: 'backend', size: 'regular' },
        { id: 'CiroBottini', type: 'backend', size: 'regular' },
        { id: 'Concierge', type: 'backend', size: 'regular' },
        { id: 'Druid', type: 'backend', size: 'regular' },
        { id: 'Feedex', type: 'backend', size: 'regular' },
        { id: 'Feeds', type: 'backend', size: 'regular' },
        { id: 'SaulGoodman', type: 'backend', size: 'regular' },
        { id: 'Sauron', type: 'backend', size: 'regular' },
        { id: 'Payments', type: 'backend', size: 'regular' },
        { id: 'Search', type: 'backend', size: 'regular' },
        { id: 'Silvio', type: 'backend', size: 'regular' },
        { id: 'ZapFin', type: 'backend', size: 'regular' },
      ],
      links: [
        { source: 'Gandalf', target: 'DonCorleone' },
        { source: 'Gandalf', target: 'Albundy' },
        { source: 'Gandalf', target: 'BilboBaggins' },
        { source: 'Gandalf', target: 'FatSmaug' },
        { source: 'Gandalf', target: 'ListingsAPI' },
        { source: 'Gandalf', target: 'CallOfCthulhu' },
        { source: 'Gandalf', target: 'Chirrin' },
        { source: 'Gandalf', target: 'CiroBottini' },
        { source: 'Gandalf', target: 'Concierge' },
        { source: 'Gandalf', target: 'Druid' },
        { source: 'Gandalf', target: 'Feedex' },
        { source: 'Gandalf', target: 'Feeds' },
        { source: 'Gandalf', target: 'SaulGoodman' },
        { source: 'Gandalf', target: 'Sauron' },
        { source: 'Gandalf', target: 'Payments' },
        { source: 'Gandalf', target: 'Search' },
        { source: 'Gandalf', target: 'Silvio' },
        { source: 'Gandalf', target: 'ZapFin' },
        { source: 'FatSmaug', target: 'Odin' },
        { source: 'Gandalf', target: 'ListingsAPI' },
        { source: 'FatSmaug', target: 'ListingsAPI' },
        { source: 'FatSmaug', target: 'MrBurns' },
        { source: 'FatSmaug', target: 'AccountAPI' },
        { source: 'ListingsAPI', target: 'Wally' },
      ],
    }
  },
  mounted() {
    this.generateGraph()
  },
  methods: {
    generateGraph() {
      /**
       * DOM selectors
       */
      const svg = d3.select('#container')
      const container_group = d3.select('.container-group')
      const node = svg.selectAll('.node').data(this.nodes).join('g')
      const circle = d3.selectAll(".node-circle").on('click', highlightRelatedNodes)
      const link = svg.selectAll('.link').data(this.links).join('line')

      /**
       * Select and highlight related nodes
       */
      let toggle = 0

      const getRelatedNodes = (circle_id) => {
        return this.links.map(each => {
          if (each.source.id === circle_id) {
            return each.target.id 
          }

          if (each.target.id === circle_id) {
            return each.source.id
          }
        })
      }

      function highlightRelatedNodes(event) {
        if (toggle === 0) {
          const circle_id = event.target.dataset.id

          link.style('stroke-opacity', (l) => {
            return l.target.id == circle_id || l.source.id == circle_id ? 1 : 0.2;
          });

          node.style('opacity', (n) => {
            const related_nodes = getRelatedNodes(circle_id)
            return related_nodes.includes(n.id) || circle_id === n.id ? 1 : 0.2
          })

          toggle = 1
        } else {
          link.style('stroke-opacity', 1)
          node.style('opacity', 1)
          toggle = 0
        }
      }

      /**
       * D3 simulation
       */
      const simulation = d3
        .forceSimulation()
        .nodes(this.nodes)
        .force('charge', d3.forceManyBody().strength(-200))
        .force('collide', d3.forceCollide(30))
        .force('center', d3.forceCenter(this.width / 2, this.height / 2))
        .force(
          'link',
          d3
            .forceLink(this.links)
            .id((d) => d.id)
            .distance(100)
        )
        .on('tick', tick)

      function tick() {
        link
          .attr('x1', (d) => d.source.x)
          .attr('y1', (d) => d.source.y)
          .attr('x2', (d) => d.target.x)
          .attr('y2', (d) => d.target.y)

        node.attr('transform', (d) => `translate(${d.x},${d.y})`)
        // node.attr('cx', (d) => d.x).attr('cy', (d) => d.y)
      }

      /**
       * Zoomable svg
       */
      let transform

      const zoom = d3.zoom().on("zoom", e => {
        container_group.attr("transform", (transform = e.transform));
        container_group.style("stroke-width", 3 / Math.sqrt(transform.k));

        circle._groups[0].forEach(each => {
          if (each.dataset.size === 'big') {
            each.setAttribute('r', 10)
          } else {
            each.setAttribute('r', 6 / Math.sqrt(transform.k))
          }
        })
      });

      svg.call(zoom).call(zoom.transform, d3.zoomIdentity)

      /**
       * Draggable nodes
       */
      const drag = d3
        .drag()
        .on('start', dragstarted)
        .on('drag', dragged)
        .on('end', dragended)

      node.call(drag).on('click', click)

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
  font-size: 10px;
}

.node-frontend {
  fill: rgb(107, 158, 235);
}

.node-backend {
  fill: darkorchid;
}
</style>
