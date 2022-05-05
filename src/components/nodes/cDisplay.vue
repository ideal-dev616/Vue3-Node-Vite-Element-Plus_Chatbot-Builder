<template>
  <div ref="el">
    <nodeHeader  title="Display"/>
    <p>Add a multimodel display</p>
  </div>
</template>

<script>
import { defineComponent, ref, getCurrentInstance, nextTick, onMounted } from 'vue'
import nodeHeader from './nodeHeader.vue'
export default defineComponent({
    components: {
        nodeHeader
    },
    setup(){}
})
</script>