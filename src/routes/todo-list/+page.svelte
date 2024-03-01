<script>
  let todoInput = "";
  let todoDate = "";
  let todoList = [{ content: "blah", date: "someday" }];

  function add() {
    todoList = [
      ...todoList,
      {
        content: todoInput,
        date: todoDate,
      },
    ];
  }

  function remove(elem) {
    todoList = todoList.filter((x) => x !== elem);
  }
</script>

<h2>Todo List</h2>

<div class="todo-input-grid">
  <input
    type="text"
    bind:value={todoInput}
    on:keypress={() => {
      if (key !== "Enter") return;
      add();
    }}
  />
  <input type="date" bind:value={todoDate} />
  <button on:click={add}>Add</button>
</div>
<div class="todo-grid">
  {#each todoList as todo}
    <p>{todo.content}</p>
    <p>{todo.date}</p>
    <button on:click={() => remove(todo)}>Del</button>
  {/each}
</div>

<style>
  .todo-input-grid,
  .todo-grid {
    display: grid;
    grid-template-columns: 500px 100px 100px;
  }
</style>
