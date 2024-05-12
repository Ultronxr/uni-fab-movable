## 1.3.2 (2024-05-12)

- 修复：在组件 `created()` 生命周期内就会初始化按钮位置。避免出现按钮初始位置与其该在的位置距离相差过大，导致刚进页面时按钮出现跳跃现象。
- 新增：`avoidDropInTabbar` 属性，值为 `true` 时可以避免落入底部的 tabbar 区域（tabbar 的点击事件优先于 movable 的拖动事件，落入之后很难拖出来）

## 1.3.1 (2024-04-17)

- 新增自定义图片 icon 功能（原先只能使用 uni-icons ）

## 1.3.0 (2024-04-15)

- 新增 movable 可拖动功能
## 1.2.5（2023-03-29）
- 新增 pattern.icon 属性，可自定义图标
## 1.2.4（2022-09-07）
小程序端由于 style 使用了对象导致报错，[详情](https://ask.dcloud.net.cn/question/152790?item_id=211778&rf=false)
## 1.2.3（2022-09-05）
- 修复 nvue 环境下，具有 tabBar 时，fab 组件下部位置无法正常获取 --window-bottom 的bug，详见：[https://ask.dcloud.net.cn/question/110638?notification_id=826310](https://ask.dcloud.net.cn/question/110638?notification_id=826310)
## 1.2.2（2021-12-29）
- 更新 组件依赖
## 1.2.1（2021-11-19）
- 修复 阴影颜色不正确的bug
## 1.2.0（2021-11-19）
- 优化 组件UI，并提供设计资源，详见:[https://uniapp.dcloud.io/component/uniui/resource](https://uniapp.dcloud.io/component/uniui/resource)
- 文档迁移，详见:[https://uniapp.dcloud.io/component/uniui/uni-fab](https://uniapp.dcloud.io/component/uniui/uni-fab)
## 1.1.1（2021-11-09） 
- 新增 提供组件设计资源，组件样式调整
## 1.1.0（2021-07-30）
- 组件兼容 vue3，如何创建vue3项目，详见 [uni-app 项目支持 vue3 介绍](https://ask.dcloud.net.cn/article/37834)
## 1.0.7（2021-05-12）
- 新增 组件示例地址
## 1.0.6（2021-02-05）
- 调整为uni_modules目录规范
- 优化 按钮背景色调整
- 优化 兼容pc端
