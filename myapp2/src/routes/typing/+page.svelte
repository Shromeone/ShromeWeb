<script>
  // @ts-nocheck

  import { onMount, tick } from "svelte";
  const GameState = Object.freeze({
    START: 0,
    PLAY: 1,
    FINISH: 2,
  });
  const removeContentSpace = true;
  const mountainPoem = `若夫霪雨霏霏，連月不開；陰風怒號，濁浪排空；日星隱耀，山
  岳潛形；商旅不行，檣傾楫摧；薄暮冥冥，虎嘯猿啼。登斯樓也，則
  有去國懷鄉，憂讒畏譏，滿目蕭然，感極而悲者矣。
  至若春和景明 ，波瀾不驚 ，上下天光，一碧萬頃；沙鷗翔集，錦
  鱗游泳，岸芷汀蘭，郁郁青青。而或長煙一空，皓月千里，浮光躍金，
  靜影沉璧；漁歌互答，此樂何極！登斯樓也，則有心曠神怡，寵辱皆
  忘，把酒臨風，其喜洋洋者矣。
  嗟夫！予嘗求古仁人之心，或異二者之為 。何哉？不以物喜，不
  以己悲。居廟堂之高，則憂其民；處江湖之遠，則憂其君。是進亦憂，
  退亦憂，然則何時 而樂耶？其必曰：「先天下之憂而憂，後天下之樂而
  樂」歟！噫！微斯人，吾誰與歸！`;
  let content = `  若夫霪雨霏霏，連月不開；陰風怒號，濁浪排空；日星隱耀，山
  岳潛形；商旅不行，檣傾楫摧；薄暮冥冥，虎嘯猿啼。登斯樓也，則
  有去國懷鄉，憂讒畏譏，滿目蕭然，感極而悲者矣。
  至若春和景明 ，波瀾不驚 ，上下天光，一碧萬頃；沙鷗翔集，錦
  鱗游泳，岸芷汀蘭，郁郁青青。而或長煙一空，皓月千里，浮光躍金，
  靜影沉璧；漁歌互答，此樂何極！登斯樓也，則有心曠神怡，寵辱皆
  忘，把酒臨風，其喜洋洋者矣。
  嗟夫！予嘗求古仁人之心，或異二者之為 。何哉？不以物喜，不
  以己悲。居廟堂之高，則憂其民；處江湖之遠，則憂其君。是進亦憂，
  退亦憂，然則何時 而樂耶？其必曰：「先天下之憂而憂，後天下之樂而
  樂」歟！噫！微斯人，吾誰與歸！
  `;

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
  let inputBox;
  let inputDisplay;
  let typePrep;
  let focused = false;

  let isCompo = false;
  $: showInputDisplay = isCompo && input !== "";
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

  async function compoUpdate(e) {
    isCompo = true;
  }

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

  function updateInputBoxPos() {
    const currentChar = document.querySelector(`#char-${currentWordIndex}`);
    const rect = currentChar.getBoundingClientRect();
    inputBox.style.top = rect.top + inputBoxOffset.y + "px";
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
    console.log("half input");
    setTimeout(clearInput, 0);
  }

  function validateInput(word) {
    for (let i = 0; i < word.length; i++) {
      currentWord = content[currentWordIndex];
      const char = word[i];
      if (!wordCorrect(char))
        wrongIndexes = [...wrongIndexes, currentWordIndex];
      else correctIndexes = [...correctIndexes, currentWordIndex];
      currentWordIndex++;
      if (currentWordIndex >= content.length) {
        finishGame();
      }
    }

    clearInput();
    updateInputBoxPos();
  }

  function tryDelete() {
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
    wrongIndexes = [];
    currentWordIndex = 0;
    // input = "";
    clearInput();
    inputBox.focus();
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
  {#if gameState !== GameState.PLAY}
    <input
      class="type-prep"
      type="text"
      placeholder="這裡可以調整輸入法，準備開始打字(按Enter 進入測試)"
      bind:this={typePrep}
    />
  {/if}
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

  <p>{wrongWords}</p>
  {#if gameState !== GameState.START}
    <button class="restart-btn" on:click={restart}>重新開始</button>
  {/if}
  {#if gameState === GameState.PLAY}
    <p>{Math.floor(timeElapsed / 1000)}</p>
  {/if}
  {#if gameState === GameState.FINISH}
    <p>Time taken: {(timeTaken / 1000).toFixed(2) + "s"}</p>
    <p>WPM: {WPM.toFixed(1)}</p>
    <p>Accuracy: {(accuracy * 100).toFixed(1) + "%"}</p>
  {/if}

  <p contenteditable="true">Hi</p>
</div>

<style>
  .background {
    background-color: black;
    min-height: 100vh;
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
    display: none;
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
