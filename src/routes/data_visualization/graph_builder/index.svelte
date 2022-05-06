<script lang='ts'>
  import { onMount } from 'svelte'
  import cytoscape from 'cytoscape'
  import { v4 as uuidv4 } from 'uuid'

  let cy
  let graph
  let nodesOutput
  let edgesOutput

  const setNodesOutput = () => {
    const nodes = cy.json().elements.nodes.map(n => n.data)
    nodesOutput = JSON.stringify(nodes)
  }

  const setEdgesOutput = () => {
    const edges = cy.json().elements.edges.map(e => e.data)
    edgesOutput = JSON.stringify(edges)
  }

  const refreshJson = () => {
    setNodesOutput()
    setEdgesOutput()
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
        },
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

    const getNode = (id: string) => cy.$(`#${id}`)

    const selectNode = (node) => {
      if (selectedId !== null) {
        getNode(selectedId).style('background-color', defaultNodeColor)
      }
      node.style('background-color', selectedNodeColor)
      selectedId = node.id()
    }

    const unselectNode = (node) => {
      node.style('background-color', defaultNodeColor)
      selectedId = null
    }

    const addNode = (id: string, position: { x: number, y: number } = null) => {
      cy.add({
        group: 'nodes',
        data: { id },
        position
      })
    }

    const addEdge = (sourceId: string, targetId: string): string => {
      const id = uuidv4()

      cy.add({
        group: 'edges',
        data: {
          id,
          source: sourceId,
          target: targetId
        }
      })

      return id
    }

    cy.on('tap', event => {
      const target = event.target
      if (target === cy) {
        const addedId = uuidv4()

        // Add a node at the position of the tap
        addNode(addedId, event.position)

        // If there is a selected element, we want to add an edge from there to this element
        if (selectedId !== null) {
          addEdge(selectedId, addedId)
          // Once we have added an edge, we want to clear the selection
          unselectNode(getNode(selectedId))
        }

        // We want to set this as selected so we can easily add an edge from it.
        selectNode(getNode(addedId))

      } else if (target.isNode()) {
        const node = target

        // If the target is the selected node, then we can unselect this node
        if (selectedId === node.id()) {
          unselectNode(node)
        }
        // If another node is selected, we can connect that selected node to the tapped node
        else if (selectedId !== null) {
          addEdge(selectedId, node.id())
          unselectNode(getNode(selectedId))
          // We want to set this as selected so we can easily add an edge from it.
          selectNode(node)
        }
        // If no other node was selected, we want to select the tapped node
        else {
          selectNode(node)
        }
      }

      // We want to refresh the json after making changes
      // (I realize that this could refresh when nothing changes atm,
      //   i.e. on selection. Ok with that for now)
      refreshJson()
    })

    cy.on('dbltap', event => {
      const target = event.target
      // If target is a node or an edge, we want to remove it
      if (target !== cy  && (target.isNode() || target.isEdge())) {
        cy.remove(target)
      }

      // We want to refresh the json after making changes
      // (I realize that this could refresh when nothing changes atm,
      //   i.e. on selection. Ok with that for now)
      refreshJson()
    })

    refreshJson()
  })
</script>

<main class='container'>
  <h1>Graph Builder</h1>
  

  <div id='graph' bind:this={graph}></div>
  <h4>Nodes</h4>
  <p>{nodesOutput}</p>
  <h4>Edges</h4>
  <p>{edgesOutput}</p>

  <fieldset>
    <legend>Legend</legend>
    <p>Blue means a node is selected, and grey means a node is not selected.</p>
    <p>Tap an empty space to create a node.</p>
    <p>Tap a selected node to deselect it.</p>
    <p>If a node is selected, tap another node to create an edge from the selected node to that, or tap an empty space to create a node at that position and select that.</p>
    <p>Double tap a node or edge to delete it.</p>
  </fieldset>

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
