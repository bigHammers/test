<template>
  <Table :columns="columns11" :data="data10" border height="500" />
</template>

<script>
  // @ is an alias to /src
  import Table from "@/components/table/table.vue";
  export default {
    name: "about",
    data() {
      return {
        columns11: [{
            title: "时间趋势",
            key: "timeTrend",
            align: "center",
            width: 200,
            fixed: "left",
            filters: [{
                label: "Joe",
                value: 1
              },
              {
                label: "John",
                value: 2
              }
            ],
          },
          {
            title: "现金类",
            align: "center",
            children: [{
                title: "微信支付",
                key: "age",
                align: "center",
                width: 200,
                sortable: true
              },
              {
                title: "支付宝支付",
                align: "center",
                key:"pay",
                width: 200,
                sortable: true
              },
              {
                title: "大众点评",
                align: "center",
                key:"people",
                width: 200,
                sortable: true
              },
              {
                title: "占比",
                align: "center",
                key:"perf",
                width: 200,
                sortable: true
              },
              {
                title: "商城赠送券",
                align: "center",
                key:"shop",
                width: 200,
                sortable: true
              }
            ]
          },
        ],
        data10: []
      };
    },
    created() {
       const data = [];
            for (let i = 0; i < 20; i++) {
                data.push({
                    key: i,
                    timeTrend: 'John Brown',
                    age: i + 1,
                    pay: 'Lake Park',
                    people: 'C',
                    perf: 2035,
                    shop: 'Lake Street 42',
                });
            }
        this.data10 = data;
    },
    components: {
      Table
    }
  };
</script>
