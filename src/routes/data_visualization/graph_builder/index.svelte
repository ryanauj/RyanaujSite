<script lang='ts'>
  import { onMount } from 'svelte'
  import cytoscape from 'cytoscape'

  let cy
  let container
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

  onMount(() => {
    cy = cytoscape({
      container: container,
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
            'background-color': '#666',
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

    setNodesOutput()
    setEdgesOutput()
  })
</script>

<h1>Graph Builder</h1>


<div id='container' class='resizable' bind:this={container}></div>
<p>{nodesOutput}</p>
<p>{edgesOutput}</p>


<div class='footer'>
  <p>Built with <a href='https://js.cytoscape.org/'>Cytoscape.js</a></p>
</div>

<style>
  /*Resizeable*/

  .resizable .resizer-right {
    width: 5px;
    height: 100%;
    background: transparent;
    position: absolute;
    right: 0;
    bottom: 0;
    cursor: e-resize;
  }

  .resizable .resizer-bottom {
    width: 100%;
    height: 5px;
    background: transparent;
    position: absolute;
    right: 0;
    bottom: 0;
    cursor: n-resize;
  }

  .resizable .resizer-both {
    width: 5px;
    height: 5px;
    background: transparent;
    z-index: 10;
    position: absolute;
    right: 0;
    bottom: 0;
    cursor: nw-resize;
  }

  .footer {
    left: 0;
    bottom: 0;
    width: 100%;
    text-align: center;
  }

  #container {
    width: 300px;
    height: 300px;
    display: block;
  }
</style>
