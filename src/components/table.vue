<template>
  <div class="table-wrapper">
    <div :class="classes">
      <!-- <div :class="[prefixCls + '-title']" v-if="showSlotHeader" ref="title"><slot name="header"></slot></div> -->
      <div :class="[prefixCls + '-header']" v-if="showHeader" ref="header" @mousewheel="handleMouseWheel">
        <table-head :prefix-cls="prefixCls" :styleObject="tableHeaderStyle" :columns="cloneColumns" :column-rows="columnRows" :obj-data="objData" :columns-width="columnsWidth" :data="rebuildData"></table-head>
      </div>
      <div :class="[prefixCls + '-body']" :style="bodyStyle" ref="body" @scroll="handleBodyScroll"
          v-show="!(((!data || data.length === 0)) || ((!rebuildData || rebuildData.length === 0)))">
          <table-body
              ref="tbody"
              :prefix-cls="prefixCls"
              :styleObject="tableStyle"
              :columns="cloneColumns"
              :data="rebuildData"
              :columns-width="columnsWidth"
              :obj-data="objData"></table-body>
      </div>
      <!-- <div
                      :class="[prefixCls + '-tip']" :style="bodyStyle" @scroll="handleBodyScroll"
                      v-show="((!!localeNoDataText && (!data || data.length === 0)) || (!!localeNoFilteredDataText && (!rebuildData || rebuildData.length === 0)))">
                      <table cellspacing="0" cellpadding="0" border="0">
                          <tbody>
                              <tr>
                                  <td :style="{'height':bodyStyle.height,'width':`${this.headerWidth}px`}">
                                      <span v-html="localeNoDataText" v-if="!data || data.length === 0"></span>
                                      <span v-html="localeNoFilteredDataText" v-else></span>
                                  </td>
                              </tr>
                          </tbody>
                      </table>
                  </div>
                  <div :class="[prefixCls + '-fixed']" :style="fixedTableStyle" v-if="isLeftFixed">
                      <div :class="fixedHeaderClasses" v-if="showHeader">
                          <table-head
                              fixed="left"
                              :prefix-cls="prefixCls"
                              :styleObject="fixedTableStyle"
                              :columns="leftFixedColumns"
                              :column-rows="columnRows"
                              :fixed-column-rows="leftFixedColumnRows"
                              :obj-data="objData"
                              :columns-width="columnsWidth"
                              :data="rebuildData"></table-head>
                      </div>
                      <div :class="[prefixCls + '-fixed-body']" :style="fixedBodyStyle" ref="fixedBody" @mousewheel="handleFixedMousewheel" @DOMMouseScroll="handleFixedMousewheel">
                          <table-body
                              fixed="left"
                              :prefix-cls="prefixCls"
                              :styleObject="fixedTableStyle"
                              :columns="leftFixedColumns"
                              :data="rebuildData"
                              :columns-width="columnsWidth"
                              :obj-data="objData"></table-body>
                      </div>
                  </div>
                  <div :class="[prefixCls + '-fixed-right']" :style="fixedRightTableStyle" v-if="isRightFixed">
                      <div :class="fixedHeaderClasses" v-if="showHeader">
                          <table-head
                              fixed="right"
                              :prefix-cls="prefixCls"
                              :styleObject="fixedRightTableStyle"
                              :columns="rightFixedColumns"
                              :column-rows="columnRows"
                              :fixed-column-rows="rightFixedColumnRows"
                              :obj-data="objData"
                              :columns-width="columnsWidth"
                              :data="rebuildData"></table-head>
                      </div>
                      <div :class="[prefixCls + '-fixed-body']" :style="fixedBodyStyle" ref="fixedRightBody" @mousewheel="handleFixedMousewheel" @DOMMouseScroll="handleFixedMousewheel">
                          <table-body
                              fixed="right"
                              :prefix-cls="prefixCls"
                              :styleObject="fixedRightTableStyle"
                              :columns="rightFixedColumns"
                              :data="rebuildData"
                              :columns-width="columnsWidth"
                              :obj-data="objData"></table-body>
                      </div>
                  </div>
                  <div :class="[prefixCls + '-fixed-right-header']" :style="fixedRightHeaderStyle" v-if="isRightFixed"></div> -->
    </div>
  </div>
</template>

<script>
  import tableHead from './table-head.vue';
  import tableBody from './table-body.vue';
  import {
    getAllColumns,
    convertToRows,
    convertColumnOrder,
    getRandomStr
  } from '../utils/util';
  import {
    deepCopy
  } from '../utils/assist';
  const prefixCls = 'table';
  let rowKey = 1;
  let columnKey = 1;
  export default {
    name: 'Table',
    components: {
      tableHead,
      tableBody
    },
    props: {
      data: { //数据源
        type: Array,
        default () {
          return [];
        }
      },
      columns: { //表头的数据
        type: Array,
        default () {
          return [];
        }
      },
      showHeader: {
        type: Boolean,
        default: true
      },
      width: {
        type: [Number, String]
      },
      height: {
        type: [Number, String]
      },
      rowClassName: {
        type: Function,
        default () {
          return '';
        }
      },
      loading: {
        type: Boolean,
        default: false
      }
    },
    data() {
      const colsWithId = this.makeColumnsId(this.columns);
      return {
        ready: false,
        tableWidth: 0,
        columnsWidth: {},
        prefixCls: prefixCls,
        compiledUids: [],
        objData: this.makeObjData(), // checkbox or highlight-row
        rebuildData: [], // for sort or filter
        cloneColumns: this.makeColumns(colsWithId),
        columnRows: this.makeColumnRows(false, colsWithId),
        leftFixedColumnRows: this.makeColumnRows('left', colsWithId),
        rightFixedColumnRows: this.makeColumnRows('right', colsWithId),
        allColumns: getAllColumns(colsWithId), // for multiple table-head, get columns that have no children
        showSlotHeader: true,
        showSlotFooter: true,
        bodyHeight: 0,
        currentContext: this.context,
        cloneData: deepCopy(this.data), // when Cell has a button to delete row data, clickCurrentRow will throw an error, so clone a data
        showVerticalScrollBar: false,
        showHorizontalScrollBar: false,
        headerWidth: 0,
        headerHeight: 0,
      }
    },
    computed: {
      classes() {
        return [
          `${prefixCls}`,
          {
            [`${prefixCls}-with-fixed-top`]: !!this.height
          }
        ];
      },
      tableHeaderStyle() {
        let style = {};
        if (this.tableWidth !== 0) {
          let width = '';
          width = this.tableWidth;
          style.width = `${width}px`;
        }
        return style;
      },
      bodyStyle () {
          let style = {};
          if (this.bodyHeight !== 0) {
              const height = this.bodyHeight;
              style.height = `${height}px`;
          }
          return style;
      },
      tableStyle () {
          let style = {};
          if (this.tableWidth !== 0) {
              let width = '';
              if (this.bodyHeight === 0) {
                  width = this.tableWidth;
              } else {
                  width = this.tableWidth - (this.showVerticalScrollBar?this.scrollBarWidth:0);
              }
//                    const width = this.bodyHeight === 0 ? this.tableWidth : this.tableWidth - this.scrollBarWidth;
              style.width = `${width}px`;
          }
          return style;
      },
    },
    methods: {
      makeColumnsId(columns) {
        return columns.map(item => {
          if ('children' in item) this.makeColumnsId(item.children);
          item.__id = getRandomStr(6);
          return item;
        });
      },
      handleMouseWheel(event) {
        const deltaX = event.deltaX;
        const $body = this.$refs.body;
        if (deltaX > 0) {
          $body.scrollLeft = $body.scrollLeft + 10;
        } else {
          $body.scrollLeft = $body.scrollLeft - 10;
        }
      },
      hideColumnFilter () {
          this.cloneColumns.forEach((col) => col._filterVisible = false);
      },
      handleBodyScroll (event) {
          if (this.showHeader) this.$refs.header.scrollLeft = event.target.scrollLeft;
          // if (this.isLeftFixed) this.$refs.fixedBody.scrollTop = event.target.scrollTop;
          // if (this.isRightFixed) this.$refs.fixedRightBody.scrollTop = event.target.scrollTop;
          this.hideColumnFilter();
      },
      
      handleResize() {
        //let tableWidth = parseInt(getStyle(this.$el, 'width')) - 1;
        let tableWidth = this.$el.offsetWidth - 1;
        let columnsWidth = {};
        let sumMinWidth = 0;
        let hasWidthColumns = [];
        let noWidthColumns = [];
        let maxWidthColumns = [];
        let noMaxWidthColumns = [];
        this.cloneColumns.forEach((col) => {
          if (col.width) {
            hasWidthColumns.push(col);
          } else {
            noWidthColumns.push(col);
            if (col.minWidth) {
              sumMinWidth += col.minWidth;
            }
            if (col.maxWidth) {
              maxWidthColumns.push(col);
            } else {
              noMaxWidthColumns.push(col);
            }
          }
          col._width = null;
        });
        let unUsableWidth = hasWidthColumns.map(cell => cell.width).reduce((a, b) => a + b, 0);
        let usableWidth = tableWidth - unUsableWidth - sumMinWidth - (this.showVerticalScrollBar ? this.scrollBarWidth : 0) - 1;
        let usableLength = noWidthColumns.length;
        let columnWidth = 0;
        if (usableWidth > 0 && usableLength > 0) {
          columnWidth = parseInt(usableWidth / usableLength);
        }
        for (let i = 0; i < this.cloneColumns.length; i++) {
          const column = this.cloneColumns[i];
          let width = columnWidth + (column.minWidth ? column.minWidth : 0);
          if (column.width) {
            width = column.width;
          } else {
            if (column._width) {
              width = column._width;
            } else {
              if (column.minWidth > width) {
                width = column.minWidth;
              } else if (column.maxWidth < width) {
                width = column.maxWidth;
              }
              if (usableWidth > 0) {
                usableWidth -= width - (column.minWidth ? column.minWidth : 0);
                usableLength--;
                if (usableLength > 0) {
                  columnWidth = parseInt(usableWidth / usableLength);
                } else {
                  columnWidth = 0;
                }
              } else {
                columnWidth = 0;
              }
            }
          }
          column._width = width;
          columnsWidth[column._index] = {
            width: width
          };
        }
        if (usableWidth > 0) {
          usableLength = noMaxWidthColumns.length;
          columnWidth = parseInt(usableWidth / usableLength);
          for (let i = 0; i < noMaxWidthColumns.length; i++) {
            const column = noMaxWidthColumns[i];
            let width = column._width + columnWidth;
            if (usableLength > 1) {
              usableLength--;
              usableWidth -= columnWidth;
              columnWidth = parseInt(usableWidth / usableLength);
            } else {
              columnWidth = 0;
            }
            column._width = width;
            columnsWidth[column._index] = {
              width: width
            };
          }
        }
        this.tableWidth = this.cloneColumns.map(cell => cell._width).reduce((a, b) => a + b, 0) + (this.showVerticalScrollBar ? this.scrollBarWidth : 0) + 1;
        this.columnsWidth = columnsWidth;
        // this.fixedHeader();
      },
      makeObjData() {
        let data = {};
        this.data.forEach((row, index) => {
          const newRow = deepCopy(row); // todo 直接替换
          newRow._isHover = false;
          if (newRow._disabled) {
            newRow._isDisabled = newRow._disabled;
          } else {
            newRow._isDisabled = false;
          }
          if (newRow._checked) {
            newRow._isChecked = newRow._checked;
          } else {
            newRow._isChecked = false;
          }
          if (newRow._expanded) {
            newRow._isExpanded = newRow._expanded;
          } else {
            newRow._isExpanded = false;
          }
          if (newRow._highlight) {
            newRow._isHighlight = newRow._highlight;
          } else {
            newRow._isHighlight = false;
          }
          data[index] = newRow;
        });
        return data;
      },
      makeColumns(cols) {
        // 在 data 时，this.allColumns 暂时为 undefined
        let columns = deepCopy(getAllColumns(cols));
        let left = [];
        let right = [];
        let center = [];
        columns.forEach((column, index) => {
          column._index = index;
          column._columnKey = columnKey++;
          column.width = parseInt(column.width);
          column._width = column.width ? column.width : ''; // update in handleResize()
          column._sortType = 'normal';
          column._filterVisible = false;
          column._isFiltered = false;
          column._filterChecked = [];
          if ('filterMultiple' in column) {
            column._filterMultiple = column.filterMultiple;
          } else {
            column._filterMultiple = true;
          }
          if ('filteredValue' in column) {
            column._filterChecked = column.filteredValue;
            column._isFiltered = true;
          }
          if ('sortType' in column) {
            column._sortType = column.sortType;
          }
          if (column.fixed && column.fixed === 'left') {
            left.push(column);
          } else if (column.fixed && column.fixed === 'right') {
            right.push(column);
          } else {
            center.push(column);
          }
        });
        return left.concat(center).concat(right);
      },
      makeColumnRows(fixedType, cols) {
        return convertToRows(cols, fixedType);
      },
      makeData () {
          let data = deepCopy(this.data);
          data.forEach((row, index) => {
              row._index = index;
              row._rowKey = rowKey++;
          });
          return data;
      },
      makeDataWithSort () {
          let data = this.makeData();
          let sortType = 'normal';
          let sortIndex = -1;
          let isCustom = false;

          for (let i = 0; i < this.cloneColumns.length; i++) {
              if (this.cloneColumns[i]._sortType !== 'normal') {
                  sortType = this.cloneColumns[i]._sortType;
                  sortIndex = i;
                  isCustom = this.cloneColumns[i].sortable === 'custom';
                  break;
              }
          }
          if (sortType !== 'normal' && !isCustom) data =  this.sortData(data, sortType, sortIndex);
          return data;
      },
      filterData (data, column) {
          return data.filter((row) => {
              //如果定义了远程过滤方法则忽略此方法
              if (typeof column.filterRemote === 'function') return true;

              let status = !column._filterChecked.length;
              for (let i = 0; i < column._filterChecked.length; i++) {
                  status = column.filterMethod(column._filterChecked[i], row);
                  if (status) break;
              }
              return status;
          });
      },
      makeDataWithSortAndFilter () {
          let data = this.makeDataWithSort();
          this.cloneColumns.forEach(col => data = this.filterData(data, col));
          return data;
      },
    },
    created(){
        this.rebuildData = this.makeDataWithSortAndFilter();

    },
    mounted(){
      this.handleResize();
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="less">
  @table-prefix-cls: ~"table";
  @table-select-item-prefix-cls: ~"@{table-prefix-cls}-filter-select-item";
  @border-color-base: #ECEEF5;
  .@{table-prefix-cls} {
    &-wrapper {
      position: relative;
      border: 1px solid @border-color-base;
      border-bottom: 0;
      border-right: 0;
    }
    width: inherit;
    height: 100%;
    max-width: 100%;
    overflow: hidden; // color: @text-color;
    // font-size: @font-size-small;
    background-color: #fff;
    box-sizing: border-box; //position: relative;
    &-hide {
      opacity: 0;
    }
    &:before {
      content: '';
      width: 100%;
      height: 1px;
      position: absolute;
      left: 0;
      bottom: 0;
      background-color: @border-color-base;
      z-index: 1;
    }
    &:after {
      content: '';
      width: 1px;
      height: 100%;
      position: absolute;
      top: 0;
      right: 0;
      background-color: @border-color-base;
      z-index: 3;
    }
    &-with-header {
      //border-radius: @border-radius-base @border-radius-base 0 0;
    }
    &-with-footer {
      //border: 1px solid @border-color-base;
      //border-radius: 0 0 @border-radius-base @border-radius-base;
    }
    &-with-header&-with-footer {
      //border-radius: @border-radius-base;
    }
    &-title,
    &-footer {
      height: 48px;
      line-height: 48px; // border-bottom: 1px solid @border-color-split;
    }
    &-footer {
      border-bottom: none;
    }
    &-header {
      overflow: hidden;
    }
    &-body {
      //overflow: auto;
      //position: relative;
    }
    &-overflowX {
      overflow-x: scroll;
    }
    &-overflowY {
      overflow-y: scroll;
    }
    &-tip {
      overflow-x: auto;
      overflow-y: hidden; //position: relative;
    }
    &-with-fixed-top&-with-footer {
      .@{table-prefix-cls}-footer {
        border-top: 1px solid @border-color-base;
      }
      tbody tr:last-child td {
        border-bottom: none;
      }
    }
    th,
    td {
      min-width: 0;
      height: 48px;
      box-sizing: border-box;
      text-align: left;
      text-overflow: ellipsis;
      vertical-align: middle; //position: relative;
      border-bottom: 1px solid @border-color-base;
    }
    th {
      height: 40px;
      white-space: nowrap;
      overflow: hidden; // background-color: @table-thead-bg;
    }
    td {
      background-color: #fff; // transition: background-color @transition-time @ease-in-out;
    }
    th&-column,
    td&-column {
      &-left {
        text-align: left;
      }
      &-center {
        text-align: center;
      }
      &-right {
        text-align: right;
      }
    }
    & table {
      //width: 100%;
      table-layout: fixed;
    } // &-border{
    th,
    td {
      border-right: 1px solid @border-color-base;
    } // }
    &-cell {
      padding-left: 18px;
      padding-right: 18px;
      overflow: hidden;
      text-overflow: ellipsis;
      white-space: normal;
      word-break: break-all;
      box-sizing: border-box;
      &-ellipsis {
        word-break: keep-all;
        white-space: nowrap;
        overflow: hidden;
        text-overflow: ellipsis;
      }
      &-tooltip {
        width: 100%;
        &-content {
          display: block;
          overflow: hidden;
          text-overflow: ellipsis;
          white-space: nowrap;
        }
      }
      &-with-expand {
        height: 47px;
        line-height: 47px;
        padding: 0;
        text-align: center;
      }
      &-expand {
        cursor: pointer; // transition: transform @transition-time @ease-in-out;
        i {
          // font-size: @font-size-base;
        }
        &-expanded {
          transform: rotate(90deg);
        }
      }
      &-sort {
        cursor: pointer;
        user-select: none;
      } // #3159
      &-with-selection {
        // .@{checkbox-prefix-cls}-wrapper{
        //     margin-right: 0;
        // }
      }
    }
    &-hidden {
      visibility: hidden;
    }
    th &-cell {
      display: inline-block; //position: relative;
      word-wrap: normal;
      vertical-align: middle;
    }
    td&-expanded-cell {
      padding: 20px 50px; // background: @table-thead-bg;
    }
    &-stripe &-body,
    &-stripe &-fixed-body {
      tr:nth-child(2n) {
        td {
          // background-color: @table-td-stripe-bg;
        }
      } // #1380
      tr.@{table-prefix-cls}-row-hover {
        td {
          // background-color: @table-td-hover-bg;
        }
      }
    }
    tr&-row-hover {
      td {
        // background-color: @table-td-hover-bg;
      }
    }
    &-large {
      // font-size: @font-size-base;
      th {
        height: 48px;
      }
      td {
        height: 60px;
      }
      &-title,
      &-footer {
        height: 60px;
        line-height: 60px;
      }
      .@{table-prefix-cls}-cell-with-expand {
        height: 59px;
        line-height: 59px;
        i {
          // font-size: @font-size-base+2;
        }
      }
    }
    &-small {
      th {
        height: 32px;
      }
      td {
        height: 40px;
      }
      &-title,
      &-footer {
        height: 40px;
        line-height: 40px;
      }
      .@{table-prefix-cls}-cell-with-expand {
        height: 39px;
        line-height: 39px;
      }
    }
    &-row-highlight,
    tr&-row-highlight&-row-hover,
    &-stripe &-body tr&-row-highlight:nth-child(2n),
    &-stripe &-fixed-body tr&-row-highlight:nth-child(2n) {
      td {
        // background-color: @table-td-highlight-bg;
      }
    }
    &-fixed,
    &-fixed-right {
      position: absolute;
      top: 0;
      left: 0;
      box-shadow: 2px 0 6px -2px rgba(0, 0, 0, 0.2);
      &::before {
        content: '';
        width: 100%;
        height: 1px; // background-color: @border-color-base;
        position: absolute;
        left: 0;
        bottom: 0;
        z-index: 4;
      }
    }
    &-fixed-right {
      top: 0;
      left: auto;
      right: 0;
      box-shadow: -2px 0 6px -2px rgba(0, 0, 0, 0.2);
    }
    &-fixed-right-header {
      position: absolute;
      top: -1px;
      right: 0; // background-color: @table-thead-bg;
      // border-top: 1px solid @border-color-base;
      // border-bottom: 1px solid @border-color-split;
    }
    &-fixed-header {
      overflow: hidden; // 在 #1387 里，添加了下面的代码，但是在 #5174 会出现新的问题。
      // 但是，在新版本里，注释掉后，#1387 的问题并没有再复现，所以注释掉
      //&-with-empty{
      //    .@{table-prefix-cls}-hidden{
      //        .@{table-prefix-cls}-sort{
      //            display: none;
      //        }
      //        .@{table-prefix-cls}-cell span{
      //            display: none;
      //        }
      //    }
      //}
    }
    &-fixed-body {
      overflow: hidden;
      position: relative;
      z-index: 3;
    }
    &-fixed-shadow {
      width: 1px;
      height: 100%;
      position: absolute;
      top: 0;
      right: 0; // box-shadow: @shadow-right;
      overflow: hidden;
      z-index: 1;
    }
    &-sort {
      // .sortable();
    }
    &-filter {
      display: inline-block;
      cursor: pointer;
      position: relative; //top: 1px;
      i {
        // color: @btn-disable-color;
        // transition: color @transition-time @ease-in-out;
        &:hover {
          color: inherit;
        }
        &.on {
          // color: @primary-color;
        }
      }
      &-list {
        padding: 8px 0 0;
        &-item {
          padding: 0 12px 8px;
          .ivu-checkbox-wrapper+.ivu-checkbox-wrapper {
            margin: 0;
          }
          label {
            display: block;
            &>span {
              margin-right: 4px;
            }
          }
        }
        ul {
          padding-bottom: 8px;
        } // .select-item(@table-prefix-cls, @table-select-item-prefix-cls);
      }
      &-footer {
        padding: 4px; // border-top: 1px solid @border-color-split;
        overflow: hidden;
        button:first-child {
          float: left;
        }
        button:last-child {
          float: right;
        }
      }
    }
    &-tip {
      table {
        width: 100%;
        td {
          text-align: center;
        }
      }
    }
    &-expanded-hidden {
      visibility: hidden;
    }
  }
</style>
