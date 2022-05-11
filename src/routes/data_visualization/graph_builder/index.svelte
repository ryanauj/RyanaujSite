<script lang='ts'>
  import { onMount } from 'svelte'
  import cytoscape from 'cytoscape'
  import { v4 as uuidv4 } from 'uuid'

  let cy
  let graph
  let nodesOutput
  let edgesOutput
  let selectedNode = null

  const setNodesOutput = () => {
    const nodes = cy.json().elements?.nodes?.map(n => n.data)
    nodesOutput = JSON.stringify(nodes)
  }

  const setEdgesOutput = () => {
    const edges = cy.json().elements?.edges?.map(e => e.data)
    edgesOutput = JSON.stringify(edges)
  }

  const refreshJson = () => {
    setNodesOutput()
    setEdgesOutput()
  }

  const defaultNodeColor = '#666'
  const selectedNodeColor = 'blue'

  const getNode = (id: string) => cy.$(`#${id}`)

  let nameCount = 3

  const onSelectedNameChange = (event) => {
    const updatedName = event?.srcElement?.value
    selectedNode.data('name', updatedName)
    refreshJson()
  }

  onMount(() => {
    cy = cytoscape({
      container: graph,
      elements: [
        {
          data: { id: 'a', name: '1' }
        },
        {
          data: { id: 'b', name: '2' }
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
            'label': ele => {
              // Use the name property if it is set
              return ele?.data()?.name
            }
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

    const selectNode = (node) => {
      if (selectedNode !== null) {
        selectedNode.style('background-color', defaultNodeColor)
      }
      node.style('background-color', selectedNodeColor)
      selectedNode = node
    }

    const unselectNode = () => {
      if (selectedNode === null) {
        return
      }
      selectedNode.style('background-color', defaultNodeColor)
      selectedNode = null
    }

    const addNode = (
      id: string,
      position: { x: number, y: number } = null,
      name: string = null
    ) => {
      let nameVal = name
      if (nameVal === null) {
        nameVal = nameCount
        nameCount += 1
      }
      cy.add({
        group: 'nodes',
        data: { id, name: nameVal },
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
        if (selectedNode !== null) {
          addEdge(selectedNode.id(), addedId)
          // Once we have added an edge, we want to clear the selection
          unselectNode()
        }

        // We want to set this as selected so we can easily add an edge from it.
        selectNode(getNode(addedId))

      } else if (target.isNode()) {
        const node = target

        // If the target is the selected node, then we can unselect this node
        if (selectedNode === node) {
          unselectNode()
        }
        // If another node is selected, we can connect that selected node to the tapped node
        else if (selectedNode !== null) {
          addEdge(selectedNode.id(), node.id())
          unselectNode()
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
        if (selectedNode !== null) {
          unselectNode()
        }
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

  {#if selectedNode !== null}
    <h5>Name</h5>
    <input on:input={onSelectedNameChange} type="text" value={selectedNode?.data()?.name}>
  {/if}
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
