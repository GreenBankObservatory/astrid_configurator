<script lang="ts">
    export let label: string;
    export let name: string;
    export let hidden = false;
  
  </script>
  
  <div class="form-check form-switch" class:visually-hidden={hidden}>
    <input class="form-check-input" type="checkbox" name={name} />
    <label class="form-check-label" for={name} >{label}</label >
  </div>
  