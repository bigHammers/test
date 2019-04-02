<template>
  <table cellspacing="0" cellpadding="0" border="0" :style="styles">
        <!-- <colgroup>
            <col v-for="(column, index) in columns" :width="setCellWidth(column)">
            <col v-if="$parent.showVerticalScrollBar" :width="$parent.scrollBarWidth"/>
        </colgroup> -->
        <thead>
            <tr v-for="(cols, rowIndex) in headRows" :key="rowIndex">
                <th
                    v-for="(column, index) in cols"
                    :colspan="column.colSpan"
                    :rowspan="column.rowSpan"
                    :key="index"
                    >
                    <div :class="cellClasses(column)">
                        <span v-if="!column.renderHeader">{{ column.title || '' }}</span>
                    </div>
                </th>
                
                <!-- <th v-if="$parent.showVerticalScrollBar && rowIndex===0" :class='scrollBarCellClass()' :rowspan="headRows.length"></th> -->
            </tr>
        </thead>
    </table>
</template>

<script>
export default {
  name: 'TableHead',
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
            // {
            //     [`${this.prefixCls}-hidden`]: !this.fixed && column.fixed && (column.fixed === 'left' || column.fixed === 'right'),
            //     [`${this.prefixCls}-cell-with-selection`]: column.type === 'selection'
            // }
        ];
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style  lang="less">
  
</style>
