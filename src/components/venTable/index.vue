<template lang="pug">
  .ven-table(ref="container" v-if="data")
    .ven-table__head
      .ven-table__title 
        span(v-if="!search_field") {{ title }}
        span(v-else) Поиск {{ search_field }}
      .ven-table__search
        input.ven-table__search-field(type="text" placeholder="Поиск:" maxlength="150" v-model="search_field")
        svg.ven-table__search-icon(fill='none' stroke='#000' stroke-linecap='round' stroke-linejoin='round' stroke-width='2' viewbox='0 0 24 24' xmlns='http://www.w3.org/2000/svg')
          circle(cx='10.5' cy='10.5' r='7.5')
          line(x1='21' x2='15.8' y1='21' y2='15.8')
    .ven-table__inner(ref="inner")
      table.ven-table__table
        thead.ven-table__header
          tr
            th.ven-table__th-open(v-if="response_cols && response_cols.length")
            th.ven-table__th(v-for="(th, ind) in headers" :data-ind="ind" :data-responsive="isResponsive(ind)")
              span {{ th }}
              button.ven-table__sort(@click="tableSort(ind)" :class="{'active': sort === ind, 'reverse': sort === ind && sort_type === 'ba'}") 
                svg(version='1.1' viewBox='0 0 24 24' xml:space='preserve' xmlns='http://www.w3.org/2000/svg' xmlns:xlink='http://www.w3.org/1999/xlink')
                  path(d='M13,13V8h-2.9c0,0-0.7,0.1-0.9-0.4s0.3-1,0.3-1L16.1,0l6.5,6.5c0,0,0.7,0.5,0.5,1.1s-1,0.4-1,0.4H19v5c0,0.6-0.3,1-0.9,1   h-4C13.6,14,13,13.6,13,13z M15,12h2V6h2.1l-2.9-3l-3.1,3H15V12z')
                  path(d='M1.1,16.4c0.2-0.5,1-0.4,1-0.4H5v-5c0-0.6,0.5-1,1-1h4c0.6,0,1,0.4,1,1v5h3.1c0,0,0.7-0.1,0.9,0.4s-0.3,1-0.3,1L8,24   l-6.5-6.5C1.6,17.5,0.9,17,1.1,16.4z M8,21l3.1-3H9v-6H7v6H5.1L8,21z')
              
        tbody(v-if="computed_rows")
          template(v-for="(row, ind) in computed_rows")
            tr.ven-table__row(:class="{'odd': ind%2 > 0}")
              td.ven-table__td-open(v-if="response_cols && response_cols.length")
                button.ven-table__open-btn(@click="open(ind)") 
                  svg(v-if="isOpen(ind)" fill="none" stroke="currentColor" stroke-linecap="round" stroke-linejoin="round" stroke-width="2" viewBox="0 0 24 24" xmlns="http://www.w3.org/2000/svg")
                    line(x1="5" x2="19" y1="12" y2="12")
                  svg(v-else viewBox="0 0 32 32" xmlns="http://www.w3.org/2000/svg")
                    line(fill="none" stroke="currentColor" stroke-width="2px" class="cls-1" x1="16" x2="16" y1="7" y2="25")
                    line(fill="none" stroke="currentColor" stroke-width="2px"  x1="7" x2="25" y1="16" y2="16")
              td.ven-table__td(v-for="(col, ind) in row" v-html="col.data" :class="{[col.cls]: col.cls, 'textwrap': aviable_cols < 2}" :data-ind="ind" :data-responsive="isResponsive(ind)")
            tr.ven-table__hidden(:class="{'odd': ind%2 > 0}" v-if="isOpen(ind) && response_cols && response_cols.length")
              td.ven-table__td-open(v-if="response_cols && response_cols.length")
              td.ven-table__td-resp(:colspan="response_cols.length")
                .ven-table__td-resp-val
                  .ven-table__td-resp-cell(v-for="(col, ind) in row" :class="col.cls" :data-ind="ind" :data-responsive="isResponsive(ind)")
                    span.ven-table__td-caption {{ headers[ind] }}
                    span.ven-table__td-value(v-html="col.data")
</template>

<script>

export default {
  data() {
    return {
      container: 0,
      viewport: 0,
      aviable_cols: [],
      response_cols: [],
      opened: [],
      sort: null,
      sort_type: 'ab',
      search_field: null
    }
  },
  props: {
    data: {
      type: Object
    },
    search: {
      type: Boolean,
      default: false
    },
    title: {
      type: String
    }
  },
  computed: {
    headers() {
      return this.data && this.data.headers ? this.data.headers : [];
    },
    rows() {
      return this.data && this.data.rows ? this.data.rows : [];
    },
    computed_rows() {
      return this.searchData && this.searchData.length ? this.searchData : this.rows
    },
    searchData() {
      const result = [];
      this.rows.forEach(tr => {
        tr.forEach(td => {
          if(this.search_field && td.data.toString().toLowerCase().replaceAll(' ', '').indexOf(this.search_field.toLowerCase().replaceAll(' ', '')) !== -1 && !result.find(el => el === tr)) result.push(tr);
        })
      });
      return result;
    }
  },
  mounted() {
    this.init();
  },
  destroyed() {
    this.destroy();
  },
  watch: {
    data() {
      this.init();
    }
  },
  methods: {
    tableSort(ind) {
      this.opened = [];
      const rows = this.data.rows;
      this.sort = ind;
      this.sort_type = this.sort_type === 'ab' ? 'ba' : 'ab';

      let type = "string";
      
      if(!isNaN(new Date(rows[0][ind].data).getTime())) type = "date";
      if (rows[0][ind].toString().match(/^[0-9]+$/) != null) type = "number";


      if(type === "number")
        rows.sort((a, b) => {
          return this.sort_type === 'ab' ? a[ind].data - b[ind].data : b[ind].data - a[ind].data;
        });


      if(type === "date")
        rows.sort((a, b) => {
          let da = new Date(a[ind].data),
              db = new Date(b[ind].data);
          return this.sort_type === 'ab' ? da - db : db - da;
        });


      if(type === "string") 
        rows.sort((a, b) => {
          let fa = a[ind].data.toLowerCase(),
              fb = b[ind].data.toLowerCase();
          if (fa < fb) return this.sort_type === 'ab' ? -1 : 1;
          if (fa > fb) return this.sort_type === 'ab' ? 1 : -1;
          return 0;
        });

    },

    isResponsive(ind) {
      return this.response_cols.includes(ind);
    },
    init() {
      if(!this.data) return;
      this.$nextTick(() => {
        this.getTableSize();
      });
      window.addEventListener("resize", this.getTableSize);
    },
    destroy() {
      window.removeEventListener("resize", this.getTableSize);
    },
    open(ind) {
      if(this.opened.includes(ind)) {
        this.opened = this.opened.filter(el => el !== ind)
      } else {
        this.opened.push(ind);
      }
    },
    isOpen(ind) {
      return this.opened.includes(ind);
    },
    indexCols() {
      this.aviable_cols = [];
      this.response_cols = [];
      for(let i = 0; i < this.rows[0].length; i++) {
        this.aviable_cols.push(i);
      }
    },
    getTableSize() {
      this.indexCols();
      
      this.$nextTick(() => {
        this.container = this.$refs.container.scrollWidth;
        this.viewport = this.$refs.inner.clientWidth;

        if(this.container > this.viewport) {
          this.tableResponsiveEmmiter();
        }
      })
    },

    transferCol() {
      if(!this.aviable_cols || this.aviable_cols.length < 2) return;
      const transfer = this.aviable_cols[this.aviable_cols.length - 1];
      this.aviable_cols = this.aviable_cols.filter(el => el !== transfer);
      this.response_cols.push(transfer);
    },

    tableResponsiveEmmiter() {
      this.transferCol();

      this.$nextTick(() => {
        const container = this.$refs.container.scrollWidth;
        const viewport = this.$refs.inner.clientWidth;
        
        if(container > viewport && this.aviable_cols && this.aviable_cols.length > 1) {
          this.tableResponsiveEmmiter()
        }
      })
      
    }
  }

}
</script>

<style lang="stylus">

:root
  --tdp: 12px 10px 10px 10px

.ven-table
  width 100%
  overflow: auto
  box-sizing border-box
  text-align left
  font-size 14px
  *
    box-sizing border-box
  &__search
    position relative
    width 300px
    flex-shrink 0
    margin-left auto
    input
      border 1px solid #bdc3c7
      height 40px
      width 100%
      padding-left 40px
      outline: none !important
      font-family inherit !important
      border-radius 8px
      &:focus
        border-color #3498db
        & + svg
          stroke #3498db!important
    svg
      position absolute
      left 10px
      top 50%
      pointer-events: none
      user-select: none
      width 24px
      height 24px
      margin-top -12px
      stroke #bdc3c7
  &__head
    display flex
    justify-content space-between
    align-items center
  &__title
    margin-bottom 20px
    font-size 24px
    font-weight 700
    margin-right 20px
  &__search
    margin-bottom 20px
  &__td-resp
    padding var(--tdp)
    padding-bottom 20px
    font-size 0.9em
  &__sort
    width 20px
    height 20px
    padding 0
    border 0
    display inline-block
    vertical-align: middle
    margin-left 5px
    cursor pointer
    opacity 0.25
    svg
      width 90%
      height 90%
    &:hover
      opacity 0.75
      path
        fill #3498E2
    &.active
      opacity 1
      path
        fill #000
        &:nth-child(2)
          opacity 0.5
    &.reverse
      path
        opacity 0.5
        &:nth-child(2)
          opacity 1
  &__open-btn
    border-radius 50%
    background-color #3498db
    color #fff
    width 24px
    height 24px
    padding 0
    margin 0
    display inline-flex
    cursor pointer
    font-size 16px
    align-items center
    justify-content center
    border 0
    outline: none !important
    svg
      width 70%
      height 70%
    &:hover
      background-color #2980b9
      color #fff
  &__td-value
    white-space: normal !important
    word-break: break-all
    margin-bottom 5px
  &__table
    min-width 100%
    border-collapse: collapse
    border-radius 8px
    overflow hidden
  &__td-open, &__th-open
    width 1px
    padding-left 10px
    padding-right 0px
    text-align center
  &__header
    background-color #ecf0f1
    border-bottom 1px solid #bdc3c7
  &__th
    padding var(--tdp)
    text-align left
    font-size 1.1em
    vertical-align: middle
    white-space: nowrap
    font-weight 600
    &[data-responsive="true"]
      display none

  &__td-resp-cell
    display none
    margin-bottom 10px
    flex-wrap wrap
    white-space: normal
    &[data-responsive="true"]
      display flex
    &:last-child
      margin-bottom 0
  &__td-caption
    display block
    width 120px
    padding-right 10px
    color #999
    margin-bottom 5px
  &__td
    padding var(--tdp)
    color #111
    width 100px
    font-size 0.9em
    &[data-responsive="true"]
      display none
    &.nowrap
      white-space: nowrap
    &.textwrap
      white-space: normal !important
      word-break: break-all
      width 100%
  &__row
    &.odd
      background-color #f7f7f7

@media (max-width 900px)
  .ven-table
    &__head
      flex-direction column
    &__search
      width 100%
    &__title
      width 100%
      margin-right 0
@media (max-width 600px)
  .ven-table
    &__td-resp-cell
      border-bottom 1px solid #eee
    &__td-caption
      width 100%

</style>