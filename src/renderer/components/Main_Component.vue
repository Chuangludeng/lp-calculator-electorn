<template>
  <div>
    <el-header>
      <p>国服EVE LP计算器</p>
    </el-header>
    <el-main>
      军团：
      <el-select
        v-model="selectedCorporation"
        placeholder="请选择军团"
        v-on:change="getLPStoreDataWithCookie"
        filterable
      >
        <el-option
          v-for="item in corporationData"
          :key="item.name"
          :label="item.name"
          :value="item.id"
        ></el-option>
      </el-select>&nbsp;&nbsp;
      <el-input v-model="search" style="width: 250px" placeholder="输入物品名称搜索" />
      &nbsp;&nbsp;
      <el-switch
        active-text="使用缓存"
        v-model="useCookie"
        active-color="#13ce66"
        inactive-color="#ff4949"
      ></el-switch>
      <el-switch
        active-text="显示蓝图"
        v-model="showBlueprint"
        active-color="#13ce66"
        inactive-color="#ff4949"
      ></el-switch>
      &nbsp;&nbsp;
      获取数据进度{{curUpdateIndex}}/{{tableData.length}}
      <el-button @click="dialogCartVisible = true">打开购物车</el-button>

      <el-dialog title="购物车" :visible.sync="dialogCartVisible">
        <el-table :data="cart">
          <el-table-column prop="name" label="名称" sortable width="250"></el-table-column>
          <el-table-column label="兑换数量" width="250">
            <template slot-scope="scope">
              <el-input-number size="small" v-model="scope.row.getNumber"></el-input-number>
            </template>
          </el-table-column>
          <el-table-column label="数量">
            <template slot-scope="scope">
              <span style="margin-left: 10px">{{(getTotalNubmer(scope.row)).toLocaleString()}}</span>
            </template>
          </el-table-column>
          <el-table-column label="LP消耗">
            <template slot-scope="scope">
              <span
                style="margin-left: 10px"
              >{{(scope.row.lp_cost * scope.row.getNumber).toLocaleString()}}</span>
            </template>
          </el-table-column>
          <el-table-column label="吉他卖单收入">
            <template slot-scope="scope">
              <span
                style="margin-left: 10px"
              >{{(scope.row.sell * scope.row.getNumber * scope.row.quantity).toLocaleString()}}</span>
            </template>
          </el-table-column>
          <el-table-column label="吉他买单收入">
            <template slot-scope="scope">
              <span
                style="margin-left: 10px"
              >{{(scope.row.buy * scope.row.getNumber * scope.row.quantity).toLocaleString()}}</span>
            </template>
          </el-table-column>
        </el-table>物品需求统计
        <el-table :data="cartRequest" style="width: 100%">
          <el-table-column prop="name" label="名称" width="200" sortable></el-table-column>
          <el-table-column prop="quantity" label="数量" width="80" sortable>
            <template slot-scope="rowNumber">
              <span style="margin-left: 10px">{{getTotalRequestNubmer(rowNumber)}}</span>
            </template>
          </el-table-column>
          <el-table-column label="现有数量" width="300" sortable>
            <template slot-scope="rowNumber">
              <el-input-number v-model="rowNumber.row.now_number" :min="0"></el-input-number>
            </template>
          </el-table-column>
          <el-table-column label="还需数量" width="120" sortable>
            <template slot-scope="rowNumber">
              <span
                style="margin-left: 10px"
              >{{getTotalRequestNubmer(rowNumber) - rowNumber.row.now_number}}</span>
            </template>
          </el-table-column>
        </el-table>
      </el-dialog>

      <el-button @click="help">帮助</el-button>

      <el-table :data="tables" style="width: 100%" v-on:expand-change="getItemOrder" stripe>
        <el-table-column type="expand">
          <template slot-scope="scope">
            兑换计算
            <el-input-number v-model="getNumber" :min="1" label="兑换计算"></el-input-number>
            <span style="margin-left: 10px">数量：{{scope.row.quantity * getNumber}}</span>
            <span
              style="margin-left: 10px"
            >LP消耗：{{(scope.row.lp_cost * getNumber).toLocaleString()}}</span>
            <span
              style="margin-left: 10px"
            >吉他卖单收入：{{(scope.row.sell * getNumber * scope.row.quantity).toLocaleString()}}</span>
            <span
              style="margin-left: 10px"
            >吉他买单收入：{{(scope.row.buy * getNumber * scope.row.quantity).toLocaleString()}}</span>
            <el-table :data="scope.row.required_items" style="width: 100%">
              <el-table-column prop="name" label="名称" width="200" sortable></el-table-column>
              <el-table-column prop="quantity" label="数量" width="80" sortable>
                <template slot-scope="rowNumber">
                  <span style="margin-left: 10px">{{rowNumber.row.quantity * getNumber}}</span>
                </template>
              </el-table-column>
              <el-table-column label="现有数量" width="300" sortable>
                <template slot-scope="rowNumber">
                  <el-input-number v-model="rowNumber.row.now_number" :min="0"></el-input-number>
                </template>
              </el-table-column>
              <el-table-column label="还需数量" width="120" sortable>
                <template slot-scope="rowNumber">
                  <span
                    style="margin-left: 10px"
                  >{{rowNumber.row.quantity * getNumber - rowNumber.row.now_number}}</span>
                </template>
              </el-table-column>
            </el-table>
            <br />卖单
            <el-table
              :data="sellData"
              style="width: 100%"
              stripe
              :default-sort="{prop: 'price', order: 'ascending'}"
            >
              <el-table-column prop="quantity" label="数量" width="80" sortable></el-table-column>
              <el-table-column prop="price" label="价格" :formatter="formatterNumber" sortable />
              <el-table-column prop="orderDate" label="订单日期" />
              <el-table-column prop="position" label="位置" />
            </el-table>
            <br />买单
            <el-table
              :data="buyData"
              style="width: 100%"
              stripe
              :default-sort="{prop: 'price', order: 'descending'}"
            >
              <el-table-column prop="quantity" label="数量" width="80" sortable></el-table-column>
              <el-table-column prop="price" label="价格" :formatter="formatterNumber" sortable />
              <el-table-column prop="orderDate" label="订单日期" />
              <el-table-column prop="position" label="位置" />
            </el-table>
          </template>
        </el-table-column>
        <el-table-column prop="name" label="名称" width="250">

        </el-table-column>
        <el-table-column prop="lp_cost" label="忠诚点数" sortable :formatter="formatterNumber"></el-table-column>
        <el-table-column prop="quantity" label="数量" width="80" sortable></el-table-column>
        <el-table-column prop="isk_cost" :formatter="formatterNumber" sortable label="兑换所需星币"></el-table-column>
        <el-table-column label="兑换所需物品" width="200">
          <template slot-scope="scope">
            <p
              v-for="item in scope.row.required_items"
              :key="item.name"
            >{{ item.name + "：" + item.quantity}}</p>
          </template>
        </el-table-column>

        <el-table-column prop="sell" label="吉他出售(单个)" :formatter="formatterNumber" sortable />
        <el-table-column prop="buy" label="吉他收购(单个)" :formatter="formatterNumber" sortable />
        <el-table-column
          prop="lp_rate_buy"
          label="吉他收购比例"
          width="150"
          :formatter="formatterNumber"
          sortable
        />
        <el-table-column
          prop="lp_rate_sell"
          label="吉他出售比例"
          width="150"
          :formatter="formatterNumber"
          sortable
        />
        <el-table-column
          prop="lp_rate_buy_all"
          label="所需物品以吉他卖价购买,吉他收购比例"
          width="190"
          sortable
          :formatter="formatterNumber"
        />
        <el-table-column
          prop="lp_rate_sell_all"
          label="所需物品以吉他卖价购买,吉他出售比例"
          width="190"
          sortable
          :formatter="formatterNumber"
        />
        <el-table-column label="操作">
          <template slot-scope="scope">
            <el-button size="mini" @click="handleAdd(scope.$index, scope.row)">加入购物车</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-main>
  </div>
</template>

<script>
import $ from "jquery";

import fs from "fs";
import path from "path";

export default {
  name: "Main_Component",
  props: {},

  data() {
    return {
      tableData: [],
      corporationData: null,
      itemDescription: null,
      blueprintProduct: null,
      curUpdateIndex: 0,
      loading: false,
      useCookie: false,
      showBlueprint: false,
      sellData: [],
      buyData: [],
      getNumber: 1,
      cart: [],
      cartRequest: [],
      dialogCartVisible: false,
      search: "",
    };
  },

  computed:{
    tables:function(){
      const search = this.search;
      if (search) {
        return this.tableData.filter(item => {
            return item.name.indexOf(search) > -1;
        });
      }
      return this.tableData;
    }
  },


  methods: {

    help: function () {
      this.$alert(
        "启动后，在上面选择军团，然后边上有一个数字在变化，这是在从国服市场中心取得数据。<b>等数字走完后，需要点击一下数据列边上的箭头进行排序，数据会显示出来</b>。 <b>点击每个物品前面的箭头会显示该物品的订单情况，以及一个简单的数量计算器</b>。<br>显示蓝图 表示是否以蓝图制造的物品来计算lp，关闭的话，蓝图的价值是0。<br>使用缓存 从市场取得的数据会缓存到本地，不会每次都到国服市场中心取数据，这个开关是控制是不是强制每次都重取。",
        "帮助",
        {
          confirmButtonText: "确定",
          dangerouslyUseHTMLString: true,
        }
      );
    },

    getTotalRequestNubmer: function (requestItem) {
      let ret = 0;
      for (let i = 0; i < requestItem.row.requests.length; i++) {
        ret +=
          requestItem.row.requests[i].quantity *
          requestItem.row.requests[i].main_item.getNumber;
      }
      return ret;
    },

    getTotalNubmer: function (row) {
      return row.getNumber * row.quantity;
    },

    formatterNumber: function (row, column, cellValue) {
      if (typeof cellValue == "undefined") return "";
      else return cellValue.toLocaleString();
    },

    handleAdd: function (index, row) {
      this.cart.push(row);

      for (var i = 0; i < row.required_items.length; i++) {
        var found = null;
        for (var j = 0; j < this.cartRequest.length; j++) {
          if (this.cartRequest[j].type_id == row.required_items[i].type_id) {
            found = this.cartRequest[j];
            break;
          }
        }

        let item = {
          main_item: row,
          quantity: row.required_items[i].quantity,
        };

        this.$set(item, "getNumber", row.getNumber);

        if (found != null) {
          found.requests.push(item);
        } else {
          found = {
            name: row.required_items[i].name,
            type_id: row.required_items[i].type_id,
            requests: [item],
          };

          this.$set(found, "now_number", 0);

          this.cartRequest.push(found);
        }
      }
    },

    getItemOrder: function (row) {
      var vm = this;

      var product_id = row.type_id;
      var product = vm.blueprintProduct.get(row.type_id);
      if (product != undefined) {
        product_id = product.product;
      }

      $.ajax({
        type: "GET",
        url:
          "https://www.ceve-market.org/api/quicklook?regionlimit=10000002&typeid=" +
          product_id,
        dataType: "xml",
        success: function (xml) {
          vm.sellData.length = 0;
          vm.buyData.length = 0;

          $(xml)
            .find("sell_orders")
            .find("order")
            .each(function () {
              var order = {
                quantity: Number($(this).find("vol_remain").text()),
                price: Number($(this).find("price").text()),
                orderDate: $(this).find("reported_time").text(),
                position: $(this).find("station_name").text(),
              };
              vm.sellData.push(order);
            });
          $(xml)
            .find("buy_orders")
            .find("order")
            .each(function () {
              var order = {
                quantity: Number($(this).find("vol_remain").text()),
                price: Number($(this).find("price").text()),
                orderDate: $(this).find("reported_time").text(),
                position: $(this).find("station_name").text(),
              };
              vm.buyData.push(order);
            });
        },
      });
    },

    getReqItemData: function (item) {
      var item_promise = $.getJSON(
        "https://www.ceve-market.org/api/market/region/10000002/type/" +
          item.type_id +
          ".json",
        function (mm_data) {
          item.buy = mm_data.buy.max;
          item.sell = mm_data.sell.min;

          window.localStorage.setItem(
            item.type_id,
            JSON.stringify({ buy: mm_data.buy.max, sell: mm_data.sell.min })
          );
        }
      );
      return item_promise;
    },

    getLPStoreData: function (corporationid, useCookie) {
      var vm = this;
      vm.curUpdateIndex = 0;
      vm.loading = true;
      fs.readFile(
        path.join(__static, "/LPStore/" + corporationid + "/loyalty.json"),
        "utf-8",
        function (err, data) {
          var d = JSON.parse(data);

          d.forEach((element) => {
            var promiseList = [];

            var item_main = vm.itemDescription.get(element.type_id);
            element.name = item_main.name;
            element.description = item_main.description;
            element.getNumber = 1;

            var product_id = element.type_id;
            if (vm.showBlueprint) {
              var product = vm.blueprintProduct.get(element.type_id);
              if (product != undefined) {
                product_id = product.product;
              }
            }

            var isUseCookie = useCookie;

            if (window.localStorage.getItem(product_id) == null) {
              isUseCookie = false;
            }

            if (!isUseCookie) {
              var main_promise = $.getJSON(
                "https://www.ceve-market.org/api/market/region/10000002/type/" +
                  product_id +
                  ".json",
                function (m_data) {
                  element.buy = m_data.buy.max;
                  element.sell = m_data.sell.min;

                  window.localStorage.setItem(
                    element.type_id,
                    JSON.stringify({
                      buy: m_data.buy.max,
                      sell: m_data.sell.min,
                    }),
                    { expires: 7 }
                  );
                }
              );
              promiseList.push(main_promise);
            } else {
              var price = JSON.parse(localStorage.getItem(product_id));
              element.buy = price.buy;
              element.sell = price.sell;
            }

            for (var i = 0; i < element.required_items.length; i++) {
              var item = element.required_items[i];
              var item_request = vm.itemDescription.get(item.type_id);
              element.required_items[i].name = item_request.name;
              element.required_items[i].description = item_request.description;
              element.required_items[i].now_number = 0;

              var isUseCookie_req = useCookie;

              if (window.localStorage.getItem(item.type_id) == null) {
                isUseCookie_req = false;
              }

              if (!isUseCookie_req) {
                promiseList.push(vm.getReqItemData(item));
              } else {
                var price_req = JSON.parse(localStorage.getItem(item.type_id));
                item.buy = price_req.buy;
                item.sell = price_req.sell;
              }
            }

            Promise.all(promiseList).then(() => {
              vm.updateItem(element);
              vm.curUpdateIndex++;

              if (vm.curUpdateIndex == vm.tableData.length) vm.loading = false;
            });
          });

          vm.tableData = d;
        }
      );
    },

    getLPStoreDataWithCookie: function (corporationid) {
      this.getLPStoreData(corporationid, this.useCookie);
    },

    updateItem: function (item) {
      item.lp_rate_buy =
        (item.buy * item.quantity - item.isk_cost) / item.lp_cost;
      item.lp_rate_sell =
        (item.sell * item.quantity - item.isk_cost) / item.lp_cost;

      var valid = true;
      var req_isk = 0;
      for (var i = 0; i < item.required_items.length; i++) {
        var r_item = item.required_items[i];

        if (r_item.sell == 0) {
          valid = false;
          break;
        } else {
          req_isk += r_item.sell * r_item.quantity;
        }
      }

      if (valid) {
        item.lp_rate_buy_all =
          (item.buy * item.quantity - item.isk_cost - req_isk) / item.lp_cost;
        item.lp_rate_sell_all =
          (item.sell * item.quantity - item.isk_cost - req_isk) / item.lp_cost;
      } else {
        item.lp_rate_buy_all = -99999;
        item.lp_rate_sell_all = -99999;
      }

      if (isNaN(item.lp_rate_buy_all)) {
        console.assert(false);
      }
    },

    getCorporationData: function () {
      var vm = this;
      fs.readFile(
        path.join(__static, "/LPStore/corporation_description.json"),
        "utf-8",
        function (err, data) {
          var d = JSON.parse(data);
          vm.corporationData = d;
        }
      );
    },

    getItemDescription: function () {
      var vm = this;
      fs.readFile(
        path.join(__static, "/LPStore/loyalty_description.json"),
        "utf-8",
        function (err, data) {
          var d = JSON.parse(data);
          vm.itemDescription = new Map(d);
        }
      );
    },

    getBlueprintProduct: function () {
      var vm = this;
      fs.readFile(
        path.join(__static, "/LPStore/blueprint_product.json"),
        "utf-8",
        function (err, data) {
          var d = JSON.parse(data);
          vm.blueprintProduct = new Map(d);
        }
      );
    },
  },

  created() {
    this.getItemDescription();
    this.getCorporationData();
    this.getBlueprintProduct();
    // this.getLPStoreData(1000001)
  },
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
h3 {
  margin: 40px 0 0;
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
</style>
