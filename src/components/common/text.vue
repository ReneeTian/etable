<template>
  <span>{{value}}</span>
</template>
<script>
export default {
  props: ['row', 'column','col'],
  data(){
    return {
      value: ''
    }
  },
  mounted () {
    const formatter = this.column.formatter
    this.value = (formatter && formatter(this.row, this.col, this.row[this.col.prop])) || this.row[this.col.prop]
  }
}
</script>