<script lang='ts'>
  import { onMount } from 'svelte'
  import cytoscape from 'cytoscape'

  let cy
  let graph
  let nodesOutput
  let edgesOutput

  const setNodesOutput = () => {
    const nodes = cy.json().elements.nodes.map(n => n.data)
    nodesOutput = JSON.stringify(nodes)
  }

  const setEdgesOutput = (json) => {
    const edges = cy.json().elements.edges.map(e => e.data)
    edgesOutput = JSON.stringify(edges)
  }

  const defaultNodeColor = '#666'
  const selectedNodeColor = 'blue'

  onMount(() => {
    cy = cytoscape({
      container: graph,
      elements: [
        {
          data: { id: 'a' }
        },
        {
          data: { id: 'b' }
        },
        {
          data: { id: 'ab', source: 'a', target: 'b' }
        }
      ],
      style: [
        {
          selector: 'node',
          style: {
            'background-color': defaultNodeColor,
            'label': 'data(id)'
          }
        },
        {
          selector: 'edge',
          style: {
            'width': 3,
            'line-color': '#ccc',
            'target-arrow-color': '#ccc',
            'target-arrow-shape': 'triangle',
            'curve-style': 'bezier'
          }
        }
      ],
      layout: {
        name: 'grid',
        rows: 1
      }
    })

    let selectedId = null

    cy.removeAllListeners('tap')

    cy.on('dbltap', event => {
      const target = event.target
      if (target === cy) {
        console.log('tap on background')
      } else {
        console.log(`tap on element`)
        const id = target.id()
        const isNode = target.isNode() ? 'is a node' : 'is not a node'
        console.log(`${id} ${isNode}`)
        if (isNode) {
          const node = target
          let isSelectedText = null

          const isSelected = selectedId === id

          if (isSelected) {
            node.style('background-color', defaultNodeColor)
            selectedId = null
            isSelectedText = 'is not selected'
          }
          else {
            if (selectedId !== null) {
              cy.$(`#${selectedId}`).style('background-color', defaultNodeColor)
            }
            node.style('background-color', selectedNodeColor)
            selectedId = id
            isSelectedText = 'is selected'
          }

          console.log(`node ${id} ${isSelectedText}`)

          console.log(node.classes())
          console.log(node.style())
        }
      }
    })

    setNodesOutput()
    setEdgesOutput()
  })
</script>

<main class='container'>
  <h1>Graph Builder</h1>

  <div id='graph' bind:this={graph}></div>
  <p>{nodesOutput}</p>
  <p>{edgesOutput}</p>

</main>

<footer>
  <p>Built with <a href="https://js.cytoscape.org/">Cytoscape.js</a></p>
</footer>

<style>
  footer {
    left: 0;
    bottom: 0;
    width: 100%;
    text-align: center;
  }

  .container {
    width: 80%;
    margin: auto;
  }

  #graph {
    width: 100%;
    height: 80vh;
    display: block;
    cursor: pointer;
    border: 1px solid black;
  }
</style>
