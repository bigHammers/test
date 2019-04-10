<template>
  <table cellspacing="0" cellpadding="0" border="0" :style="styleObject">
        <colgroup>
            <col v-for="(column, index) in columns" :width="setCellWidth(column)">
        </colgroup>
        <tbody :class="[prefixCls + '-tbody']">
            <template v-for="(row, index) in data" >
                    <tr :class="rowClasses(row._index)">
                        <td v-for="column in columns" :class="alignCls(column, row)">
                          {{ row[column.key] }}
                          <!-- <table-cell
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
                        ></table-cell> -->
                         </td>
                    </tr>
                    
            </template>
        </tbody>
    </table>
</template>

<script>
export default {
  name: 'TableBody',
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

  methods:{
    alignCls (column, row = {}) {
        let cellClassName = '';
        if (row.cellClassName && column.key && row.cellClassName[column.key]) {
            cellClassName = row.cellClassName[column.key];
        }
        return [
            {
                // [`${cellClassName}`]: cellClassName,    // cell className
                [`${column.className}`]: column.className,    // column className
                [`${this.prefixCls}-column-${column.align}`]: column.align,
                // [`${this.prefixCls}-hidden`]: (this.fixed === 'left' && column.fixed !== 'left') || (this.fixed === 'right' && column.fixed !== 'right') || (!this.fixed && column.fixed && (column.fixed === 'left' || column.fixed === 'right'))
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
    }
  }

}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style  lang="less">
  
</style>
