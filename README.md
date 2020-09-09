# lp-calculator-electorn

> An electron-vue project

国服EVE LP兑换计算器 价格基于国服市场中心

## 右侧 Releases 里面下载生成好的应用程序，当然你不放心可以自己build。

## 请注意兑换比例仅供参考，注意游戏中实际成交量，很多东西兑换比较高但是成交量极少，不要一股脑换完没法出货就傻眼了。

# 前排接受国服EVE捐赠，ID: LittleDemo

# 使用说明

启动后，在上面选择军团，然后边上有一个数字在变化，这是在从国服市场中心取得数据。等数字走完后，**需要点击一下数据列边上的箭头进行排序**，数据会显示出来。
**点击每个物品前面的箭头会显示该物品的订单情况，以及一个简单的数量计算器。**

“显示蓝图” 表示是否以蓝图制造的物品来计算lp，关闭的话，蓝图的价值是0。

“使用缓存” 从市场取得的数据会缓存到本地，不会每次都到国服市场中心取数据，这个开关是控制是不是强制每次都重取。

#### Build Setup

``` bash
# install dependencies
npm install

# serve with hot reload at localhost:9080
npm run dev

# build electron application for production
npm run build


```

---

This project was generated with [electron-vue](https://github.com/SimulatedGREG/electron-vue) using [vue-cli](https://github.com/vuejs/vue-cli). Documentation about the original structure can be found [here](https://simulatedgreg.gitbooks.io/electron-vue/content/index.html).
