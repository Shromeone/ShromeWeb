<script>
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
  const hotNumbers = [49, 22, 10, 24, 30, 13, 9, 33, 6, 4, 14];
  let currentNumbers = [];
  function generateNumbers() {
    const genNumbers = [];
    while (genNumbers.length < 6) {
      const num = Math.ceil(Math.random() * 49);
      if (genNumbers.includes(num) || hotNumbers.includes(num)) continue;
      genNumbers.push(num);
    }
    currentNumbers = genNumbers;
  }
</script>

<h2>六合彩生成器</h2>
<button class="gen-btn" on:click={generateNumbers}> 發發發！</button>
<p>中獎號碼：</p>
{#each currentNumbers as num}
  <p style="border: 1rem solid {func(num)};" class="result-num">{num}</p>
{/each}

<style>
  .gen-btn {
    padding: 1em 3em;
    font-size: 3rem;
  }

  .result-num {
    display: inline-block;
    width: 3rem;
    height: 3rem;
    line-height: 3rem;
    color: black;
    background-color: white;
    font-weight: bolder;
    font-size: 2rem;
    text-align: center;
    vertical-align: middle;
    border-radius: 100rem;
    margin-right: 0.5rem;
  }
</style>
