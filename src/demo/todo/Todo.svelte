<script lang="ts">
  import { remult } from "remult";
  import { Task } from "./Task";
  import Tile from "../Tile.svelte";

  let tasks = $state<Task[]>([]);
  let hideCompleted = $state(false);
  function toggleHideCompleted() {
    hideCompleted = !hideCompleted;
  }

  const refresh = async (_hideCompleted: boolean) => {
    const ts = await remult.repo(Task).find({
      where: {
        completed: hideCompleted ? false : undefined,
      },
      orderBy: {
        createdAt: "desc",
      },
    });
    // svelte 5 needs this to have tasks reactive
    tasks = [...ts];
  };

  $effect(() => {
    refresh(hideCompleted);
  });

  let newTaskTitle = $state("");
  const addTask = async () => {
    const newTask = await remult.repo(Task).insert({ title: newTaskTitle });
    tasks = [...tasks, newTask];
    newTaskTitle = "";
  };
  const setCompleted = async (task: Task, event: Event) => {
    const input = event.target as HTMLInputElement;
    return {
      ...(await remult.repo(Task).save({ ...task, completed: input.checked })),
    };
  };
  const deleteTask = async (task: Task) => {
    await remult.repo(Task).delete(task);
    tasks = tasks.filter((c) => c.id !== task.id);
  };
</script>

<Tile
  title="Todos"
  subtitle="Fully functional todo app"
  icon=""
  width="full"
  className="todo"
  status="Info"
>
  <main>
    <form onsubmit={addTask}>
      <input
        bind:value={newTaskTitle}
        placeholder="What needs to be done?"
        type="text"
      />
      <button type="submit">
        <img src="plus.svg" alt="Add" />
      </button>
    </form>
    {#each tasks as task, i}
      <div class="todo__task {task.completed ? 'completed' : ''}">
        <input
          id={task.id}
          type="checkbox"
          bind:checked={task.completed}
          onchange={async (e) => {
            const t = await setCompleted(task, e);
            if (hideCompleted && t.completed) {
              tasks.splice(i, 1);
            } else {
              tasks[i] = t;
            }
          }}
        />
        <label for={task.id}>{task.title}</label>
        <span></span>
        <button onclick={() => deleteTask(task)}>
          <img src="trash.svg" alt="Delete" /></button
        >
      </div>
    {/each}
  </main>
  <div class="button-row">
    <button onclick={toggleHideCompleted}>
      {hideCompleted ? "Show" : "Hide"} completed
    </button>
  </div>
</Tile>
