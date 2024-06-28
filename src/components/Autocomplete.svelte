<script lang="ts">
  type Choice = {
    value: any;
    label: string;
  };

  export let choices: Choice[] = [];
  export let name: string;
  export let id: string;
  export let selection: Array<any> = [];

  let componentElement: HTMLDivElement;
  let focus = false;
  let search = "";
  let searchResult: Choice[] = [];

  $: searchResult = choices.filter(
    (choice) =>
      choice.label.toLowerCase().includes(search.toLowerCase()) &&
      !selection.includes(choice.value),
  );

  function focusOut(event: FocusEvent) {
    if (!componentElement.contains(event.relatedTarget as Node)) {
      focus = false;
    }
  }

  function inputKeydown(event: KeyboardEvent) {
    if (event.key === "ArrowDown") {
      event.preventDefault();
      const button = componentElement.querySelector("button");
      if (button) {
        button.focus();
      }
    }
  }

  function addSelection(choice: Choice) {
    selection = [...selection, choice.value];
    search = "";
    focus = false;
  }
</script>

<div bind:this={componentElement}>
  {#each selection as value (value)}
    <input type="hidden" {name} {value} />
  {/each}
  {#if selection.length > 0}
    <ul class="selection">
      {#each selection as value (value)}
        <li>
          <button
            on:click|capture|stopImmediatePropagation|stopPropagation|preventDefault={() =>
              (selection = selection.filter((v) => v !== value))}
          >
            x
          </button>
          {choices.find((choice) => choice.value === value)?.label}
        </li>
      {/each}
    </ul>
  {/if}
  <input
    type="text"
    {id}
    placeholder="Tapez pour chercher..."
    bind:value={search}
    on:focus={() => (focus = true)}
    on:blur={focusOut}
    on:focusout={focusOut}
    on:keydown={inputKeydown}
  />
  {#if focus}
    <ul class="choices">
      {#each searchResult as choice (choice.value)}
        <li>
          <button
            on:click|capture|stopImmediatePropagation|stopPropagation|preventDefault={() =>
              addSelection(choice)}
            on:focusout={focusOut}
          >
            {choice.label}
          </button>
        </li>
      {/each}
    </ul>
  {/if}
</div>

<style>
  ul.selection {
    list-style-type: none;
  }

  ul.selection button {
    border: 0;
    background-color: transparent;
    cursor: pointer;
    height: 1.5rem;
    width: 1.5rem;
    color: #666;

    &:hover {
      background-color: #f9f9f9;
    }
  }

  ul.choices {
    list-style-type: none;
    padding: 0;
    margin: 0;
    border: 1px solid #ccc;
    max-height: 200px;
    overflow-y: auto;
  }

  ul.choices li {
    padding: 0;
    cursor: pointer;
  }

  ul.choices li:hover {
    background-color: #f9f9f9;
  }

  ul.choices button {
    text-align: left;
    border: 0;
    padding: 0.5rem;
    width: 100%;
  }
</style>
