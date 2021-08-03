<script lang="ts">
  export let label: string;
  export let name: string;
  export let hidden = false;

  export let items = [];
  console.log("items", items);
</script>

<fieldset class="row mb-1" class:visually-hidden="{hidden}">
  <legend class="col-form-label col-sm-6 pt-0">{label}</legend>
  <div class="col-sm-6">
    {#each items as item}
      <div class="form-check">
        <input
          class="form-check-input"
          type="radio"
          name="{name}"
          id="{`${item.name}_${item.value}`}"
          value="{item.value}"
        />
        <label class="form-check-label" for="{`${name}_${item.value}`}"
          >{item.name}
        </label>
      </div>
    {/each}
    <div
      class="error-msg"
      id="{`${name}-validation`}"
      data-felte-reporter-dom-for="{name}"
    ></div>
  </div>
</fieldset>
