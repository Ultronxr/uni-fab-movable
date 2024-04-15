## 可拖动的 Fab 悬浮按钮

> **组件名：uni-fab-movable**

基于 `uni-fab` v1.2.5 版本修改，官方文档见此：[链接](https://uniapp.dcloud.io/component/uniui/uni-fab) 查看。

本组件的拖动功能使用 uniapp 的 `movable-area` 与 `movable-view` 实现，官方文档见此：[movable-area](https://uniapp.dcloud.net.cn/component/movable-area.html) 、[movable-view](https://uniapp.dcloud.net.cn/component/movable-view.html) 。

## 使用示例

```html
<template>
    <view>
        <uniFabMovable ref="fab" :pattern="fabPattern" :content="fabContent"
            horizontal="right" vertical="top"
            direction="horizontal" @fabClick="fabClick"
            :movableDisabled="false" :movableCanDocking="true"
            :movableTopPx="150" :movableRightPx="0" :movableOffsetToLeftOrRight="3" />
    </view>
</template>
```

```javascript
<script>
    import uniFabMovable from "@/uni_modules/uni-fab/components/uni-fab/uni-fab-movable.vue"
    export default {
    components: {
        uniFabMovable,
    },
    data() {
        return {
            // 悬浮按钮样式
            fabPattern: {
                color: '#7A7E83',
                backgroundColor: '#fff',
                selectedColor: '#007AFF',
                buttonColor: '#007AFF',
                iconColor: '#fff'
            },
            fabContent: [
                {
                    iconPath: '/static/image.png',
                    selectedIconPath: '/static/image-active.png',
                    text: '相册',
                    active: false
                },
            ],
            // 悬浮按钮点击事件
            fabClick() {
            },
        }
    },
</script>
```
