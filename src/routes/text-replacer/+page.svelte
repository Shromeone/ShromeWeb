<script>
  // @ts-nocheck
  import Input from "./Input.svelte";
  import Checkbox from "./Checkbox.svelte";
  import { onMount } from "svelte";
  import { quickSelectOptions } from "./quick-select";
  let quickRegex = "";
  let quickRegexList = [];
  let regexInput;
  let regexCmd = "";
  let result = "";
  let textToReplace = "A quick brown fox jumps over the lazy dog.";
  let replacer = "";

  onMount(() => {
    updateResult();
  });
  function updateResult() {
    try {
      result = textToReplace.replace(new RegExp(regexCmd, "g"), replacer);
      regexInput.setError(false);
    } catch {
      regexError();
    }
  }

  function regexError() {
    regexInput.setError(true);
  }

  function setRegexToQuick() {
    regexCmd = quickRegexList.join("|");
    updateResult();
  }

  function quickRegexChange(val = "", add = true) {
    if (add) {
      quickRegexList.push(val);
    } else {
      quickRegexList.splice(quickRegexList.indexOf(val), 1);
    }
    console.log(quickRegexList);
    setRegexToQuick();
  }

  function handleCheckboxClick(e) {
    quickRegexChange(e.detail.regex, e.detail.checked);
  }

  function copyResult() {
    navigator.clipboard.writeText(result);
    alert("Copy success!");
  }
</script>

<p>Target Text</p>
<textarea
  class="content"
  name="content"
  cols="30"
  rows="10"
  placeholder="Enter the text that you want to replace here"
  bind:value={textToReplace}
  on:input={updateResult}
  on:change={updateResult}
></textarea>
<p>Quick select</p>
<div class="quick-options">
  {#each quickSelectOptions as option}
    <Checkbox regex={option.regex} on:click={handleCheckboxClick}
      >{option.text}</Checkbox
    >
  {/each}
</div>
<Input
  on:update={updateResult}
  text="Regex"
  bind:this={regexInput}
  bind:value={regexCmd}
/>
<Input on:update={updateResult} text="Replace with" bind:value={replacer} />

<div class="result">
  <div class="result-text">{result}</div>
  <button on:click={copyResult} class="copy-btn">Copy Result</button>
</div>

<style>
  p {
    font-family: Verdana, Geneva, Tahoma, sans-serif;
  }
  .result-text {
    display: inline-block;
    overflow-y: scroll;
    width: 50%;
    height: 20rem;
    border: 1px solid rgb(142, 142, 142);
    color: white;
  }

  .quick-options {
    display: flex;
    gap: 1rem;
    width: 100%;
  }

  .error {
    border: red;
  }

  .result {
    display: flex;
  }

  .copy-btn {
    border-radius: 5rem;
    border: none;
    padding: 2em;
    font-size: 2rem;
    transition: all 0.3s;
  }

  .copy-btn:hover {
    background-color: rgb(125, 0, 21);
  }

  .copy-btn:active {
    opacity: 80%;
  }
</style>
