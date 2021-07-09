<script lang="ts">
  export let label: string;
  export let name: string;
  export let hidden = false;
  export let disabled = false;

  export let items = [];
  console.log("items", items)
</script>

<div class="row mb-1" class:visually-hidden={hidden}>
  <label class="col-form-label col-sm-6 pt-0" for="{name}">{label}</label>
  <div class="col-sm-6">
    <select
      class="form-select"
      id={`${name}-select`}
      name="{name}"
      aria-label={`Select ${label}`}
      disabled={disabled}
    >
      <option value="">Select {label}</option>
      {#each items as item}
        <option value={item.value}>{item.label}</option>
      {/each}
    </select>
    <div
      class="error-msg"
      id={`${name}-validation`}
      data-felte-reporter-dom-for="{name}"
    />
  </div>
</div>
