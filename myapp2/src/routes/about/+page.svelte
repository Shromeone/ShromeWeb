<script>
  let count = 0;
  let inputVal = "";
  let selected = [""];
  $: txt = 3;
  function addCount() {
    count++;
  }
</script>

<h1>About</h1>
<p>{inputVal}</p>
<p>Count: {count}</p>
<input type="text" bind:value={inputVal} />
<button on:click={addCount}>Click me</button>

{#each [1, 2, 3, 4] as num}
  <label>
    <input type="checkbox" name="num" value={num} bind:group={selected} />

    Click {num}
  </label>
{/each}

<p>Clicked {selected}</p>
