<script lang='ts'>
  import { onMount } from 'svelte'
  import cytoscape from 'cytoscape'

  let cy
  let container

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
  })
</script>

<h1>Graph Builder</h1>


<div id='container' bind:this={container}></div>


<div class="footer">
  <p>Built with <a href="https://js.cytoscape.org/">Cytoscape.js</a></p>
</div>

<style>
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
