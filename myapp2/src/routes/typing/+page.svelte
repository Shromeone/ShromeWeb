<script>
  // @ts-nocheck

  import { tick } from "svelte";
  const GameState = Object.freeze({
    START: 0,
    PLAY: 1,
    FINISH: 2,
  });
  let content = `颱風彭梭`;
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
  let dth = false;

  $: contentArr = content;
  $: currentWord = content[currentWordIndex];

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
  }

  async function compoUpdate(e) {}

  function finishGame() {
    gameState = GameState.FINISH;
    timeTaken = Date.now() - startTime;
    WPM = (content.length / timeTaken) * 60000;
    accuracy = 1 - wrongWords / content.length;
  }

  function wordCorrect(word) {
    let correct = toHalfWidth(word) === toHalfWidth(currentWord);
    return correct;
  }

  function validateInput(word) {
    input = "";
    if (wordCorrect(word)) {
      currentWordIndex++;
    } else {
      wrongIndexes = [...wrongIndexes, currentWordIndex];
      currentWordIndex++;
    }
    if (currentWordIndex >= content.length) {
      finishGame();
    }
  }

  function keyDown(e) {
    if (e.key === "Backspace") {
      tryDelete();
    }
  }

  function halfInput(e) {
    if (e.data === " ") return;
    if (e.inputType === "deleteContentBackward") return;
    if (e.inputType === "insertCompositionText") return;
    validateInput(e.data);
    // input = "";
    // console.log("half: ", e);
    // if (wordCorrect(e.data)) {
    //   currentWordIndex++;
    // } else {
    //   wrongWords++;
    //   wrongIndexes = [...wrongIndexes, currentWordIndex];
    //   currentWordIndex++;
    // }
  }

  function tryDelete() {
    if (currentWordIndex === 0) return;
    currentWordIndex--;
    const wrongIndex = wrongIndexes.indexOf(currentWordIndex);
    console.log(wrongIndexes);
    console.log(wrongIndex);
    if (wrongIndex === -1) return;
    wrongIndexes.splice(wrongIndex, 1);
    wrongIndexes = wrongIndexes;
  }

  async function compoEnd(e) {
    if (gameState !== GameState.PLAY) return;
    if (!e.data) return;
    validateInput(e.data);
    // input = "";
    // if (wordCorrect(e.data)) {
    //   currentWordIndex++;
    // } else {
    //   wrongWords++;
    //   wrongIndexes = [...wrongIndexes, currentWordIndex];
    //   currentWordIndex++;
    // }
    // if (currentWordIndex >= content.length) {
    //   finishGame();
    // }
    // console.log("end", e);
  }

  function restart() {
    gameState = GameState.START;
    wrongIndexes = [];
    currentWordIndex = 0;
    input = "";
    textbox.focus();
  }
</script>

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

  .test-content p {
    display: inline-block;
  }

  .wrong {
    color: red;
  }

  .current-char {
    color: skyblue;
  }

  p {
    color: white;
  }
  .current-word {
    color: lightblue;
  }
</style>
