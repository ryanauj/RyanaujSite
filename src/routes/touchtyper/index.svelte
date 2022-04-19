<script>
  import Navbar from '../../components/navbar.svelte'
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
      radioText: '[a-z0-9]',
      getRandomKeyCode: () => {
        const randomBaseVal = getRandomValInRange(0, 35)
        const digitKeyVal = digitRangeStart + randomBaseVal
        const alphaKeyVal = alphaRangeStart + randomBaseVal - digitRangeLength
        return (randomBaseVal < digitRangeLength) ? digitKeyVal : alphaKeyVal
      }
    },
    [ALPHA]: {
      text: 'All Alpha Keys [a-z]',
      radioText: '[a-z]',
      getRandomKeyCode: () => alphaRangeStart + getRandomValInRange(0, 25)
    },
    [NUM]: {
      text: 'All Numeric Keys [0-9]',
      radioText: '[0-9]',
      getRandomKeyCode: () => digitRangeStart + getRandomValInRange(0, 9)
    }
  }

  let randomKey = ''
  let startTime = null
  let currentCharRange = ALPHANUM

  let elapsedTime = 0
  let numMisses = 0
  let makesInRow = 0
  let maxMakesInRow = 0
  let total = 0
  let averageMakeTime = Infinity

  let charCounts = {}

  const resetTimer = () => {
    startTime = Date.now()
  }

  const reset = () => {
    elapsedTime = 0
    numMisses = 0
    makesInRow = 0
    maxMakesInRow = 0
    total = 0
    averageMakeTime = Infinity
    charCounts = {}
    setRandomKey()
  }

  const setRandomKey = () => {
    let newKey = randomKey
    while (newKey === randomKey) {
      newKey = CharRange[currentCharRange].getRandomKeyCode()
    }
    randomKey = newKey 

    if (!(randomKey in charCounts)) {
      charCounts[randomKey] = {
        makes: 0,
        misses: 0
      }
    }

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
        charCounts[randomKey].makes += 1
        setRandomKey()
        numMisses = 0
        makesInRow += 1
        total += 1

        maxMakesInRow = Math.max(makesInRow, maxMakesInRow)

        if (total === 1) {
          averageMakeTime = elapsedTime
        }
        else {
          const previousAverageTotal = averageMakeTime * (total-1)
          const averageTotal = previousAverageTotal + elapsedTime
          averageMakeTime = averageTotal / total
        }
      }
      else {
        makesInRow = 0
        numMisses += 1
        charCounts[randomKey].misses += 1
      }
    }
  })
  // Current makes in row
  // Max makes in row
  // Average make time overall

  // Num makes and num misses per character
  // Average make time per character
  // Average attempts per character (makes / total attempts per character)
</script>

<Navbar/>
<div class="w3-container">
  <h1>Touch Typer</h1>

  <p>{CharRange[currentCharRange].text}</p>

  <button class="w3-button w3-black" on:click={reset}>
    {#if randomKey === ''}
      Random Key
    {:else}
      Reset
    {/if}
  </button>

  <div>
    {#each Object.keys(CharRange) as charRangeKey}
      <label>
        <input class="w3-radio" type=radio bind:group={currentCharRange} name="charRange" value={charRangeKey}/>
        {CharRange[charRangeKey].radioText}
      </label>
    {/each}
  </div>

  <p id="target">{String.fromCharCode(randomKey)}</p>

  {#if randomKey != ''}
    <table class="w3-table-all w3-large">
      <tr>
        <th>Current Num Misses</th>
        <th>Makes In Row</th>
        <th>Last Char Elapsed Time</th>
      </tr>
      <tr>
        <td>{numMisses}</td>
        <td>{makesInRow}</td>
        <td>{roundToDecimalPlaces(elapsedTime, 3)}</td>
      </tr>
    </table>
    <table class="w3-table-all w3-large">
      <tr>
        <th>Total</th>
        <th>Max Makes In Row</th>
        <th>Average Make Time</th>
      </tr>
      <tr>
        <td>{total}</td>
        <td>{maxMakesInRow}</td>
        <td>{roundToDecimalPlaces(averageMakeTime, 3)}</td>
      </tr>
    </table>
    <table id="char_counts" class="w3-table-all w3-large">
      <tr>
        <th>Character</th>
        <th>Makes</th>
        <th>Misses</th>
      </tr>
      {#each Object.keys(charCounts) as char }
        <tr>
          <td>{String.fromCharCode(char)}</td>
          <td>{charCounts[char].makes}</td>
          <td>{charCounts[char].misses}</td>
        </tr>
      {/each}
    </table>
  {/if}
</div>

<style>
  label {
    padding-right: 16px;
  }
  td, th {
    max-width: 35px;
  }
  table {
    margin-bottom: 10px;
  }
  #char_counts {
    max-width: 40ch;
  }
  #target {
    font-size: 20px;
    font-weight: bold;
    color: orange;
  }
</style>
