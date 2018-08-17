<template>
  <el-button @click="todo" :type="type" :size="size" :icon="icon"></el-button>
</template>
<script>
import ElButton from 'element-ui/lib/button'
export default {
  components: {
    ElButton
  },
  props: ['row', 'column', 'icon', 'type', 'size'],

  methods: {
    todo() {
      this.$emit('click', this.row)
    }
  }
}
</script>
