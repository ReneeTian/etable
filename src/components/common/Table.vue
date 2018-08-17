<template>
  <div class="el-table-self">
    <el-table v-loading="loading" @row-click="rowClick" :data="tableData" :height="tableHeight" border @selection-change="selectionChange" style="width: 100%">
      <template v-for="tp in typesColumns">
        <el-table-column v-if="tp.type === 'expand'" v-bind="tp.props" type="expand" :key="tp.type">
          <template slot-scope="props">
            <slot name="expand" v-bind="props"></slot>
          </template>
        </el-table-column>
        <el-table-column v-else :key="tp.type" :type="tp.type" v-bind="tp.props">
        </el-table-column>
      </template>
      <template v-for="column in renderColumns">
        <el-table-column :key="column.label" v-bind="getColBind(column)">
          <template slot-scope="scope">
            <template v-for="cp in column.components">
              <component :is="cp.component" v-bind="getCptBind(scope, column, cp)" v-on="cp.listeners">
              </component>
            </template>
          </template>
        </el-table-column>
      </template>
      <template v-if="slotAppend" slot="append">
        <slot name="append"></slot>
      </template>
    </el-table>
    <div class="pagination-footer">
      <span class="description">{{description}}</span>
      <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="currentPage" :page-sizes="pageSizes" :page-size="tpageSize" layout="total, sizes, prev, pager, next, jumper" :total="totalCount">
      </el-pagination>
    </div>
  </div>
</template>
<script>
import Vue from 'vue'
import 'element-ui/lib/theme-chalk/index.css'
import Loading from 'element-ui/lib/loading'
import ElTable from 'element-ui/lib/table'
import ElTableColumn from 'element-ui/lib/table-column'
import ElPagination from 'element-ui/lib/pagination'
import TextNode from './text.vue'
Vue.use(Loading)
const TYPES = ['selection', 'expand', 'index']
//表格列默认加载方式
const COLUMN_PROPS = {
  align: 'left',
  components: [{ component: TextNode }]
}
export default {
  components: {
    ElTable,
    ElTableColumn,
    ElPagination,
    TextNode
  },
  data() {
    return {
      currentPage: 1,
      tpageSize: 10,
      totalCount: 0,
      tableData: [],
      loading: true
    }
  },
  props: {
    rowClick: {
      type: Function,
      default () {
        return function() {}
      }
    },
    slotAppend: Boolean,
    columnType: [String, Array], //特殊列，多选、扩展、索引
    columnTypeProps: Object, //特殊列定义方法
    columnsProps: { //定义所有 columns 公共的属性
      type: Object,
      default () {
        return {
          // width: 120,
          //定义表格如何渲染 
          components: [{ component: TextNode }]
        }
      }
    },
    /**
    *
    @param dataOption {Object} 
    *url 接口地址  默认获取数据方式为url获取
    *method get/post 请求方式
    *queryParams 额外参数 
    *ajaxMethod 自定义获取参数方式
    *rows 通过props直接渲染
    *
    */
    dataOption: Object,
    columns: Array, //表格列配置数据,{vlaue:对应数据对象中的属性，label：对应的是标题文字，className：对应的是列的样式类名}
    description: String, //分页脚底左侧的数据说明
    tableHeight: Number, //分页列表的高度
    pageSizes: {
      type: Array,
      default () {
        return [5, 10, 15, 20, 25]
      }
    }, //决定每页显示的条数[10,15,20,25]
    pageSize: Number, //每页显示的条数
    cPage: Number //当前页数
  },
  computed: {
    // 用于渲染特殊列,索引、扩展、多选
    typesColumns() {
      const { columnType: type, columnTypeProps } = this
      // console.log(type)
      let typeColums = []
      if (typeof type === 'string' && ~TYPES.indexOf(type)) {
        typeColums = [type]
      }
      if (Array.isArray(type)) {
        typeColums = type.filter(it => ~TYPES.indexOf(it))
      }
      const map = columnTypeProps || {}
      return typeColums.map(type => {
        return {
          type,
          props: map[type]
        }
      })
    },
    renderColumns() {
      const {
        columns,
        columnsProps: props,
      } = this
      const renderColumns = columns.map(col => {
        const it = Object.assign({}, COLUMN_PROPS, props, col)
        return it
      })
      return renderColumns
    },

  },
  created() {
    console.log('created table')
  },
  mounted() {
    this.$nextTick(() => {
      this.getTableData();
    });
  },
  methods: {
    getColBind(col) {
      //处理表格列中自定义属性
      //debugger
      const bind = Object.assign({}, col)
      delete bind.components
      return bind
    },
    getCptBind({ row, column }, cols, cp) {
      //处理propHandler中自定义属性
      // debugger
      const col = Object.assign({}, cols, cp);
      const props = { row, column, col }
      const handler = col.propsHandler
      return handler && handler(props) || props
    },
    getTableData() {
      if (this.dataOption.rows) {
        this.tableData = this.dataOption.rows;
        this.totalCount = this.dataOption.rows.length;
        this.loading = false;
      } else if (this.dataOption.ajaxMethod) {
        //自定义获取数据方法，需要返回promise对象
        this.tableData = this.dataOption.ajaxMethod().then((res) => {
          this.loading = false;
        }).catch((err) => {
          this.loading = false;
        });
      } else {
        if (this.dataOption.method && this.dataOption.method == 'post') {
          let postData = Object.assign({}, {
            pageNum: this.currentPage,
            pageSize: this.tpageSize
          }, this.dataOption.queryParams);
          this.$axios.post(this.dataOption.url, { params: postData }).then((res) => {
            this.tableData = res.data;
            this.totalCount = res.pageInfo.total;
            this.loading = false;
          });
        } else {
          let postData = Object.assign({}, {
            pageNum: this.currentPage,
            pageSize: this.tpageSize
          }, this.dataOption.queryParams);
          this.$axios.get(this.dataOption.url, { params: postData }).then((res) => {
            this.tableData = res.data;
            this.totalCount = res.pageInfo.total;
            this.loading = false;
          });
        }
      }
    },
    refreshTable() {
      this.getTableData();
    },
    handleSizeChange: function(size) {
      this.currentPage = 1
      this.tpageSize = size;
      this.refreshTable();
    },
    handleCurrentChange: function(currentPage) {
      this.currentPage = currentPage;
      this.refreshTable();
    },
    selectionChange: function(selections) {
      this.$emit("selectionChange", selections);
    }
  }
}

</script>
<style>
.el-table-self {
  margin-left: 16px;
  margin-right: 16px;
}

.pagination-footer .description {
  float: left;
  margin-left: 20px;
  margin-top: 12px;
}

.pagination-footer .el-pagination {
  float: right;
  margin-top: 8px;
  margin-bottom: 8px;
}

.el-table__empty-block {
  position: relative;
  min-height: 60px;
  text-align: center;
  width: 100%;
  height: 100%;
}

.el-table td,
.el-table th {
  height: 30px;
  min-width: 0;
  text-overflow: ellipsis;
  vertical-align: middle;
}

.el-table__empty-text {
  position: absolute;
  left: 50%;
  width: 110px;
  height: 110px;
  top: 50%;
  line-height: 220px;
  -ms-transform: translate(-50%, -50%);
  transform: translate(-50%, -50%);
  color: #5e7382;
  background-position: center center;
  background-repeat: no-repeat;
  background-size: cover;
}

</style>
