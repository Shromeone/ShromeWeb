<script>
  // @ts-nocheck

  import { tick } from "svelte";
  const GameState = Object.freeze({
    START: 0,
    PLAY: 1,
    FINISH: 2,
  });
  let content = `維基百科是一個多語言、內容自由、任何人都能參與的協作計劃，其目標是建立一個完整、準確且中立的百科全書。 中文維基百科的成長依靠您的參與，無論是創建新條目`;
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
    class="type-input"
    placeholder={gameState === GameState.PLAY ? "" : "在這裡開始打字"}
    on:compositionupdate={compoUpdate}
    on:compositionend={compoEnd}
    on:input={startTimer}
    on:input={halfInput}
    on:keydown={keyDown}
    bind:value={input}
    bind:this={textbox}
    type="text"
  />

  {#if gameState !== GameState.PLAY}
    <input
      class="type-prep"
      type="text"
      placeholder="這裡可以調整輸入法，準備開始打字"
    />
  {/if}
  <button class="restart-btn" on:click={restart}>重新開始</button>
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

  .type-input {
    font-size: 3rem;
    display: block;
    margin-bottom: 3rem;
  }

  .type-prep {
    font-size: 2rem;
    width: 80%;
    display: block;
  }

  .restart-btn {
    padding: 1em;
    font-size: 1rem;
    border-radius: 1rem;
    border: none;
  }

  .restart-btn:hover {
    opacity: 80%;
  }
</style>
