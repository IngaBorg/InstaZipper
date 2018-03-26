<template>
  <div class="row">
     <div class="col align-self-center">
       <p class="display-1 text-primary">Processing</p>
       <p class="display-4">{{progress}}/{{total}}</p>
      </div>
  </div>
</template>

<script>
export default {
  props: {
    progress: null,
    total: null
  }
}
</script>

<style>
div.row {
  transition: .5s;
}
</style>
