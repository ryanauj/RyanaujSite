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

  const setNums = () => {
    try {
      nums = JSON.parse(numsText)
      error = null
    }
    catch (e) {
      error = e.message
    }
  }

  const sortNums = () => {
    nums.sort((a, b) => a-b)
    nums = nums
    numsText = JSON.stringify(nums)
  }

  // ----------------------------------------
  // Run
  // ----------------------------------------
  let i: number = 0
  let complete: boolean = false
  let result: [number, number] = null

  const algorithm = () => {
    sortNums()
    let left = 0
    let right = nums.length - 1
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
<input type="text" on:keyup={setNums} bind:value={numsText} disabled={i > 0}>
<input type="number" bind:value={sum} disabled={i > 0}>

<button on:click={algorithm}>Run</button>

{#if error !== null}
  <p>Inputs must be an array of numbers and a number as input</p>
  <p>Error: {error}</p>
{/if}

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
