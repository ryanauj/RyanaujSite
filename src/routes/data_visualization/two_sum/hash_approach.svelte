<script lang='ts'>
  let seen: Map<number, number> = new Map<number, number>()
  let seenText: string = JSON.stringify(Object.fromEntries(seen))
  let i: number = 0
  let complete: boolean = false

  let numsText: string = '[1, 2, 3]'
  let nums: number[] = JSON.parse(numsText)
  let sum: number = 3
  let result: [number, number] = null
  let error: string = null

  const updateSeen = () => {
    seen = seen
    seenText = JSON.stringify(Object.fromEntries(seen))
  }

  const step = () => {
    if (complete) {
      seen = new Map<number, number>()
      updateSeen()
      i = 0
      result = null
      complete = false
      return
    }

    const diff = sum - nums[i]
    if (seen.has(diff)) {
      seen.set(nums[i], i)
      updateSeen()
      result = [seen.get(diff), i]
      complete = true
      return 
    }

    seen.set(nums[i], i)
    updateSeen()

    i += 1

    if (i >= nums.length) {
      complete = true
    }
  }

  const run = () => {
    try {
      while (!complete) {
        step()
      }
      error = null
    }
    catch (e) {
      error = e.message
    }
  }

  const setNums = () => {
    try {
      nums = JSON.parse(numsText)
      error = null
    }
    catch (e) {
      error = e.message
    }
  }
</script>








<h4>Hash Approach</h4>

<input type="text" on:keyup={setNums} bind:value={numsText} disabled={i > 0}>
<input type="number" bind:value={sum} disabled={i > 0}>

<button on:click={run}>Run</button>
<button on:click={step}>
  {#if complete}
    Reset
  {:else}
    Step
  {/if}
</button>

<p>[ 
  {#each nums as num, n}
    {#if n !== 0}
    , 
    {/if}
    {#if i === n && !complete}
      <span class="highlight">{num}</span> 
    {:else}
      {num} 
    {/if}
     
  {/each}
  ]
</p>

<table class="w3-table-all w3-large w3-centered">
  <tr>
    <th>Number</th>
    <th>Index</th>
  </tr>
  {#each [...seen] as [num, index] }
    <tr>
      <td>{num}</td>
      <td>{index}</td>
    </tr>
  {/each}
</table>

{#if result !== null}
  <p class="highlight">Result: [{nums[result[0]]}, {nums[result[1]]}]</p>
{/if}

{#if error !== null}
  <p>Inputs must be an array of numbers and a number as input</p>
  <p>Error: {error}</p>
{/if}









<style>
  table {
    max-width: 25ch;
  }
  td, th {
    max-width: 35px;
    border-right: 1px solid lightgrey;
  }
  .highlight {
    color: limegreen;
    font-weight: bold;
  }
</style>
