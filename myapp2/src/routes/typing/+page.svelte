<script>
  // @ts-nocheck

  import { tick } from "svelte";
  const GameState = Object.freeze({
    START: 0,
    PLAY: 1,
    FINISH: 2,
  });
  let content = `快速測試並提高您的打字速度！ 一個好用的在線打字工具。可選測試：時間、單詞、自定義文本和 WPM 測試。`;
  let currentWordIndex = 0;
  let input = "";
  $: wrongWords = wrongIndexes.length;
  let wrongIndexes = [];
  let startTime = 0;
  let gameState = GameState.START;
  let timeTaken = 0;
  let WPM = 0;
  let accuracy = 0;
  let textbox;

  $: currentWord = content[currentWordIndex];
  let timeElapsed = 0;
  let updateTimerInterval = null;

  function toHalfWidth(x) {
    if (x === "。") return ".";
    return x.replace(/[\uff01-\uff5e]/g, function (ch) {
      return String.fromCharCode(ch.charCodeAt(0) - 0xfee0);
    });
  }

  function startTimer() {
    if (gameState !== GameState.START) return;
    gameState = GameState.PLAY;
    startTime = Date.now();
    updateTimerInterval = setInterval(updateTimer, 100);
  }

  function updateTimer(time = -1) {
    if (time >= 0) {
      timeElapsed = time;
      return;
    }
    timeElapsed = Date.now() - startTime;
  }

  async function compoUpdate(e) {}

  function finishGame() {
    gameState = GameState.FINISH;
    timeTaken = Date.now() - startTime;
    console.log(wrongWords, content.length);
    accuracy = 1 - wrongIndexes.length / content.length;
    WPM = ((content.length * accuracy) / timeTaken) * 60000;
  }

  function wordCorrect(word) {
    let correct = toHalfWidth(word) === toHalfWidth(currentWord);
    return correct;
  }

  function validateInput(word) {
    input = "";
    console.log("word");

    for (let i = 0; i < word.length; i++) {
      currentWord = content[currentWordIndex];
      const char = word[i];
      if (!wordCorrect(char))
        wrongIndexes = [...wrongIndexes, currentWordIndex];

      currentWordIndex++;
      if (currentWordIndex >= content.length) {
        finishGame();
      }
    }
  }

  function keyDown(e) {
    if (e.key === "Backspace") {
      tryDelete();
    }
  }

  function halfInput(e) {
    // if (e.data === " ") return;
    if (e.inputType === "deleteContentBackward") return;
    if (e.inputType === "insertCompositionText") return;
    validateInput(e.data);
  }

  function tryDelete() {
    if (currentWordIndex === 0) return;
    currentWordIndex--;
    const wrongIndex = wrongIndexes.indexOf(currentWordIndex);
    if (wrongIndex === -1) return;
    wrongIndexes.splice(wrongIndex, 1);
    wrongIndexes = wrongIndexes;
  }

  async function compoEnd(e) {
    if (gameState !== GameState.PLAY) return;
    if (!e.data) return;
    validateInput(e.data);
  }

  function restart() {
    gameState = GameState.START;
    wrongIndexes = [];
    currentWordIndex = 0;
    input = "";
    textbox.focus();
    clearInterval(updateTimerInterval);
    updateTimer(0);
  }
</script>

<head>
  <link rel="preconnect" href="https://fonts.googleapis.com" />
  <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
  <link
    href="https://fonts.googleapis.com/css2?family=Noto+Serif+TC&display=swap"
    rel="stylesheet"
  />
</head>

<div class="background">
  <div class="test-content">
    {#each content as char, index (index)}
      {#if index === currentWordIndex}
        <p class="current-char">{char}</p>
      {:else if wrongIndexes.includes(index)}
        <p class="wrong">{char}</p>
      {:else}
        <p>{char}</p>
      {/if}
    {/each}
  </div>
  <input
    on:compositionupdate={compoUpdate}
    on:compositionend={compoEnd}
    on:input={startTimer}
    on:input={halfInput}
    on:keydown={keyDown}
    bind:value={input}
    bind:this={textbox}
    type="text"
  />
  <button on:click={restart}>Restart</button>
  <p>{wrongWords}</p>
  {#if gameState === GameState.PLAY}
    <p>{Math.floor(timeElapsed / 1000)}</p>
  {/if}
  {#if gameState === GameState.FINISH}
    <p>Time taken: {(timeTaken / 1000).toFixed(2) + "s"}</p>
    <p>WPM: {WPM.toFixed(1)}</p>
    <p>Accuracy: {(accuracy * 100).toFixed(1) + "%"}</p>
  {/if}
</div>

<style>
  .background {
    background-color: black;
    height: 100vh;
  }

  .test-content {
    padding: 3% 2%;
  }

  .test-content p {
    display: inline-block;
    font-family: "Noto Serif TC";
    font-size: 2rem;
    margin: 2px;
    width: 2.5rem;
    height: 2.5rem;
    border: 1px solid rgba(255, 255, 255, 0.416);
    text-align: center;
    vertical-align: middle;
  }

  .wrong {
    color: red;
  }

  .test-content .current-char {
    color: skyblue;
    border: 1px solid skyblue;
  }

  p {
    color: white;
  }
  .current-word {
    color: lightblue;
  }
</style>
