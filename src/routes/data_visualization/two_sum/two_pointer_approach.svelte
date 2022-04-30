<!----------------------------------------------------
                      LOGIC 
----------------------------------------------------->

<script lang='ts'>
  // ----------------------------------------
  // Input
  // ----------------------------------------
  let numsText: string = '[1, 2, 3]'
  let nums: number[] = JSON.parse(numsText)
  let sum: number = 3
  let error: string = null
  let complete: boolean = false
  let left: number = 0
  let right: number = nums.length - 1

  const setNums = () => {
    try {
      complete = false
      nums = JSON.parse(numsText)
      right = nums.length - 1
      error = null
    }
    catch (e) {
      error = e.message
    }
  }

  const sortNums = () => {
    nums.sort((a, b) => a-b)
    nums = nums
  }

  // ----------------------------------------
  // Run
  // ----------------------------------------
  let running: boolean = false
  let result: [number, number] = null

  const step = () => {
    if (!running) {
      complete = false
      running = true
      left = 0
      right = nums.length - 1
      sortNums()
      return
    }

    if (left >= right) {
      complete = true
      running = false
      return
    }

    const runningSum = nums[left] + nums[right]

    if (runningSum < sum) {
      left += 1
    }
    else if (runningSum > sum) {
      right -= 1
    }
    else {
      result = [nums[left], nums[right]]
      complete = true
      running = false
    }
  }

  const algorithm = () => {
    sortNums()
    left = 0
    right = nums.length - 1
    while (left < right) {
      const runningSum = nums[left] + nums[right]
      if (runningSum < sum) {
        left += 1
      }
      else if (runningSum > sum) {
        right -= 1
      }
      else {
        result = [nums[left], nums[right]]
        complete = true
        break
      }
    }
  }

</script>



<!----------------------------------------------------
                      OUTPUT 
----------------------------------------------------->



<h4>Two Pointer Approach</h4>

<!-- Need to refactor this into separate inputs component -->
<input type="text" on:keyup={setNums} bind:value={numsText} disabled={running}>
<input type="number" bind:value={sum} disabled={running}>

<button on:click={algorithm}>Run</button>
<button on:click={step}>Step</button>

{#if error !== null}
  <p>Inputs must be an array of numbers and a number as input</p>
  <p>Error: {error}</p>
{/if}

<p>[ 
  {#each nums as num, n}
    {#if n !== 0}
    , 
    {/if}
    {#if (left === n || right == n) && (complete || running)}
      <span class="highlight">{num}</span> 
    {:else}
      {num} 
    {/if}
  {/each}
  ]
</p>

{#if complete}
  <p class="highlight">Result: [{result[0]}, {result[1]}]</p>
{/if}



<!----------------------------------------------------
                      STYLING 
----------------------------------------------------->



<style>
  .highlight {
    color: limegreen;
    font-weight: bold;
  }
</style>
