<script lang='ts'>
  console.log('script running')

  import TextArea from '../../../components/TextAreaAutosize.svelte'
  import { onMount } from 'svelte'
  import cytoscape from 'cytoscape'
  // For some reason dagre is causing an error when compiled:
  // Cannot read properties of undefined (reading 'Graph')
  // Omitting for now.
  import dagre from 'cytoscape-dagre'
  import cola from 'cytoscape-cola'
  import klay from 'cytoscape-klay'
  import elk from 'cytoscape-elk'
  import { v4 as uuidv4 } from 'uuid'

  let cy
  let graph
  let nodesOutput
  let edgesOutput
  let jsonElementsText
  let fitToViewport = true
  let selectedNode = null
  const layoutTypes = [
    'null',
    'random',
    'preset',
    'grid',
    'circle',
    'concentric',
    'breadthfirst',
    'cose',
    'dagre',
    'cola',
    'klay',
    'elk'
  ]
  let selectedLayoutType = layoutTypes[layoutTypes.length-1]

  const updateLayout = () => {
    console.log('Layout Run')
    const layout = cy.layout({
      name: selectedLayoutType,
      fit: fitToViewport
    })
    layout.run()
    console.log(layout)
  }

  const setNodesOutput = () => {
    const nodes = cy.json().elements?.nodes?.map(n => n.data)
    nodesOutput = JSON.stringify(nodes)
  }

  const setEdgesOutput = () => {
    const edges = cy.json().elements?.edges?.map(e => e.data)
    edgesOutput = JSON.stringify(edges)
  }

  const setJsonElementsText = () => {
    let jsonElements = cy.json().elements ?? {}
    jsonElements.nodes = jsonElements.nodes?.map(n => n.data)
    jsonElements.edges = jsonElements.edges?.map(n => n.data)
    jsonElementsText = JSON.stringify(jsonElements, null, 2)
  }

  const refreshJson = () => {
    console.log(cy.json())
    setNodesOutput()
    setEdgesOutput()
    setJsonElementsText()
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

  const setElements = () => {
    // https://stackoverflow.com/questions/35770055/replace-all-elements-and-redraw-graph-softly-in-cytoscape-js
    const elements = JSON.parse(jsonElementsText)
    if (elements === null) {
      return
    }
    cy.elements().remove()
    if (elements.nodes !== null) {
      const nodes = elements.nodes.map(node => ({ data: { ...node }, group: 'nodes' }))
      console.log('nodes', nodes)
      cy.add(nodes)
    }
    refreshJson()
    if (elements.edges !== null) {
      const edges = elements.edges.map(edge => ({ data: { ...edge }, group: 'edges' }))
      console.log('edges', edges)
      cy.add(edges)
    }
    updateLayout()
    refreshJson()
  }

  onMount(() => {
    console.log('onMount')
    cytoscape.use(dagre)
    cytoscape.use(cola)
    cytoscape.use(klay)
    cytoscape.use(elk)

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
        nameVal = `${nameCount}`
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

    const add = (event, keepSelection=false) => {
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

        if (keepSelection) {
          // If a new node was added, we want to set this as selected so we can easily add an edge from it.
          selectNode(getNode(addedId))
        }

      } else if (target.isNode()) {
        const node = target

        // If the target is the selected node, then we can unselect this node
        if (selectedNode === node || selectedNode?.id() === node?.id()) {
          unselectNode()
        }
        // If another node is selected, we can connect that selected node to the tapped node
        else if (selectedNode !== null) {
          addEdge(selectedNode.id(), node.id())
          unselectNode()

          if (keepSelection) {
            // We want to set this as selected so we can easily add an edge from it.
            selectNode(node)
          }
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
    }

    const remove = (event) => {
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
    }

    let tapholdOccurred = false 

    cy.on('tap', event => {
      console.log('tap')
      if (!tapholdOccurred) {
        add(event)
      }
      tapholdOccurred = false
    })

    cy.on('dbltap', event => {
      console.log('dbltap')
      remove(event)
    })

    cy.on('taphold', event => {
      console.log('taphold')
      tapholdOccurred = true
      add(event, true)
    })

    cy.on('cxttap', event => {
      console.log('cxttap')
      add(event, true)
    })

    refreshJson()
  })
</script>

<main class='container'>
  <h1>Graph Builder</h1>

  <div id='graph' bind:this={graph}></div>

  <div>
    <h5>Selected Node</h5>
    <input
      type="text"
      disabled={selectedNode === null}
      on:input={onSelectedNameChange}
      value={selectedNode?.data()?.name ?? ''}>
  </div>

  <div class="section">
    <fieldset>
      <legend>
        <h4>Layout Options</h4>
      </legend>

      <div class="subsection">
        <label>Types</label>
        <select bind:value={selectedLayoutType}>
          {#each layoutTypes as layoutType}
            <option value={layoutType}>
              {layoutType}
            </option>
          {/each}
        </select>
      </div>

      <div class="subsection">
        <label>
          <input type="checkbox" bind:checked={fitToViewport}>
          Fit To Viewport
        </label>
      </div>

      <div class="extra-space">
        <button on:click={updateLayout}>Organize Layout</button>
      </div>
    </fieldset>
  </div>

  <div class="section">
    <fieldset>
      <legend>
        <h4 class="slim-bottom">Edit JSON</h4>
      </legend>
      <TextArea
        bind:value={jsonElementsText}
        minRows={4}
        maxRows={40}/>
      <button on:click={setElements}>Update Elements</button>
    </fieldset>
  </div>

  <div class="section">
    <fieldset>
      <legend>
        <h4 class="slim-bottom">Legend</h4>
      </legend>
      <p>Blue means a node is selected, and grey means a node is not selected.</p>
      <p>Tap an empty space to create a node.</p>
      <p>Tap a selected node to deselect it.</p>
      <p>If a node is selected, tap another node to create an edge from the selected node to that, or tap an empty space to create a node at that position and select that.</p>
      <p>Double tap a node or edge to delete it.</p>
      <p>Tap with two fingers, tap and hold, or right click to create a node and keep it selected.</p>
    </fieldset>
  </div>

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

  code {
    display: block;
    padding: 10px 5px;
    margin-bottom: 5px;
    background-color: #eee;
  }

  legend {
    padding: 0 3px;
  }

  .slim-bottom {
    padding-bottom: 0px;
    margin-bottom: 0px;
    margin-top: 0px;
  }

  .extra-space {
    padding: 10px 0;
  }

  .section {
    padding: 10px 0;
  }

  .subsection {
    padding: 2px 0;
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
