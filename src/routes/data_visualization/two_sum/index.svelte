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

  const updateSeenText = () => {
    seenText = JSON.stringify(Object.fromEntries(seen))
  }

  const step = () => {
    if (complete) {
      seen = new Map<number, number>()
      updateSeenText()
      i = 0
      result = null
      complete = false
      return
    }

    const diff = sum - nums[i]
    if (seen.has(diff)) {
      seen.set(nums[i], i)
      updateSeenText()
      result = [seen.get(diff), i]
      complete = true
      return 
    }

    seen.set(nums[i], i)
    updateSeenText()

    i += 1

    if (i >= nums.length) {
      complete = true
    }
  }

  const hashApproach = (nums: number[], sum: number) => {
    seen = new Map<number, number>()

    for (let j = 0; j < nums.length; j++) {
      const diff = sum - nums[j]
      if (seen.has(diff)) {
        updateSeenText()
        result = [seen.get(diff), j]
        i = j
        complete = true
        return
      }
      seen.set(nums[j], j)
    }
    complete = true
  }

  const run = () => {
    try {
      hashApproach(nums, sum)
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

<h1>Two Sum</h1>

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

<p>Seen: {seenText}</p>
<p>Iteration: {i}</p>

{#if result !== null}
  <p>Result: [{nums[result[0]]}, {nums[result[1]]}]</p>
{/if}

{#if error !== null}
  <p>Inputs must be an array of numbers and a number as input</p>
  <p>Error: {error}</p>
{/if}
