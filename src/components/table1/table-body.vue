<template>
  <table cellspacing="0" cellpadding="0" border="0" :style="styleObject">
        <colgroup>
            <col v-for="(column, index) in columns" :width="setCellWidth(column)">
        </colgroup>
        <tbody :class="[prefixCls + '-tbody']">
            <template v-for="(row, index) in data" >
                <table-tr
                    :row="row"
                    :key="row._rowKey"
                    :prefix-cls="prefixCls"
                    @click.native="clickCurrentRow(row._index)"
                    >
                    <td v-for="column in columns" :class="alignCls(column, row)">
                        <table-cell
                            :fixed="fixed"
                            :prefix-cls="prefixCls"
                            :row="row"
                            :key="column._columnKey"
                            :column="column"
                            :natural-index="index"
                            :index="row._index"
                            :checked="rowChecked(row._index)"
                            :disabled="rowDisabled(row._index)"
                            :expanded="rowExpanded(row._index)"
                        ></table-cell>
                    </td>
                </table-tr>
                <tr v-if="rowExpanded(row._index)" :class="{[prefixCls + '-expanded-hidden']: fixed}">
                    <td :colspan="columns.length" :class="prefixCls + '-expanded-cell'">
                        <Expand :key="row._rowKey" :row="row" :render="expandRender" :index="row._index"></Expand>
                    </td>
                </tr>
                    
            </template>
        </tbody>
    </table>
</template>

<script>
import TableTr from './table-tr.vue';
import TableCell from './cell.vue';
import Expand from './expand.js';
export default {
  name: 'TableBody',
  components: { TableCell, TableTr,Expand },
  props: {
    prefixCls: String,
    styleObject: Object,
    columns: Array,
    data: Array,    // rebuildData
    objData: Object,
    columnsWidth: Object,
    fixed: {
        type: [Boolean, String],
        default: false
    }
  },
  computed:{
      expandRender () {
            let render = function () {
                return '';
            };
            for (let i = 0; i < this.columns.length; i++) {
                const column = this.columns[i];
                if (column.type && column.type === 'expand') {
                    if (column.render) render = column.render;
                }
            }
            return render;
        }
  },
  methods:{
    alignCls (column, row = {}) {
        let cellClassName = '';
        if (row.cellClassName && column.key && row.cellClassName[column.key]) {
            cellClassName = row.cellClassName[column.key];
        }
        return [
            {
                [`${cellClassName}`]: cellClassName,    // cell className
                [`${column.className}`]: column.className,    // column className
                [`${this.prefixCls}-column-${column.align}`]: column.align,
                [`${this.prefixCls}-hidden`]: (this.fixed === 'left' && column.fixed !== 'left') || (this.fixed === 'right' && column.fixed !== 'right') || (!this.fixed && column.fixed && (column.fixed === 'left' || column.fixed === 'right'))
            }
        ];
    },
    rowClasses (_index) {
        return [
            `${this.prefixCls}-row`,
            this.rowClsName(_index),
        ];
    },
    rowClsName (_index) {
        return this.$parent.rowClassName(this.objData[_index], _index);
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
    rowChecked (_index) {
        return this.objData[_index] && this.objData[_index]._isChecked;
    },
    rowDisabled(_index){
        return this.objData[_index] && this.objData[_index]._isDisabled;
    },
    rowExpanded(_index){
        return this.objData[_index] && this.objData[_index]._isExpanded;
    },
    clickCurrentRow (_index) {
        this.$parent.clickCurrentRow(_index);
    },
  }

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style  lang="less">
  
</style>
