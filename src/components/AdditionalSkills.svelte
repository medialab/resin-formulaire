<script lang="ts">
  import Fuse, { type FuseResult } from "fuse.js";

  export let id: string;
  export let value: string = "";
  export let choices: { value: number; label: string }[];
  export let add: (value: number) => void;

  let match: FuseResult<{ value: number; label: string }>[];

  const fuse = new Fuse(choices, {
    keys: ["label"],
  });

  function warning(event: Event) {
    const input = event.target as HTMLInputElement;
    match = fuse.search(input.value);
  }
</script>

<input
  type="text"
  {id}
  name={id}
  class="form-control"
  {value}
  on:change={warning}
/>
{#if match?.length}
  <div class="form-text text-danger">
    Les compétences existantes suivantes existent déjà et peuvent être ajoutées
    directement : {#each match.slice(0, 4) as { item }, i (item.value)}
      <a
        href="#"
        on:click|preventDefault|stopPropagation={() => {
          add(item.value);
          value = "";
        }}>{item.label}</a
      >
      {#if i < match.slice(0, 4).length - 2},
      {:else if i < match.slice(0, 4).length - 1}
        et
      {:else}.{/if}
    {/each}
  </div>
{/if}
