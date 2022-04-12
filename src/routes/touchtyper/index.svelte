<script>
  // https://www.freecodecamp.org/news/javascript-keycode-list-keypress-event-key-codes/
  // Digits are codes 48-57 inclusive
  // Letters are codes 65-90 inclusive
  // Map 0-35 and get random number in that range
  // if < 9, add to 48
  // else, add to 65 and subtract 10
  // return string.fromCharCode to get char value
  const min = 0
  const max = 35
  const digitRangeStart = 48
  const digitRangeLength = 10
  const alphaRangeStart = 65

  let randomKey = ""

  const getRandomKeyCode = () => {
    const randomBaseVal = getRandomValInRange(min, max)
    const keyValue = mapAlphaNumKeyVal(randomBaseVal)

    return keyValue
  }

  const getRandomValInRange = (min, max) => {
    const range = Math.floor(max) - Math.ceil(min)

    return Math.floor(Math.random() * range)
  }

  const mapAlphaNumKeyVal = (randomBaseVal) => {
    const digitKeyVal = digitRangeStart + randomBaseVal
    const alphaKeyVal = alphaRangeStart + randomBaseVal - digitRangeLength

    return (randomBaseVal < digitRangeLength) ? digitKeyVal : alphaKeyVal
  }

  const setRandomKey = () => {

    randomKey = getRandomKeyCode()
  }

  document.onkeydown = function (event) {
    let char = (typeof event !== 'undefined') ? event.keyCode : event.which
    if (char == randomKey) {
      setRandomKey()
    }
  }
</script>

<h1>Touch Typer</h1>
<button on:click={setRandomKey}>Random Key</button>
<p>{String.fromCharCode(randomKey)}</p>
