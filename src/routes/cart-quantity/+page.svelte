<script>
  let quantity = 0;
</script>

<button on:click={() => quantity++}>Add to Cart</button>
<button>+2</button>
<button>+3</button>
<button>+4</button>

<p>Cart quantity: {quantity}</p>
