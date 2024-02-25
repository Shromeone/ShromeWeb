<script>
  // @ts-nocheck

  import { mountainPoem } from "./passages.json";
  import { onMount, tick } from "svelte";
  import { charPoints, bonus } from "./bonus-points.json";
  const GameState = Object.freeze({
    START: 0,
    PLAY: 1,
    FINISH: 2,
  });
  const removeContentSpace = true;

  let content = mountainPoem;

  let currentWordIndex = 0;
  let input;

  $: correctWords = correctIndexes.length;
  $: wrongWords = wrongIndexes.length;
  let correctIndexes = [];
  let wrongIndexes = [];
  let startTime = 0;
  let gameState = GameState.START;
  let timeTaken = 0;
  let WPM = 0;
  let accuracy = 0;

  $: currentWord = content[currentWordIndex];
  let timeElapsed = 0;
  let updateTimerInterval = null;
  let updateInfoInterval = null;

  let inputBox;
  let inputDisplay;
  let typePrep;
  let resultsScreen;

  let focused = false;

  let isCompo = false;
  const timeLimit = 15;
  let points;
  let accuracyPoint;
  let speedPoint;
  let accuracyCutoff;
  let speedCutoff;

  let isTimeUp = false;
  $: showInputDisplay = isCompo && input !== "";
  const scrollDeadzone = 500;
  const scrollOffset = 100;
  const inputBoxOffset = {
    x: 0,
    y: 20,
  };
  onMount(() => {
    content = content.replace(/(?:\r\n|\r|\n)/g, "");
    if (removeContentSpace) content = content.replace(/\s/g, "");
    updateInputBoxPos();
    inputBox.focus();
    document.onkeydown = (e) => {
      tryPressEnterFocus(e);
      if (document.activeElement === inputBox) return;
      if (document.activeElement === typePrep) return;
      e.preventDefault();
      inputBox.focus();
    };
  });

  function tryPressEnterFocus(e) {
    if (document.activeElement !== typePrep) return;
    if (e.key !== "Enter") return;
    inputBox.focus();
  }

  function clearInput() {
    inputBox.innerHTML = "";
    input = "";
  }

  function toHalfWidth(x) {
    if (x === "。") return ".";
    return x.replace(/[\uff01-\uff5e]/g, function (ch) {
      return String.fromCharCode(ch.charCodeAt(0) - 0xfee0);
    });
  }

  function startTimer(e) {
    if (gameState !== GameState.START) return;
    isTimeUp = false;
    timeTaken = 0;
    gameState = GameState.PLAY;
    startTime = Date.now();
    updateInfoInterval = setInterval(updateInfo, 2000);
    updateTimerInterval = setInterval(updateTimer, 100);
  }

  function updateTimer(time = -1) {
    if (time >= 0) {
      timeElapsed = time;
      return;
    }
    timeElapsed = Date.now() - startTime;

    const timeElapsedInSec = timeElapsed / 1000;
    if (timeElapsedInSec > timeLimit) {
      timeUp();
    }
  }

  function timeUp() {
    isTimeUp = true;
    finishGame();
  }

  async function compoUpdate(e) {
    isCompo = true;
  }

  function finishGame() {
    clearInterval(updateTimerInterval);
    clearInterval(updateInfoInterval);
    gameState = GameState.FINISH;
    console.log(wrongWords, content.length);
    updateInfo();
    calcFinalPoints();
    setResultsPanelVisibility(true);
  }

  function updateInfo() {
    timeTaken = isTimeUp ? timeLimit * 1000 : Date.now() - startTime;
    const wrongs = wrongIndexes.length;
    const corrects = correctIndexes.length;
    const wordsTyped = corrects + wrongs;
    accuracy = 1 - wrongs / wordsTyped;
    WPM = ((wordsTyped * accuracy) / timeTaken) * 60000;
  }

  function wordCorrect(word) {
    let correct = toHalfWidth(word) === toHalfWidth(currentWord);
    return correct;
  }

  function updateInputBoxPos() {
    const currentChar = document.querySelector(`#char-${currentWordIndex}`);
    const rect = currentChar.getBoundingClientRect();
    inputBox.style.top = rect.top + scrollY + inputBoxOffset.y + "px";
    inputBox.style.left = rect.left + inputBoxOffset.x + "px";
    currentChar.appendChild(inputDisplay);
  }

  function keyDown(e) {
    if (e.key === "Backspace") {
      tryDelete();
    }
  }

  function halfInput(e) {
    if (e.inputType === "deleteContentBackward") return;
    if (e.inputType === "insertCompositionText") return;
    isCompo = false;
    validateInput(e.data);
    setTimeout(clearInput, 0);
  }

  function updateScroll() {
    const currentChar = document.querySelector(`#char-${currentWordIndex}`);
    const y = currentChar.getBoundingClientRect().top + scrollY;
    if (Math.abs(scrollY - y) < scrollDeadzone) return;
    console.log("scroll");
    scroll({ top: y - scrollOffset, behavior: "smooth" });
  }

  function validateInput(word) {
    if (gameState === GameState.FINISH) return;
    for (let i = 0; i < word.length; i++) {
      currentWord = content[currentWordIndex];
      const char = word[i];
      if (!wordCorrect(char)) {
        wrongIndexes = [...wrongIndexes, currentWordIndex];
      } else {
        correctIndexes = [...correctIndexes, currentWordIndex];
      }
      currentWordIndex++;
      if (currentWordIndex >= content.length) {
        finishGame();
      }
    }
    updateScroll();
    clearInput();
    updateInputBoxPos();

    if (gameState !== GameState.FINISH) calcTempPoints();
  }

  function calcTempPoints() {
    points =
      correctIndexes.length * charPoints.correct +
      wrongIndexes.length * charPoints.wrong;
  }

  function calcFinalPoints() {
    accuracyPoint = getAccuracyPoint();
    speedPoint = getSpeedPoint();

    points =
      correctIndexes.length * charPoints.correct +
      wrongIndexes.length * charPoints.wrong +
      accuracyPoint +
      speedPoint;

    console.log(`points: ${points}`);
  }

  function getSpeedPoint() {
    const speedPoints = Object.keys(bonus.speed);
    speedPoints.sort((a, b) => Number(b) - Number(a));
    for (let speed of speedPoints) {
      console.log(WPM, Number(speed));
      if (WPM >= Number(speed)) {
        console.log(`speed: ${bonus.speed[speed]}`);
        speedCutoff = speed;
        return bonus.speed[speed];
      }
    }
    return 0;
  }

  function getAccuracyPoint() {
    const accuracyPoints = Object.keys(bonus.accuracy);
    accuracyPoints.sort((a, b) => Number(b) - Number(a));
    for (let accu of accuracyPoints) {
      console.log(accuracy * 100, Number(accu));
      if (accuracy * 100 >= Number(accu)) {
        console.log(`accu: ${bonus.accuracy[accu]}`);
        accuracyCutoff = accu;
        return bonus.accuracy[accu];
      }
    }

    return 0;
  }

  function tryDelete() {
    if (gameState === GameState.FINISH) return;
    if (currentWordIndex === 0) return;
    currentWordIndex--;
    updateInputBoxPos();
    wrongIndexes = wrongIndexes.filter((x) => x !== currentWordIndex);
    correctIndexes = correctIndexes.filter((x) => x !== currentWordIndex);
  }

  async function compoEnd(e) {
    if (gameState !== GameState.PLAY) return;
    if (!e.data) return;
    validateInput(e.data);
  }

  function restart() {
    gameState = GameState.START;
    timeTaken = 0;
    wrongIndexes = [];
    correctIndexes = [];
    currentWordIndex = 0;
    // input = "";
    clearInput();
    inputBox.focus();
    clearInterval(updateTimerInterval);
    clearInterval(updateInfoInterval);
    updateTimer(0);
    updateInputBoxPos();
    updateScroll();
  }

  function setResultsPanelVisibility(show = false) {
    if (show) {
      resultsScreen.classList.remove("hidden");
    } else {
      resultsScreen.classList.add("hidden");
    }
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
  {#if gameState !== GameState.PLAY}
    <input
      class="type-prep"
      type="text"
      placeholder="這裡可以調整輸入法，準備開始打字(按Enter 進入測試)"
      bind:this={typePrep}
    />
  {/if}
  <div class="info-bar">
    {#if gameState !== 3}
      <p>Time Left: {Math.ceil(timeLimit - timeElapsed / 1000)}</p>
      <p>Time taken: {(timeTaken / 1000).toFixed(2) + "s"}</p>
      <p>WPM: {WPM.toFixed(1)}</p>
      <p>Accuracy: {(accuracy * 100).toFixed(1) + "%"}</p>
      <p>Correct: {correctWords}</p>
      <p>Wrong: {wrongWords}</p>
      <p>Points : {points}</p>
    {/if}
  </div>
  <input
    type="text"
    id="type-input"
    placeholder={gameState === GameState.PLAY ? "" : "在這裡開始打字"}
    on:compositionupdate={compoUpdate}
    on:compositionend={compoEnd}
    on:input={startTimer}
    on:input={halfInput}
    on:keydown={keyDown}
    on:focus={() => (focused = true)}
    on:focusout={() => (focused = false)}
    bind:value={input}
    bind:this={inputBox}
  />
  <div
    id="input-display"
    class={showInputDisplay ? "" : "hidden"}
    bind:this={inputDisplay}
  >
    {showInputDisplay ? input : ""}
  </div>
  <div class="test-content">
    {#each content as char, index (index)}
      <div class="char" id="char-{index}">
        {#if index === currentWordIndex}
          <div class={focused ? "" : "inactive"}>
            <div class="current-char">
              <p>{char}</p>
            </div>
          </div>
        {:else if wrongIndexes.includes(index)}
          <p class="wrong">{char}</p>
        {:else if correctIndexes.includes(index)}
          <p class="correct">{char}</p>
        {:else}
          <p>{char}</p>
        {/if}
      </div>
    {/each}
  </div>

  {#if gameState !== GameState.START}
    <button class="restart-btn" on:click={restart}>重新開始</button>
  {/if}
  {#if gameState === GameState.PLAY}
    <p>{Math.floor(timeElapsed / 1000)}</p>
  {/if}

  <div class="results-screen" bind:this={resultsScreen}>
    <div class="results-panel">
      <p>準確度：{(accuracy * 100).toFixed(1) + "%"}</p>
      <p>速度：{WPM.toFixed(1)}WPM</p>
      <p>正確：{correctIndexes.length}字</p>
      <p>錯誤：{wrongIndexes.length}字</p>
      <p>分數：{points}</p>
      <p>
        正確加分：{correctIndexes.length}*{charPoints.correct}={correctIndexes.length *
          charPoints.correct}
      </p>
      <p>
        錯誤扣分：{wrongIndexes.length}*{charPoints.wrong}={wrongIndexes.length *
          charPoints.wrong}
      </p>
      <p>
        準確度加分：{accuracyPoint}
        {#if accuracyPoint > 0}
          ({accuracyCutoff}%或以上)
        {/if}
      </p>
      <p>
        速度加分：{speedPoint}
        {#if speedPoint > 0}
          ({speedCutoff}WPM或以上)
        {/if}
      </p>
      <button
        class="close-btn"
        on:click={() => setResultsPanelVisibility(false)}>關閉</button
      >
    </div>
  </div>
</div>

<style>
  .results-screen {
    opacity: 100%;
    position: fixed;
    display: flex;
    top: 0;
    bottom: 0;
    left: 0;
    right: 0;
    background-color: rgba(0, 0, 0, 0.258);
    align-items: center;
    justify-content: center;
    transition: all 0.3s;
  }

  .results-panel {
    background-color: rgb(91, 97, 148);
    width: 40vw;
    padding: 4em;
    border-radius: 2rem;
    border: 3px dashed rgb(38, 38, 84);
    box-shadow: 0px 0px 30px black;
    justify-content: center;
  }

  .close-btn {
  }
  .background {
    background-color: black;
    min-height: 100vh;
  }

  .info-bar {
    display: flex;
    gap: 20px;
  }

  .test-content {
    padding: 3% 2%;
  }

  .test-content p {
    display: inline-block;
    font-family: "Noto Serif TC";
    font-size: 2rem;
    margin: 5px;
    width: 2.5rem;
    height: 2.5rem;
    border: 1px solid rgba(255, 255, 255, 0.416);
    text-align: center;
    vertical-align: middle;
  }

  .char {
    display: inline-block;
    position: relative;
  }

  .wrong {
    color: red;
  }

  .correct {
    color: rgb(101, 196, 101);
  }

  #type-input {
    position: absolute;
    color: white;
    left: 0;
    font-size: 1rem;
    height: 2rem;
    background-color: transparent;
    border: none;
    overflow: hidden;
    width: 0.1px;
  }

  #type-input:focus {
    border: none;
    outline: 0;
    caret-color: rgba(0, 0, 0, 0);
  }

  .hidden {
    visibility: hidden;
    opacity: 0%;
  }

  #input-display {
    z-index: 3;
    white-space: nowrap;
    color: white;
    position: absolute;
    background-color: rgb(85, 85, 85);
    text-align: left;
    padding: 0.2rem 0.2rem;
    opacity: 100%;
  }

  #input-display:blank {
    display: none;
  }
  .current-char {
    display: inline-block;
    position: relative;
  }

  .current-char p {
    color: skyblue;
    border: 1px solid skyblue;
  }

  .inactive p {
    border-color: grey;
    color: grey;
  }

  p {
    color: white;
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
