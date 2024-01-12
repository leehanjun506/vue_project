<template>
  <div class="name">{{name}}</div>
  <button
      class="btn btn-primary"
  v-on:click="updateName">Click</button>
</template>

<script>
import {ref} from "vue";

export default {
  setup() {
    const name = ref('Han jun1');

    // const greeting = (name) =>{
    //   return 'Hello, ' + name;
    // };
    const updateName = () =>{
      name.value = name.value === 'Han jun1' ? 'Han jun2' : 'Han jun1'
      console.log(name);
    }

    return {
      name,
      updateName
    };
  }
}
</script>

<style>
.name {
  color: red;
}
</style>
