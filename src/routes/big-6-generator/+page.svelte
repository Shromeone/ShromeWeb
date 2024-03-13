<script>
  // @ts-nocheck

  import { tick } from "svelte";

  // @ts-nocheck
  const redNums = [
    1, 2, 7, 8, 12, 13, 18, 19, 23, 24, 29, 30, 34, 35, 40, 45, 46,
  ];
  const blueNums = [3, 4, 9, 10, 14, 15, 20, 31, 41, 42, 47, 48];
  function func(num) {
    if (redNums.includes(num)) return "coral";
    if (blueNums.includes(num)) return "skyblue";
    return "limegreen";
  }
  const hotNumbers = [
    49, 22, 10, 24, 30, 13, 9, 33, 6, 4, 14, 35, 12, 20, 28, 7, 34,
  ];
  let currentNumbers = [];

  async function resetBallsAnimation() {
    for (let i = 0; i < 6; i++) {
      const el = document.querySelector(`#ball-${i}`);
      if (!el) continue;
      el.classList.add("hidden");
      el.style.animation = "none";
      setTimeout(() => {
        el.classList.remove("hidden");
        el.style.animation = "none";
        el.offsetHeight; /* trigger reflow */
        el.style.animation = null;
      }, 200 * i);
      //   await tick().then(() => {});
    }
  }

  function generateNumbers() {
    const genNumbers = [];
    while (genNumbers.length < 6) {
      const num = Math.ceil(Math.random() * 49);
      if (genNumbers.includes(num) || hotNumbers.includes(num)) continue;
      genNumbers.push(num);
    }
    currentNumbers = genNumbers;
    setTimeout(() => {
      resetBallsAnimation();
    }, 0);
  }
</script>

import {tick} from "svelte"

<h2>六合彩生成器</h2>
<div class="gen-btn-div">
  <button class="gen-btn" on:click={generateNumbers}>發發發！</button>
</div>
<p>中獎號碼：</p>
<div class="nums">
  {#each currentNumbers as num, idx}
    <p
      id="ball-{idx}"
      style="border: 1rem solid {func(num)};"
      class="result-num ball-animation hidden"
    >
      {num}
    </p>
  {/each}
</div>

<style>
  .hidden {
    opacity: 0%;
  }
  .gen-btn-div {
    margin: 0%;
    width: 100%;
    display: flex;
    justify-content: center;
  }
  .gen-btn {
    width: 50vw;
    height: 20vw;
    font-size: 3rem;
  }

  h2 {
    font-size: 3rem;
    text-align: center;
  }

  .nums {
    display: grid;
    grid-template-columns: 1fr 1fr 1fr 1fr 1fr 1fr;
  }

  @keyframes ball-animation {
    0% {
      transform: scale(0, 0) translateY(400px);
      opacity: 0%;
    }

    /* 50% {
      translate: 0px -200px;
    } */

    to {
      transform: scale(100%, 100%) translateY(0px);
      opacity: 100%;
    }
  }

  .ball-animation {
    animation: ball-animation 0.4s ease-out;
  }

  .result-num {
    flex-shrink: 0;
    display: inline-block;
    width: 3rem;
    height: 3rem;
    line-height: 3rem;
    color: black;
    background-color: white;
    font-weight: bolder;
    font-size: 2rem;
    text-align: center;
    border-radius: 100rem;
    margin-right: 0.5rem;
  }

  @media (max-width: 600px) {
    .nums {
      grid-template-columns: 1fr 1fr 1fr;
    }
  }
</style>
