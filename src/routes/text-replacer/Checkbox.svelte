<script>
  import { createEventDispatcher } from "svelte";
  const dispatch = createEventDispatcher();
  let checked = false;
  export let regex = "";

  function toggleCheck() {
    checked = !checked;
    dispatch("click", { regex, checked });
  }
</script>

<div class="checkbox">
  <button class={checked ? "checked" : ""} on:click={toggleCheck}
    ><slot /></button
  >
  <!-- <input type="checkbox" bind:checked on:click={toggleCheck} /> -->
</div>

<style>
  .checkbox button {
    color: white;
    border: none;
    background-color: transparent;
    border-radius: 3rem;
    background-color: rgb(75, 75, 75);
  }

  .checkbox button.checked {
    background-color: rgb(42, 207, 0);
  }

  .checkbox {
    max-width: 40vw;
    display: flex;
    justify-content: space-between;
  }
</style>
