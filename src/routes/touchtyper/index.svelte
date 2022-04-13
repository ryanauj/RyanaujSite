<script>
  import { onMount, onDestroy } from 'svelte'

  const getRandomValInRange = (min, max) => {
    const range = Math.floor(max) - Math.ceil(min)

    return Math.floor(Math.random() * range)
  }

  const roundToDecimalPlaces = (num, decimalPlaces) => {
    if (decimalPlaces < 1) {
      return num
    }
    const factor = Math.pow(10, decimalPlaces)
    return Math.round((num + Number.EPSILON) * factor) / factor
  }

  // https://www.freecodecamp.org/news/javascript-keycode-list-keypress-event-key-codes/
  // Digits are codes 48-57 inclusive
  // Letters are codes 65-90 inclusive
  // Map 0-35 and get random number in that range
  // if < 9, add to 48
  // else, add to 65 and subtract 10
  // return string.fromCharCode to get char value
  const digitRangeStart = 48
  const digitRangeLength = 10
  const alphaRangeStart = 65

  const ALPHANUM = 'ALPHANUM'
  const ALPHA = 'ALPHA'
  const NUM = 'NUM'

  const CharRange = {
    [ALPHANUM]: {
      text: 'All Alphanumeric Keys [a-z] and [0-9]',
      getRandomKeyCode: () => {
        const randomBaseVal = getRandomValInRange(0, 35)
        const digitKeyVal = digitRangeStart + randomBaseVal
        const alphaKeyVal = alphaRangeStart + randomBaseVal - digitRangeLength
        return (randomBaseVal < digitRangeLength) ? digitKeyVal : alphaKeyVal
      }
    },
    [ALPHA]: {
      text: 'All Alpha Keys [a-z]',
      getRandomKeyCode: () => alphaRangeStart + getRandomValInRange(0, 25)
    },
    [NUM]: {
      text: 'All Numeric Keys [0-9]',
      getRandomKeyCode: () => digitRangeStart + getRandomValInRange(0, 9)
    }
  }

  let randomKey = ''
  let startTime = null
  let elapsedTime = null
  let numMisses = 0
  let makesInRow = 0
  let averageMakeTime = Infinity
  let currentCharRange = ALPHANUM

  const setCurrentCharRange = (charRange) => {
    currentCharRange = charRange
  }

  const resetTimer = () => {
    startTime = Date.now()
  }

  const setRandomKey = () => {
    randomKey = CharRange[currentCharRange].getRandomKeyCode()
    resetTimer()
  }

  const stopTimer = () => {
    const stopTime = Date.now()
    elapsedTime = (stopTime - startTime) / 1000
  }

  onMount(() => {
    document.onkeydown = function (event) {
      let char = (typeof event !== 'undefined') ? event.keyCode : event.which
      if (char == randomKey) {
        stopTimer()
        setRandomKey()
        numMisses = 0
        makesInRow += 1

        if (makesInRow === 1) {
          averageMakeTime = elapsedTime
        }
        else {
          const previousAverageTotal = averageMakeTime * (makesInRow-1)
          const averageTotal = previousAverageTotal + elapsedTime
          averageMakeTime = averageTotal / makesInRow
        }
      }
      else {
        makesInRow = 0
        numMisses += 1
        averageMakeTime = Infinity
      }
    }
  })
</script>

<h1>Touch Typer</h1>
<p>{CharRange[currentCharRange].text}</p>
<button on:click={setRandomKey}>Random Key</button>
<button on:click={() => setCurrentCharRange(ALPHANUM)}>
  [a-z0-9]
</button>
<button on:click={() => setCurrentCharRange(ALPHA)}>
  [a-z]
</button>
<button on:click={() => setCurrentCharRange(NUM)}>
  [0-9]
</button>
<p>{String.fromCharCode(randomKey)}</p>
{#if randomKey != ''}
  <p>{numMisses}</p>
  <p>{makesInRow}</p>
  <p>{roundToDecimalPlaces(averageMakeTime, 3)}</p>
{/if}
{#if elapsedTime !== null}
  <p>{roundToDecimalPlaces(elapsedTime, 3)}</p>
{/if}
