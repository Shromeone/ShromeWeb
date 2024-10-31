<script>
  let { increment = 1 } = $props();
  let count = $state(0);

  function add() {
    count += increment;
  }
</script>

<button onclick={add}>{count}</button>

<style>
  button {
    color: var(--light);
  }
</style>
