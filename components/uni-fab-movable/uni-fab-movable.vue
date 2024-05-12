<template>
	<movable-area class="movable-area" :scale-area="false">
		<movable-view class="movable-view uni-cursor-point"
			:class="!movable.isMoving?'animation-info':''" style="pointer-events: auto;"
			@touchstart="touchstart" @touchend="touchend" @change="onChange" direction="all"
			inertia="true" :x="movable.x" :y="movable.y" :disabled="movableDisabled" :out-of-bounds="true" :damping="200" :friction="100"
		>
			<view v-if="popMenu && (leftBottom||rightBottom||leftTop||rightTop) && content.length > 0" :class="{
				'uni-fab--leftBottom': leftBottom,
				'uni-fab--rightBottom': rightBottom,
				'uni-fab--leftTop': leftTop,
				'uni-fab--rightTop': rightTop
			  }" class="uni-fab"
				:style="nvueBottom"
			>
				<view :class="{
				  'uni-fab__content--left': horizontal === 'left',
				  'uni-fab__content--right': horizontal === 'right',
				  'uni-fab__content--flexDirection': direction === 'vertical',
				  'uni-fab__content--flexDirectionStart': flexDirectionStart,
				  'uni-fab__content--flexDirectionEnd': flexDirectionEnd,
				  'uni-fab__content--other-platform': !isAndroidNvue
				}" :style="{ width: boxWidth, height: boxHeight, backgroundColor: styles.backgroundColor }"
					class="uni-fab__content" elevation="5"
				>
					<view v-if="flexDirectionStart || horizontalLeft" class="uni-fab__item uni-fab__item--first" />
					<view v-for="(item, index) in content" :key="index" :class="{ 'uni-fab__item--active': isShow }"
						class="uni-fab__item" @click="_onItemClick(index, item)">
						<image :src="item.active ? item.selectedIconPath : item.iconPath" class="uni-fab__item-image"
							mode="aspectFit" />
						<text class="uni-fab__item-text"
							:style="{ color: item.active ? styles.selectedColor : styles.color }">{{ item.text }}</text>
					</view>
					<view v-if="flexDirectionEnd || horizontalRight" class="uni-fab__item uni-fab__item--first" />
				</view>
			</view>
			<view :class="{
			  'uni-fab__circle--leftBottom': leftBottom,
			  'uni-fab__circle--rightBottom': rightBottom,
			  'uni-fab__circle--leftTop': leftTop,
			  'uni-fab__circle--rightTop': rightTop,
			  'uni-fab__content--other-platform': !isAndroidNvue
			}" class="uni-fab__circle uni-fab__plus"
				:style="{ 'background-color': styles.buttonColor, 'bottom': nvueBottom }"
				@click="_onClick"
			>
				<uni-icons v-if="styles.imageIconSrc == null" class="fab-circle-icon"
					:type="styles.icon" :color="styles.iconColor" size="32"
					:class="{'uni-fab__plus--active': isShow && content.length > 0}"></uni-icons>
				<!-- <view class="fab-circle-v"  :class="{'uni-fab__plus--active': isShow && content.length > 0}"></view>
				<view class="fab-circle-h" :class="{'uni-fab__plus--active': isShow  && content.length > 0}"></view> -->
				
				<!-- 自定义图片图标 -->
				<image v-if="styles.imageIconSrc != null" class="fab-circle-icon-image" :src="styles.imageIconSrc" />
			</view>
		</movable-view>
	</movable-area>
</template>

<script>
	let platform = 'other'
	// #ifdef APP-NVUE
	platform = uni.getSystemInfoSync().platform
	// #endif

	/**
	 * FabMovable 可移动悬浮按钮
	 * @property {Object} pattern 可选样式配置项
	 * @property {Object} direction = [horizontal | vertical] 展开菜单显示方式
	 * 	@value horizontal 水平显示
	 * 	@value vertical 垂直显示
	 * @property {Array} content 展开菜单内容配置项
	 * @property {Boolean} popMenu 是否使用弹出菜单
	 * @property {Boolean} movableDisabled 是否禁用可拖动功能；true - 不可拖动、false - 可拖动
	 * @property {Boolean} movableCanDocking 是否开启自动停靠屏幕边缘；true - 自动停靠屏幕左右侧边缘、false - 不开启自动停靠
	 * @property {Number} movableTopPx 距离屏幕顶部的初始距离
	 * @property {Number} movableRightPx 距离屏幕右侧的初始距离
	 * @property {Number} movableOffsetToLeftOrRight 距离屏幕左右侧的向内偏移量（按钮不直接贴在左/右侧边缘上，而是向内悬浮一定距离）
	 * @event {Function} trigger 展开菜单点击事件，返回点击信息
	 * @event {Function} fabClick 悬浮按钮点击事件
	 */
	export default {
		name: 'UniFabMovable',
		emits: ['fabClick', 'trigger'],
		props: {
			pattern: {
				type: Object,
				default () {
					return {}
				}
			},
			direction: {
				type: String,
				default: 'horizontal'
			},
			content: {
				type: Array,
				default () {
					return []
				}
			},
			show: {
				type: Boolean,
				default: false
			},
			popMenu: {
				type: Boolean,
				default: true
			},
			
			// ========== movable 内容 ========== //
			// 是否禁用拖动
			movableDisabled: {
				type: Boolean,
				default: false
			},
			// 是否自动停靠
			movableCanDocking: {
				type: Boolean,
				default: true
			},
			// 按钮默认位置离顶部距离（px）
			movableTopPx: {
				type: Number,
				default: 150
			},
			// 按钮默认位置离右边距离（px）
			movableRightPx: {
				type: Number,
				default: 0
			},
			// 距离左侧/右侧边界的偏移量（不直接贴在边上）
			movableOffsetToLeftOrRight: {
				type: Number,
				default: 0
			},
			// 是否避免落入底部的 tabbar 区域（tabbar 的点击事件优先于 movable 的拖动事件，落入之后很难拖出来）
			avoidDropInTabbar: {
				type: Boolean,
				default: false
			}
		},
		data() {
			return {
				fabShow: false,
				isShow: false,
				isAndroidNvue: platform === 'android',
				styles: {
					color: '#3c3e49',
					selectedColor: '#007AFF',
					backgroundColor: '#fff',
					buttonColor: '#007AFF',
					iconColor: '#fff',
					icon: 'plusempty',
					imageIconSrc: null,
				},
				movable: {
					left: 0,
					top: 0,
					isMoving: true,
					windowWidth: 0,
					windowHeight: 0,
					btnWidth: 0,
					btnHeight: 0,
					x: 0,
					y: 0,
					old: {
						x: 0,
						y: 0
					}
				}
			}
		},
		computed: {
			// 按钮左右侧位置属性（该属性并不决定按钮位置）
			horizontal() {
				return this.movable.x < this.movable.windowWidth / 2 ? 'left' : 'right'
			},
			// 按钮左右侧位置属性（该属性并不决定按钮位置）
			vertical() {
				return this.movable.y < this.movable.windowHeight / 2 ? 'top' : 'bottom'
			},
			contentWidth(e) {
				return (this.content.length + 1) * 58 + 15 + 'px'
			},
			contentWidthMin() {
				return '58px'
			},
			// 动态计算宽度
			boxWidth() {
				return this.getPosition(3, 'horizontal')
			},
			// 动态计算高度
			boxHeight() {
				return this.getPosition(3, 'vertical')
			},
			// 计算左下位置
			leftBottom() {
				return this.getPosition(0, 'left', 'bottom')
			},
			// 计算右下位置
			rightBottom() {
				return this.getPosition(0, 'right', 'bottom')
			},
			// 计算左上位置
			leftTop() {
				return this.getPosition(0, 'left', 'top')
			},
			rightTop() {
				return this.getPosition(0, 'right', 'top')
			},
			flexDirectionStart() {
				return this.getPosition(1, 'vertical', 'top')
			},
			flexDirectionEnd() {
				return this.getPosition(1, 'vertical', 'bottom')
			},
			horizontalLeft() {
				return this.getPosition(2, 'horizontal', 'left')
			},
			horizontalRight() {
				return this.getPosition(2, 'horizontal', 'right')
			},
			// 计算 nvue bottom
			nvueBottom() {
				const safeBottom = uni.getSystemInfoSync().windowBottom;
				// #ifdef APP-NVUE
				return 30 + safeBottom
				// #endif
				// #ifndef APP-NVUE
				return 30
				// #endif
			},
		},
		watch: {
			pattern: {
				handler(val, oldVal) {
					this.styles = Object.assign({}, this.styles, val)
				},
				deep: true
			}
		},
		created() {
			this.isShow = this.show
			if (this.top === 0) {
				this.fabShow = true
			}
			// 初始化样式
			this.styles = Object.assign({}, this.styles, this.pattern)
			// 初始化位置
			this.initPosition();
		},
		mounted() {
			this.getSysInfo()
		},
		methods: {
			_onClick() {
				this.$emit('fabClick')
				if (!this.popMenu) {
					return
				}
				this.isShow = !this.isShow
			},
			open() {
				this.isShow = true
			},
			close() {
				this.isShow = false
			},
			/**
			 * 按钮点击事件
			 */
			_onItemClick(index, item) {
				if (!this.isShow) {
					return
				}
				this.$emit('trigger', {
					index,
					item
				})
			},
			/**
			 * 获取 位置信息
			 */
			getPosition(types, paramA, paramB) {
				if (types === 0) {
					return this.horizontal === paramA && this.vertical === paramB
				} else if (types === 1) {
					return this.direction === paramA && this.vertical === paramB
				} else if (types === 2) {
					return this.direction === paramA && this.horizontal === paramB
				} else {
					return this.isShow && this.direction === paramA ? this.contentWidth : this.contentWidthMin
				}
			},
			/**
			 * 初始化按钮位置。避免出现按钮初始位置与其该在的位置距离相差过大，导致刚进页面时按钮出现跳跃现象
			 */
			initPosition() {
				let sysInfo = uni.getSystemInfoSync();
				this.movable.windowWidth = sysInfo.windowWidth;
				this.movable.windowHeight = sysInfo.windowHeight;
				this.movable.x = this.movable.windowWidth - this.movable.btnWidth - this.movableRightPx - this.movableOffsetToLeftOrRight;
				this.movable.y = 0 + this.movableTopPx;
				this.movable.old.x = this.movable.x;
				this.movable.old.y = this.movable.y;
			},
			getSysInfo() {
				let view = uni.createSelectorQuery().in(this).select(".movable-view")
				view.boundingClientRect(rect => {
					this.movable.btnWidth = rect.width
					this.movable.btnHeight = rect.height
					this.movable.x = this.movable.old.x
					this.movable.y = this.movable.old.y
					this.$nextTick(res => {
						this.movable.x = this.movable.windowWidth - this.movable.btnWidth - this.movableRightPx - this.movableOffsetToLeftOrRight
						this.movable.y = 0 + this.movableTopPx
					})
				}).exec()
			},
			//移动按钮
			onChange(e) {
				this.movable.old.x = e.detail.x
				this.movable.old.y = e.detail.y
			},
			//开始移动
			touchstart(e) {
				this.movable.isMoving = true
			},
			//结束移动
			touchend(e) {
				if (this.movableCanDocking && this.movable.old.x) {
					this.movable.x = this.movable.old.x
					this.movable.y = this.movable.old.y
					
					let halfWidth = (this.movable.windowWidth - this.movable.btnWidth) / 2
					// 检查移动结束时，按钮位置在屏幕的左半侧或右半侧
					if (this.movable.x < 0 || (this.movable.x > 0 && this.movable.x <= halfWidth)) {
						this.$nextTick(res => {
							this.movable.x = 0 + this.movableOffsetToLeftOrRight
						})
					} else {
						this.$nextTick(res => {
							this.movable.x = this.movable.windowWidth - this.movable.btnWidth - this.movableOffsetToLeftOrRight
						})
					}
					// 避免按钮落入底部 tab 栏中
					if(this.avoidDropInTabbar && this.movable.y >= this.movable.windowHeight - 100 - this.movable.btnHeight / 2) {
						this.movable.y = this.movable.windowHeight - 100 - this.movable.btnHeight / 2;
					}
					
					this.movable.isMoving = false
				}
			},
		}
	}
</script>

<style lang="scss" >
	$uni-shadow-base: 0 1px 5px 2px rgba($color: #000000, $alpha: 0.3) !default;
	$button-width: 58px !default;
	$button-height: 58px !default;
	$button-border-radius: 58px !default;

	.uni-fab {
		position: fixed;
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		justify-content: center;
		align-items: center;
		z-index: 9999;
		border-radius: $button-border-radius;
		box-shadow: $uni-shadow-base;
	}

	.uni-cursor-point {
		/* #ifdef H5 */
		cursor: pointer;
		/* #endif */
	}

	.uni-fab--active {
		opacity: 1;
	}

	.uni-fab--leftBottom {
		left: 0px;
		bottom: 0px;
		/* #ifdef H5 */
		left: calc(0px + var(--window-left));
		bottom: calc(0px + var(--window-bottom));
		/* #endif */
		// padding: 10px;
	}

	.uni-fab--leftTop {
		left: 0px;
		top: 0px;
		/* #ifdef H5 */
		left: calc(0px + var(--window-left));
		top: calc(0px + var(--window-top));
		/* #endif */
		// padding: 10px;
	}

	.uni-fab--rightBottom {
		right: 0px;
		bottom: 0px;
		/* #ifdef H5 */
		right: calc(0px + var(--window-right));
		bottom: calc(0px + var(--window-bottom));
		/* #endif */
		// padding: 10px;
	}

	.uni-fab--rightTop {
		right: 0px;
		top: 0px;
		/* #ifdef H5 */
		right: calc(0px + var(--window-right));
		top: calc(0px + var(--window-top));
		/* #endif */
		// padding: 10px;
	}

	.uni-fab__circle {
		position: fixed;
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		justify-content: center;
		align-items: center;
		width: $button-width;
		height: $button-height;
		background-color: #3c3e49;
		border-radius: $button-border-radius;
		z-index: 9999;
		box-shadow: $uni-shadow-base;
	}
	
	.uni-fab__circle--leftBottom {
		left: 0px;
		bottom: 0px;
		/* #ifdef H5 */
		left: calc(0px + var(--window-left));
		bottom: calc(0px + var(--window-bottom));
		/* #endif */
	}
	
	.uni-fab__circle--leftTop {
		left: 0px;
		top: 0px;
		/* #ifdef H5 */
		left: calc(0px + var(--window-left));
		top: calc(0px + var(--window-top));
		/* #endif */
	}
	
	.uni-fab__circle--rightBottom {
		right: 0px;
		bottom: 0px;
		/* #ifdef H5 */
		right: calc(0px + var(--window-right));
		bottom: calc(0px + var(--window-bottom));
		/* #endif */
	}
	
	.uni-fab__circle--rightTop {
		right: 0px;
		top: 0px;
		/* #ifdef H5 */
		right: calc(0px + var(--window-right));
		top: calc(0px + var(--window-top));
		/* #endif */
	}

	.uni-fab__circle--left {
		left: 0;
	}

	.uni-fab__circle--right {
		right: 0;
	}

	.uni-fab__circle--top {
		top: 0;
	}

	.uni-fab__circle--bottom {
		bottom: 0;
	}

	.uni-fab__plus {
		font-weight: bold;
	}

	// .fab-circle-v {
	// 	position: absolute;
	// 	width: 2px;
	// 	height: 24px;
	// 	left: 0;
	// 	top: 0;
	// 	right: 0;
	// 	bottom: 0;
	// 	/* #ifndef APP-NVUE */
	// 	margin: auto;
	// 	/* #endif */
	// 	background-color: white;
	// 	transform: rotate(0deg);
	// 	transition: transform 0.3s;
	// }

	// .fab-circle-h {
	// 	position: absolute;
	// 	width: 24px;
	// 	height: 2px;
	// 	left: 0;
	// 	top: 0;
	// 	right: 0;
	// 	bottom: 0;
	// 	/* #ifndef APP-NVUE */
	// 	margin: auto;
	// 	/* #endif */
	// 	background-color: white;
	// 	transform: rotate(0deg);
	// 	transition: transform 0.3s;
	// }

	.fab-circle-icon {
		transform: rotate(0deg);
		transition: transform 0.3s;
		font-weight: 200;
	}
	
	.fab-circle-icon-image {
		width: $button-width;
		height: $button-height;
	}

	.uni-fab__plus--active {
		transform: rotate(135deg);
	}

	.uni-fab__content {
		/* #ifndef APP-NVUE */
		box-sizing: border-box;
		display: flex;
		/* #endif */
		flex-direction: row;
		border-radius: $button-border-radius;
		overflow: hidden;
		transition-property: width, height;
		transition-duration: 0.2s;
		width: $button-width;
		border-color: #DDDDDD;
		border-width: 1rpx;
		border-style: solid;
	}

	.uni-fab__content--other-platform {
		border-width: 0px;
		box-shadow: $uni-shadow-base;
	}

	.uni-fab__content--left {
		justify-content: flex-start;
	}

	.uni-fab__content--right {
		justify-content: flex-end;
	}

	.uni-fab__content--flexDirection {
		flex-direction: column;
		justify-content: flex-end;
	}

	.uni-fab__content--flexDirectionStart {
		flex-direction: column;
		justify-content: flex-start;
	}

	.uni-fab__content--flexDirectionEnd {
		flex-direction: column;
		justify-content: flex-end;
	}

	.uni-fab__item {
		/* #ifndef APP-NVUE */
		display: flex;
		/* #endif */
		flex-direction: column;
		justify-content: center;
		align-items: center;
		width: $button-width;
		height: $button-height;
		opacity: 0;
		transition: opacity 0.2s;
	}

	.uni-fab__item--active {
		opacity: 1;
	}

	.uni-fab__item-image {
		width: 20px;
		height: 20px;
		margin-bottom: 4px;
	}

	.uni-fab__item-text {
		color: #FFFFFF;
		font-size: 12px;
		line-height: 12px;
		margin-top: 2px;
	}

	.uni-fab__item--first {
		width: $button-width;
	}
	
	.movable-area {
		width: 100%;
		height: 100%;
		position: fixed;
		top: 0;
		left: 0;
		right: 0;
		bottom: 0;
		z-index: 999999 !important;
		pointer-events: none;
	}
	
	.movable-view {
		width: $button-width;
		height: $button-height;
		// background: linear-gradient(360deg, #287BF8 0%, #6EA8FF 100%);
		// box-shadow: 0px 4rpx 12rpx 0px #ADC3F8;
		// border-radius: 50rpx;
		// color: #FFFFFF;
		// font-size: 26rpx;
		touch-action: none;
		display: flex;
		align-items: center;
		justify-content: center;
	}
	
	.animation-info {
		transition: left .25s ease;
	}
</style>
