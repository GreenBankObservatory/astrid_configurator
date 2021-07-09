<script lang="ts">
  export let label: string;
  export let name: string;
  // export let clean; // TODO: Func type
  export let hidden = false;

  let props = {
    class: "form-control",
    type: "text",
    name: name,
  }
</script>

<div class="row mb-1" class:visually-hidden={hidden}>
  <label class="col-sm-6 col-form-label" for={name}>{label}</label >
    <div class="col-sm-6">
    <input {...props} />
    <div
      class="error-msg"
      id={`${name}-validation`}
      data-felte-reporter-dom-for={name}
    />
  </div>
</div>
