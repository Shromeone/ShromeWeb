<script>
  import { createEventDispatcher } from "svelte";

  const dispatch = createEventDispatcher();

  function update() {
    dispatch("update");
  }
  export function setError(isError = true) {
    error = isError;
  }

  let error = false;
  export let text = "Item";
  export let placeholder = "";
  export let value = "";
</script>

<div class="input">
  <p>{text}</p>
  <input
    class={error ? "error" : ""}
    bind:value
    {placeholder}
    on:input={update}
    on:change={update}
    type="text"
  />

  <button on:click={() => (value = "")}>Clear</button>
</div>

<style>
  .input {
    display: flex;
    width: 100%;
    gap: 2em;
  }

  .input input {
    color: lightgray;
    background-color: rgb(62, 62, 62);
    flex: 1;
    align-items: right;
  }

  .error {
    border: 1px solid red;
  }
</style>
