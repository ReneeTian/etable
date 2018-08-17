<template>
  <div class="hello">
    <Table 
      ref="mytable"
      :column-type="['selection','index','expand']"
      :columns="columns"
      :dataOption="dataOption">
        <template slot="expand" slot-scope="{ row }">
          <section class="expand-detail">
            <div v-for="col in columns" :key="col.label" v-if="!col.components">
              {{ col.label }}：{{ row[col.prop] }}
            </div>
          </section>
        </template>
      </Table>
  </div>
</template>

<script>
import Table from "./common/Table.vue"
import Button from "./common/cell-btn"
import Data from './common/data.js'
export default {
  name: 'HelloWorld',
  components: {
    Table,
    Button
  },
  data () {
    return {
      msg: 'Welcome to Your Vue.js App',
      columns: [],
      dataOption: {
        rows: Data.data
      },
    }
  },
  mounted(){
    this.columns = [...Data.columns,{
      prop: '',
      label: '操作',
      width: 140,
      className: 'el-button-group',
      components: [{
        propsHandler ({ column, row }) {
          return Object.assign({},{ icon: 'el-icon-edit' , type: 'primary', size: 'small' },{column,row})
        },
        component: Button,
        listeners : {
          'click' (row) {
            event.stopPropagation()
            _this.dialogMode = 'modify';
            _this.domainObj = row;
            _this.domainDialogVisible = true
          }
        }
      },
      {
        propsHandler ({ column, row }) {
          return Object.assign({},{ icon: 'el-icon-delete' , type: 'danger', size: 'small' },{ column, row })
        },
        component: Button,
        listeners : {
          'click' (row) {
            event.stopPropagation()
            _this.confirmDeleteAction(false,row);
          }
        }
      }]
    }]
  }
}
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h1, h2 {
  font-weight: normal;
}
ul {
  list-style-type: none;
  padding: 0;
}
li {
  display: inline-block;
  margin: 0 10px;
}
a {
  color: #42b983;
}
.expand-detail div {
  text-align: left;
  padding: 4px 0;
}
</style>
