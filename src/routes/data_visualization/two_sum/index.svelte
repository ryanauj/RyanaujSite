<script lang='ts'>
  const hashApproach = (nums: number[], sum: number): [number, number] => {
    const seen = new Map<number, number>()

    for (let i = 0; i < nums.length; i++) {
      const diff = sum - nums[i]
      if (seen.has(diff)) {
        return [seen.get(diff), i]
      }
      seen.set(nums[i], i)
    }
    return [-1, -1]
  }

  const next = () => {
    
  }

  let numsText: string = '[]'
  let sum: number = -1
  let result: [number, number] = [-1, -1]
  let error: string = null

  const run = () => {
    try {
      const nums: number[] = JSON.parse(numsText)
      result = hashApproach(nums, sum)
      error = null
    }
    catch (e) {
      error = e.message
    }
  }
</script>

<h1>Two Sum</h1>

<input type="text" bind:value={numsText}>
<input type="number" bind:value={sum}>

<button on:click={run}>Run</button>

{#if result !== null}
  <p>[{result[0]}, {result[1]}]</p>
{/if}

{#if error !== null}
  <p>Error: {error}</p>
{/if}
