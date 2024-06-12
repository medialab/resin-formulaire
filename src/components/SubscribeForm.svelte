<script lang="ts">
  import Autocomplete from "./Autocomplete.svelte";

  export let success = false;

  type LanguageChoice = {
    pt2b: string;
    name: string;
  };

  type SkillChoice = {
    id: number;
    field: string;
    skill: string;
    detail: string;
  };

  type FormField = {
    label: string;
    required: boolean;
    help_text: string;
    choices?: { value: any; display_name: string }[];
  };

  const getFormFields = async (): Promise<Map<string, FormField>> =>
    (
      await (
        await fetch("http://localhost:8000/api/members/", {
          headers: {
            "Content-Type": "application/json",
          },
          method: "OPTIONS",
        })
      ).json()
    ).actions.POST;

  const [getLanguages, getSkills, getFields] = [
    "languages",
    "skills",
    "fields",
  ].map(
    (endpoint) => async () =>
      await (
        await fetch("http://localhost:8000/api/" + endpoint + "/", {
          headers: {
            "Content-Type": "application/json",
          },
          method: "GET",
        })
      ).json(),
  );

  const getChoices = async () => {
    const [languages, skills, fields]: [
      LanguageChoice[],
      SkillChoice[],
      SkillChoice[],
    ] = await Promise.all([getLanguages(), getSkills(), getFields()]);
    return {
      languages: languages.map(({ pt2b, name }) => ({
        value: pt2b,
        label: name,
      })),
      expertise: fields.map(({ id, field }) => ({ value: id, label: field })),
      skills: skills.map(({ id, field, skill, detail }) => ({
        value: id,
        label: `${field} - ${detail ? `${skill} - ${detail}` : skill}`,
      })),
    };
  };

  let errors: Record<string, Array<string>> = {};

  async function submit(event: Event) {
    errors = {};
    const form = event.target as HTMLFormElement;
    const formData = new FormData(form);
    // remove empty fields
    for (const [key, value] of formData.entries()) {
      if (value === "") {
        formData.delete(key);
      }
    }
    const response = await fetch("http://localhost:8000/api/members/", {
      method: "POST",
      body: formData,
    });
    const data = await response.json();
    if (response.status === 400) {
      errors = data;
    }
    if (response.status === 201) {
      success = true;
    }
  }

  function scroll(node: HTMLElement) {
    node.scrollIntoView({ behavior: "smooth", block: "center" });
  }
</script>

<form id="subscribe" on:submit|preventDefault={submit}>
  {#await Promise.all([getFormFields(), getChoices()])}
    <p>Loading...</p>
  {:then [formFields, choices]}
    {#each Object.entries(formFields).filter(([, d]) => !d.read_only) as [id, details] (id)}
      <div class="form-group">
        <label for={id}
          >{details.label}
          {#if details.required}*{/if}</label
        >
        {#if id === "gender"}
          <select
            {id}
            name={id}
            class="form-control"
            required={details.required}
          >
            <option value="">---</option>
            {#each details.choices as gender}
              <option value={gender.value}>{gender.display_name}</option>
            {/each}
          </select>
        {:else if id === "email"}
          <input
            type="email"
            {id}
            name={id}
            class="form-control"
            required={details.required}
          />
        {:else if id === "photo"}
          <input
            type="file"
            {id}
            name={id}
            class="form-control"
            required={details.required}
          />
        {:else if id === "languages" || id === "expertise" || id === "skills"}
          <Autocomplete
            {id}
            name={id}
            choices={choices[id]}
            required={details.required}
          />
        {:else if id === "long_bio" || id === "publications" || id === "training" || id === "comments"}
          <textarea
            {id}
            name={id}
            class="form-control"
            required={details.required}
          ></textarea>
        {:else}
          <input
            type="text"
            {id}
            name={id}
            class="form-control awesomplete"
            required={details.required}
          />
        {/if}
        {#if details.help_text}
          <div id="{id}Help" class="form-text text-muted">
            <small>{details.help_text}</small>
          </div>
        {/if}
        {#if errors[id]}
          <div class="invalid-feedback">
            {#each errors[id] as error}
              <small style:color="#B04838" use:scroll>{error}</small>
            {/each}
          </div>
        {/if}
      </div>
    {/each}
    <button type="submit" class="btn btn-primary">S'inscrire</button>
  {/await}
</form>
