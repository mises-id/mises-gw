<template>
  <div class="content content-top">
    <Card title="Address Detail" :loading="loading">
      <template #content>
        <MisesList :data="block" title="Overview" />
        <div class="">
          <p class="list-title">Transactions</p>
          <a-table
            :dataSource="txns"
            :columns="columns"
            :loading="listloading"
            @change="handleChange"
            :bordered="false"
            rowKey="height"
            :customRow="handleClickRow"
            :pagination="{
              total: total,
              current: pages,
              pageSize: page_size,
              showQuickJumper: true,
            }"
            :scroll="{ x: 1000 }"
            :rowClassName="
              (_record, index) =>
                index % 2 === 1 ? 'table-striped' : undefined
            "
          />
        </div>
      </template>
    </Card>
  </div>
</template>

<script lang="ts">
import Card from "../components/MisesCard";
import MisesList from "../components/MisesList";
import { getAddress, getTxs } from "../api/serve";
import { dataItem } from "../components/MisesList/MisesList.vue";
import { formatTime, shortenAddress } from "../utils/plugins";
import { Table } from "ant-design-vue";
import dayjs from "dayjs";
import { h } from "@vue/runtime-core";
export default {
  components: {
    Card,
    MisesList,
    ATable: Table,
  },
  data() {
    return {
      block: [] as dataItem[],
      txns: [],
      loading: false,
      pages: 1,
      page_size: 10,
      total: 0,
      username: "",
      columns: [
        {
          title: "Txn Hash",
          dataIndex: "hash",
          width: "25%",
          ellipsis: true,
        },
        {
          title: "Block",
          dataIndex: "height",
          width: "8%",
        },
        {
          title: "Time",
          dataIndex: "block_time",
          width: "15%",
        },
        {
          title: "From",
          dataIndex: "from_address",
          customRender: ({ text }, record) => {
            const isMe = text === this.$route.params.misesid;
            return h("div", {}, [
              h("span", {}, isMe ? this.username : shortenAddress(text)),
            ]);
          },
        },
        {
          title: "",
          width: "50px",
          customRender: ({ text }, record) => {
            const message = this.messageType(text.tx_result.body.messages);
            if (message === "Reinvest") {
              return "";
            }
            const isMe = text.from_address === this.$route.params.misesid;
            return h(
              "span",
              { class: !isMe ? "in" : "out" },
              !isMe ? "In" : "Out"
            );
          },
        },
        {
          title: "To",
          dataIndex: "to_address",
          customRender: ({ text, record }) => {
            const message = this.messageType(record.tx_result.body.messages);
            if (message === "Reinvest") {
              return h("div", {}, [
                h("span", {}, shortenAddress(record.from_address)),
              ]);
            }
            
            const isMe = text === this.$route.params.misesid;
            return h("div", {}, [
              h("span", {}, isMe ? this.username : shortenAddress(text)),
            ]);
          },
        },
        {
          title: "Message Type",
          dataIndex: "tx_result",
          customRender: ({ text: item }, record) => {
            const { messages } = item.body;
            return h("span", {}, this.messageType(messages));
          },
        },
        {
          title: "Value",
          dataIndex: "value_mis",
          width: "10%",
        },
        {
          title: "Gas Fee",
          dataIndex: "gas_fee",
          width: "10%",
        },
      ],
    };
  },
  async created() {
    this.initPage()
  },
  watch:{
    '$route.params.misesid'(val){
      this.initPage()
    }
  },
  methods: {
    getTxsList() {
      this.listloading = true;
      getTxs({
        page_num: this.pages,
        page_size: this.page_size,
        address: this.$route.params.misesid,
      })
        .then((res) => {
          this.txns = res.data.map((val) => {
            return {
              block_time: formatTime(val.block_time),
              from_address: val.tx_msg?.from_address ?? "-",
              to_address: val.tx_msg?.to_address ?? "-",
              value_mis: `${val.value_mis} MIS`,
              gas_fee: `${val.gas_fee} MIS`,
              hash: val.hash,
              height: val.height,
              msg_type: val.msg_type,
              tx_result: val.tx_result.tx,
            };
          });
          this.total = res.pagination.total_records;
          this.listloading = false;
        })
        .catch(() => {
          this.listloading = false;
        });
    },
    handleClickRow(record) {
      return {
        onClick: () => {
          this.$router.push(`/tx/${record.hash}`);
        },
      };
    },
    handleChange(val) {
      this.page_size = val.pageSize;
      this.pages = val.current;
      this.getTxsList();
    },
    messageType(messages) {
      let message = "";
      messages.forEach((element) => {
        if (!message) {
          message = element["@type"];
        } else {
          if (
            message ===
              "/cosmos.distribution.v1beta1.MsgWithdrawDelegatorReward" &&
            element["@type"] === "/cosmos.staking.v1beta1.MsgDelegate"
          ) {
            message = "/cosmos.distribution.v1beta1.MsgReinvest";
          }
        }
      });
      // const text = messages[messages.length - 1]["@type"];
      const typeParams = {
        "/cosmos.bank.v1beta1.MsgSend": "Transfer",
        "/cosmos.staking.v1beta1.MsgDelegate": "Delegate",
        "/cosmos.staking.v1beta1.MsgUndelegate": "Undelegate",
        "/cosmos.staking.v1beta1.MsgBeginRedelegate": "Redelegate",
        "/cosmos.distribution.v1beta1.MsgWithdrawDelegatorReward":
          "Withdraw Rewards",
        "/cosmos.distribution.v1beta1.MsgReinvest": "Reinvest",
      };
      return typeParams[message];
    },
    async initPage(){
      this.loading = true;
      try {
        const res = await getAddress({ misesid: this.$route.params.misesid });
        const username = res.name_tag;
        const that = this;
        this.block = [
          {
            title: "Address",
            value: res.misesid,
          },
          {
            title: "Name Tag",
            value: username,
          },
          {
            title: "Register Time",
            value: res.user_ext.register_time
              ? dayjs(res.user_ext.register_time).format("M/DD/YYYY")
              : "-",
          },
          {
            title: "Balance",
            value: `${res.user_ext.quantity}MIS`,
          },
          {
            title: "Social Relationships",
            value: res.user_ext.social_relationships,
          },
        ] as dataItem[];
        if (res.user_ext.validator) {
          this.block.unshift({
            title: "Validator Address",
            value: res.user_ext.validator.operator_address,
            render({ value }) {
              return h(
                "span",
                {
                  className: "active",
                  onClick: () => {
                    that.$router.push({
                      path: `/validators/${res.user_ext.validator.operator_address}`,
                    });
                  },
                },
                value
              );
            },
          });
        }
        this.username = username;
        this.getTxsList();
      } finally {
        this.loading = false;
      }
    }
  },
};
</script>
<style lang="scss">
.list-title {
  font-size: 18px;
  color: #666666;
  font-family: "hlt-75";
  margin-top: 40px;
  margin-bottom: 25px;
}
.out {
  color: #ff9600;
  background: #ffeacc;
  display: inline-block;
  margin-left: 5px;
  border-radius: 3px;
  width: 30px;
  height: 18px;
  text-align: center;
  font-size: 13px;
  font-family: "hlt-45";
  line-height: 22px;
}
.in {
  color: #50c68d;
  background: #dcf4e8;
  display: inline-block;
  margin-left: 5px;
  width: 30px;
  height: 18px;
  text-align: center;
  font-size: 13px;
  font-family: "hlt-45";
  line-height: 22px;
}
</style>