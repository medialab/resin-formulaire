<script lang="ts">
  type Choice = {
    value: any;
    label: string;
  };

  export let choices: Choice[] = [];
  export let name: string;
  export let id: string;
  export let required: boolean;
  export let selection: Array<any> = [];

  let componentElement: HTMLDivElement;
  let inputElement: HTMLInputElement;
  let inputValue = "";
  let focus = false;
  let search = "";
  let searchResult: Choice[] = [];

  let selectionDisplay: string;

  $: selectionDisplay = selection.length
    ? selection
        .map((value) => {
          const choice = choices.find((choice) => choice.value === value);
          return choice ? choice.label : value;
        })
        .join(", ") + (focus ? ", " : "")
    : "";
  $: inputValue = selectionDisplay + search;

  $: searchResult = choices.filter((choice) =>
    choice.label.toLowerCase().includes(search.toLowerCase()),
  );

  function updateSearch(event: Event) {
    const value = (event.target as HTMLInputElement).value;
    if (
      search === "" &&
      value.length < selectionDisplay.length &&
      selection.length > 0
    ) {
      selection = selection.slice(0, -1);
      search = "";
    } else if (value.startsWith(selectionDisplay)) {
      search = value.replace(selectionDisplay, "");
    } else {
      (event.target as HTMLInputElement).value = selectionDisplay;
    }
  }

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
    inputElement.focus();
  }
</script>

<div bind:this={componentElement}>
  {#each selection as value (value)}
    <input type="hidden" {name} {value} />
  {/each}
  <input
    bind:this={inputElement}
    type="text"
    {id}
    {required}
    value={inputValue}
    on:input={updateSearch}
    on:focus={() => (focus = true)}
    on:focusout={focusOut}
    on:keydown={inputKeydown}
  />
  {#if focus}
    <ul>
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
  ul {
    list-style-type: none;
    padding: 0;
    margin: 0;
    border: 1px solid #ccc;
    max-height: 200px;
    overflow-y: auto;
  }

  li {
    padding: 0;
    cursor: pointer;
  }

  button {
    text-align: left;
    border: 0;
    padding: 0.5rem;
    width: 100%;
  }

  li:hover {
    background-color: #f9f9f9;
  }
</style>
