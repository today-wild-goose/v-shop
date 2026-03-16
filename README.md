# v-shop（uni-app + uni-ui）

一个基于 uni-app（Vue3 组合式 API）与 uni-ui 搭建的简易购物网站示例工程，包含首页/分类/发现/购物车/我的五个 Tab 页面，并实现了商品详情、搜索、登录拦截等基础能力。

## 技术栈

- uni-app（Vue 3）
- uni-ui（通过 `uni_modules` 方式集成）
- 样式单位统一 `rpx`

## 功能概览

- TabBar 五页面：首页 / 分类 / 发现 / 购物车 / 我的
- 首页：顶部固定搜索栏、Banner 轮播、分类 Tab、推荐商品列表、回到顶部
- 分类：左侧一级分类 + 右侧二级分类网格 + 推荐商品列表
- 发现：Banner、分段 Tab（推荐/活动/种草/专题）、信息流列表、回到顶部
- 商品详情：轮播、价格标题标签、详情/参数/评价、规格弹层、底部操作栏（店铺/购物车/加入/购买）
- 搜索页：热搜、搜索历史、结果列表、跳转商品详情
- 登录/拦截：默认未登录；未登录进入购物车/我的会跳转登录页；登录后可正常展示内容；支持退出登录

## 页面路由

- `/pages/index/index`：首页（Tab）
- `/pages/category/category`：分类（Tab）
- `/pages/discover/discover`：发现（Tab）
- `/pages/cart/cart`：购物车（Tab，需登录）
- `/pages/mine/mine`：我的（Tab，需登录）
- `/pages/goods-detail/goods-detail?id=xxx`：商品详情
- `/pages/search/search?q=xxx`：搜索
- `/pages/login/login`：登录

## 登录与本地存储约定

当前登录/购物车均为“本地模拟”逻辑，用于快速打通页面与交互。

- 登录状态
  - `vshop_token`：是否登录的标记（存在即认为已登录）
  - `vshop_user`：用户信息 JSON（`{ id, nickname, avatar }`）
- 购物车（商品详情页加入购物车时写入）
  - `vshop_cart`：购物车数组 JSON
    - 元素示例：
      - `goodsId`：商品 id
      - `skuId`：规格 id
      - `title`：商品标题
      - `skuText`：规格文本
      - `price`：价格（number）
      - `image`：主图
      - `quantity`：数量
      - `checked`：是否勾选
- 搜索历史
  - `vshop_search_history`：搜索关键词数组 JSON（最多 12 条，去重）

## 运行方式

该项目为 uni-app 工程结构，推荐使用 HBuilderX 运行：

1. 使用 HBuilderX 打开项目根目录 `v-shop`
2. 运行到目标平台（如：运行到浏览器 / 微信开发者工具 / App）

## 开发说明

- 目前商品/分类/信息流数据为页面内 mock 数据，后续可替换为真实接口返回
- 商品详情页支持通过 `id` 查询 mock 详情数据；首页、分类、搜索页点击商品会跳转详情页
- 购物车页目前含 UI 与交互（勾选/全选/数量/删除/结算），后续可进一步对接 `vshop_cart` 做数据打通

