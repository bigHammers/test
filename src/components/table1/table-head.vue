<template>
  <table cellspacing="0" cellpadding="0" border="0" :style="styles">
        <colgroup>
            <col v-for="(column, index) in columns" :width="setCellWidth(column)" />
            <!-- <col v-if="$parent.showVerticalScrollBar" :width="$parent.scrollBarWidth"/> -->
        </colgroup>
        <thead>
            <tr v-for="(cols, rowIndex) in headRows">
                <th
                    v-for="(column, index) in cols"
                    :colspan="column.colSpan"
                    :rowspan="column.rowSpan"
                    :key="index"
                    >
                    <div :class="cellClasses(column)">
                        <template v-if="column.type === 'expand'">
                            <span v-if="!column.renderHeader">{{ column.title || '' }}</span>
                            <render-header v-else :render="column.renderHeader" :column="column" :index="index"></render-header>
                        </template>
                        <template v-else>
                            <span v-if="!column.renderHeader" :class="{[prefixCls + '-cell-sort']: column.sortable}" @click="handleSortByHead(getColumn(rowIndex, index)._index)">{{ column.title || '#' }}</span>
                            <render-header v-else :render="column.renderHeader" :column="column" :index="index"></render-header>
                            <span :class="[prefixCls + '-sort']" v-if="column.sortable">
                                <i class="ivu-icon ivu-icon-md-arrow-dropup" :class="{on: getColumn(rowIndex, index)._sortType === 'asc'}" @click="handleSort(getColumn(rowIndex, index)._index, 'asc')"></i>
                                <i class="ivu-icon ivu-icon-md-arrow-dropdown" :class="{on: getColumn(rowIndex, index)._sortType === 'desc'}" @click="handleSort(getColumn(rowIndex, index)._index, 'desc')"></i>
                            </span>
                        </template>
                    </div>
                </th>
                
                <!-- <th v-if="$parent.showVerticalScrollBar && rowIndex===0" :class='scrollBarCellClass()' :rowspan="headRows.length"></th> -->
            </tr>
        </thead>
    </table>
</template>

<script>
import renderHeader from './header';
export default {
  name: 'TableHead',
  components:{
      renderHeader
  },
  props: {
    prefixCls: String,
    styleObject: Object,
    columns: Array,
    objData: Object,
    data: Array,    // rebuildData
    columnsWidth: Object,
    fixed: {
        type: [Boolean, String],
        default: false
    },
    columnRows: Array,
    fixedColumnRows: Array
  },
  computed:{
    styles () {
        const style = Object.assign({}, this.styleObject);
        const width = parseInt(this.styleObject.width) ;
        style.width = `${width}px`;
        return style;
    },
    headRows () {
        const isGroup = this.columnRows.length > 1;
        if (isGroup) {
            return this.fixed ? this.fixedColumnRows : this.columnRows;
        } else {
            return [this.columns];
        }
    }
  },
  methods:{
    cellClasses (column) {
        return [
            `${this.prefixCls}-cell`,
            {
                [`${this.prefixCls}-hidden`]: !this.fixed && column.fixed && (column.fixed === 'left' || column.fixed === 'right'),
                [`${this.prefixCls}-cell-with-selection`]: column.type === 'selection'
            }
        ];
    },
    setCellWidth (column) {
        let width = '';
        if (column.width) {
            width = column.width;
        } else if (this.columnsWidth[column._index]) {
            width = this.columnsWidth[column._index].width;
        }
        if (width === '0') width = '';
        return width;
    },
    getColumn (rowIndex, index) {
        const isGroup = this.columnRows.length > 1;
        if (isGroup) {
            const id = this.headRows[rowIndex][index].__id;
            return this.columns.filter(item => item.__id === id)[0];
        } else {
            return this.headRows[rowIndex][index];
        }
    },
    //排序部分
    handleSort (index, type) {
        const column = this.columns[index];
        const _index = column._index;

        if (column._sortType === type) {
            type = 'normal';
        }
        this.$parent.handleSort(_index, type);
    },
    handleSortByHead (index) {
        const column = this.columns[index];
        if (column.sortable) {
            const type = column._sortType;
            if (type === 'normal') {
                this.handleSort(index, 'asc');
            } else if (type === 'asc') {
                this.handleSort(index, 'desc');
            } else {
                this.handleSort(index, 'normal');
            }
        }
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style  lang="less">
  
</style>
