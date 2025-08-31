<template>
    <view class="modern-cart">
        <view class="empty" v-if="list.length === 0">
            <!-- 购物车为空 -->
            <view class="empty-container">
                <view class="empty-icon">
                    <u-icon name="shopping-cart" color="#ddd" size="100"></u-icon>
                </view>
                <view class="empty-text">
                    <text class="empty-title">购物车为空</text>
                    <text class="empty-subtitle">快去挑选喜欢的商品吧</text>
                </view>
                <view class="empty-button">
                    <u-button type="primary" :customStyle="btnStyle" text="去逛逛" @click="toHome"></u-button>
                </view>
            </view>
        </view>

        <!-- 购物车商品列表 -->
        <view class="cart-container" v-else>
            <scroll-view :scroll-y="shouldScroll"
                         :style="{ height: height + 'px' }"
                         refresher-enabled="shouldEnableRefresh"
                         :refresher-threshold="70"
                         :refresher-triggered="triggered"
                         @refresherrefresh="onRefresh"
                         @scroll="handleScroll"
                         :scroll-anchoring="true"
                         class="cart-scroll">
                <view class="cart-items">
                    <!-- 商品卡片 -->
                    <view class="cart-item" v-for="(cart, index) in list" :key="index"
                          @longpress="showDeleteCartDalog(index)">

                        <!-- 选择框 -->
                        <view class="item-checkbox"
                              :class="{'disabled': cart.goods.store <= 0 || cart.goods.store < cart.num}"
                              @tap.stop="checkedGoods(cart.ID, cart.checked, index)">
                            <image v-if="cart.checked == 1" src="../../static/select.png" class="checkbox-icon checked"></image>
                            <image v-else src="../../static/not_select.png" class="checkbox-icon unchecked"></image>
                        </view>

                        <!-- 商品内容 -->
                        <view class="item-content" @click.stop="toGoodsDetail(cart.goodsId)">
                            <!-- 商品图片 -->
                            <view class="item-image">
                                <image v-if="cart.goods.images && cart.goods.images.length > 0"
                                       :src="cart.goods.images[0].url"
                                       class="product-image"
                                       mode="aspectFill" />
                                <image v-else src="/static/nopicture.jpg"
                                       class="product-image"
                                       mode="aspectFill" />

                                <!-- 缺货遮罩 -->
                                <view class="stock-mask" v-if="cart.goods.store <= 0 || cart.goods.store < cart.num">
                                    <view class="stock-badge">补货中</view>
                                </view>
                            </view>

                            <!-- 商品信息 -->
                            <view class="item-info">
                                <view class="info-top">
                                    <text class="product-name">{{ cart.goods.name }}</text>
                                    <view class="product-tags">
                                        <text class="unit-tag">{{ cart.goods.unit }}</text>
                                        <text class="stock-tag" :class="{'low-stock': cart.goods.store < 10}">库存{{ cart.goods.store }}</text>
                                    </view>
                                </view>

                                <view class="info-bottom">
                                    <view class="price-box">
                                        <text class="currency">¥</text>
                                        <text class="price">{{ cart.goods.price > 0 && cart.goods.price < cart.goods.costPrice ? cart.goods.price : cart.goods.costPrice }}</text>
                                    </view>

                                    <!-- 数量选择器 -->
                                    <view class="quantity-control">
                                        <view class="quantity-btn minus"
                                              @tap.stop="addCartReq(cart.goods.ID, index, 2, cart.num - 1)"
                                              :class="{'disabled': cart.num <= 1}">
                                            <text class="btn-text">-</text>
                                        </view>
                                        <view class="quantity-number">{{ cart.num }}</view>
                                        <view class="quantity-btn plus"
                                              @tap.stop="addCartReq(cart.goods.ID, index, 1, cart.num + 1)"
                                              :class="{'disabled': cart.num >= cart.goods.store}">
                                            <text class="btn-text">+</text>
                                        </view>
                                    </view>
                                </view>
                            </view>
                        </view>
                    </view>
                </view>
            </scroll-view>

            <!-- 底部结算栏 -->
            <view class="checkout-bar">
                <view class="checkout-content">
                    <!-- 全选 -->
                    <view class="select-all" @tap="allCheck">
                        <image v-if="isCheckAll" src="../../static/select.png" class="checkbox-icon checked-small"></image>
                        <image v-else src="../../static/not_select.png" class="checkbox-icon unchecked-small"></image>
                        <text class="select-text">全选</text>
                    </view>

                    <!-- 价格信息 -->
                    <view class="price-info">
                        <view class="price-label">合计:</view>
                        <view class="total-price">
                            <text class="price-currency">¥</text>
                            <text class="price-amount">{{ total }}</text>
                        </view>
                    </view>

                    <!-- 结算按钮 -->
                    <view class="checkout-button" @tap="accounts">
                        <text class="button-text">结算</text>
                        <text class="button-count" v-if="selectedCount > 0">({{ selectedCount }})</text>
                    </view>
                </view>
            </view>
        </view>

        <!-- 删除确认弹窗 -->
        <u-modal :show="showDeleteCart"
                 :showCancelButton="true"
                 :closeOnClickOverlay="true"
                 @confirm="deleteCart"
                 @cancel="hideDeleteCartDalog"
                 @close="hideDeleteCartDalog"
                 :buttonReverse="true">
            <view class="delete-modal">
                <u-icon name="warning-fill" color="#ff9500" size="48"></u-icon>
                <text class="delete-title">确定要删除这个商品吗？</text>
                <text class="delete-subtitle">删除后将无法恢复</text>
            </view>
        </u-modal>

        <u-toast ref="toast" style="z-index: 9999"></u-toast>
    </view>
</template>

<script>
import {addCart, updateCart, selectAllCart, clearSelectAllCart, deleteCartByIds} from "@/api/cart";

export default {
    name: "modernCart",
    data() {
        return {
            statisticsIndex: false,
            total: 0,
            selectedCount: 0, // 选中的商品数量
            isCut: true, // 是否编辑
            isCheckAll: false, // 是否全选
            showDeleteCart: false, // 删除购物车
            currentDeleteIndex: 0, // 当前长按删除的商品索引
            scrollTop: 0, // 当前滚动位置
            isAtTop: true, // 是否在顶部，用于控制下拉刷新
            shouldScroll: false, // 是否应该允许滚动
            shouldEnableRefresh: false, // 是否应该启用下拉刷新
            btnStyle: {
                width: '140px',
                height: '44px',
                borderRadius: '22px',
                background: '#3c9cff',
                border: 'none',
                color: '#ffffff',
                fontSize: '16px',
                fontWeight: '500',
            },
        }
    },
    props: {
        list: {
            type: [Array],
            default: []
        },
        height: {
            type: Number,
            default: 0
        },
        // 下拉刷新状态
        triggered: {
            type: Boolean,
            default: false
        }
    },
    watch: {
        list: {
            handler(newVal, oldVal) {
                let checkedAll = true
                if (newVal.length > 0) {
                    newVal.forEach(item => {
                        // 库存大于选择数字
                        if (item.goods.store > 0 && item.goods.store >= item.num && item.checked === 0) {
                            checkedAll = false
                        }
                    })
                    this.statistics()
                }
                this.isCheckAll = checkedAll
                // 更新滚动和刷新状态
                this.updateScrollAndRefreshStatus()
            },
            deep: true,
            immediate: true
        },
        height: {
            handler() {
                this.updateScrollAndRefreshStatus()
            },
            immediate: true
        }
    },
    computed: {
        // 计算商品列表的总高度
        cartItemsHeight() {
            // 每个商品卡片高度约120px + 10px间距
            const itemHeight = 130;
            return this.list.length * itemHeight;
        },
        // 判断是否需要滚动
        needScroll() {
            return this.cartItemsHeight > this.height;
        }
    },
    methods: {
        // 结算
        accounts() {
            uni.navigateTo({
                url: '/pages/order/submit'
            })
        },
        // 显示删除提示框
        showDeleteCartDalog(id) {
            this.currentDeleteIndex = id
            this.showDeleteCart = true
        },
        hideDeleteCartDalog() {
            this.showDeleteCart = false
        },
        async deleteCart() {
            // 保存删除的商品信息用于失败时恢复
            const deletedItem = this.list[this.currentDeleteIndex]
            const deletedIndex = this.currentDeleteIndex

            // 立即从UI中删除商品
            this.$emit('deleteCart', deletedIndex)
            this.hideDeleteCartDalog()

            // 异步请求接口
            let ids = []
            ids.push(deletedItem.ID)
            let data = {
                ids: ids
            }

            try {
                const res = await deleteCartByIds(data, this.$refs.toast)
                if (res.code !== 0) {
                    // 接口失败，恢复删除的商品
                    this.list.splice(deletedIndex, 0, deletedItem)
                    this.$emit('update', this.list)
                    this.statistics()
                    this.$message(this.$refs.toast).error('删除失败，请重试')
                }
            } catch (error) {
                // 请求异常，恢复删除的商品
                this.list.splice(deletedIndex, 0, deletedItem)
                this.$emit('update', this.list)
                this.statistics()
                this.$message(this.$refs.toast).error('网络异常，请重试')
            }
        },
        //商品选择
        async checkedGoods(id, checked, index) {
            // 先保存原始状态用于失败时回滚
            const originalChecked = checked

            // 立即更新UI状态
            const newChecked = checked === 1 ? 0 : 1
            this.list[index].checked = newChecked
            this.$emit('update', this.list)

            // 更新全选状态和统计
            let checkedAll = true
            this.list.forEach(item => {
                if (item.checked === 0) {
                    checkedAll = false
                }
            })
            this.isCheckAll = checkedAll
            this.statistics()

            // 异步请求接口
            const data = {
                ID: id,
                checked: newChecked
            }

            try {
                const res = await updateCart(data, this.$refs.toast)
                if (res.code !== 0) {
                    // 接口失败，回滚状态
                    this.list[index].checked = originalChecked
                    this.$emit('update', this.list)

                    // 重新计算全选状态和统计
                    let checkedAll = true
                    this.list.forEach(item => {
                        if (item.checked === 0) {
                            checkedAll = false
                        }
                    })
                    this.isCheckAll = checkedAll
                    this.statistics()

                    this.$message(this.$refs.toast).error('操作失败，请重试')
                }
            } catch (error) {
                // 请求异常，回滚状态
                this.list[index].checked = originalChecked
                this.$emit('update', this.list)

                // 重新计算全选状态和统计
                let checkedAll = true
                this.list.forEach(item => {
                    if (item.checked === 0) {
                        checkedAll = false
                    }
                })
                this.isCheckAll = checkedAll
                this.statistics()

                this.$message(this.$refs.toast).error('网络异常，请重试')
            }
        },
        // type 1增 2减
        // num 数量
        async addCartReq(id, index, type, num) {
            // 验证数量
            if (num < this.list[index].goods.minCount) {
                this.$message(this.$refs.toast).error("商品数量不能小于 " + this.list[index].goods.minCount)
                return
            }
            if (num < 1) {
                this.$message(this.$refs.toast).error("商品数量不能小于 1")
                return
            }

            // 保存原始状态用于失败时回滚
            const originalNum = this.list[index].num

            // 立即更新UI状态
            this.list[index].num = num
            this.$emit('update', this.list)
            this.statistics()

            // 异步请求接口
            const data = {
                goodsId: id,
                specType: 0, // 单规格
                num: num
            }

            try {
                const res = await addCart(data)
                if (res.code !== 0) {
                    // 接口失败，回滚状态
                    this.list[index].num = originalNum
                    this.$emit('update', this.list)
                    this.statistics()
                    this.$message(this.$refs.toast).error('更新失败，请重试')
                    return false
                }
            } catch (error) {
                // 请求异常，回滚状态
                this.list[index].num = originalNum
                this.$emit('update', this.list)
                this.statistics()
                this.$message(this.$refs.toast).error('网络异常，请重试')
                return false
            }
        },
        //全选
        async allCheck() {
            // 保存原始状态用于失败时回滚
            const originalList = JSON.parse(JSON.stringify(this.list))
            const originalIsCheckAll = this.isCheckAll

            if (this.isCheckAll) {
                // 取消全选 - 先更新UI
                this.list.forEach(item => {
                    item.checked = 0
                })
                this.isCheckAll = false
                this.$emit('update', this.list)
                this.statistics()

                // 异步请求接口
                try {
                    const res = await clearSelectAllCart(this.$refs.toast)
                    if (res.code !== 0) {
                        // 接口失败，回滚状态
                        this.list = originalList
                        this.isCheckAll = originalIsCheckAll
                        this.$emit('update', this.list)
                        this.statistics()
                        this.$message(this.$refs.toast).error('操作失败，请重试')
                    }
                } catch (error) {
                    // 请求异常，回滚状态
                    this.list = originalList
                    this.isCheckAll = originalIsCheckAll
                    this.$emit('update', this.list)
                    this.statistics()
                    this.$message(this.$refs.toast).error('网络异常，请重试')
                }
            } else {
                // 全选 - 先更新UI
                this.list.forEach(item => {
                    if (item.goods.store > 0 && item.goods.store >= item.num) {
                        item.checked = 1
                    }
                })
                this.isCheckAll = true
                this.$emit('update', this.list)
                this.statistics()

                // 异步请求接口
                try {
                    const res = await selectAllCart(this.$refs.toast)
                    if (res.code !== 0) {
                        // 接口失败，回滚状态
                        this.list = originalList
                        this.isCheckAll = originalIsCheckAll
                        this.$emit('update', this.list)
                        this.statistics()
                        this.$message(this.$refs.toast).error('操作失败，请重试')
                    }
                } catch (error) {
                    // 请求异常，回滚状态
                    this.list = originalList
                    this.isCheckAll = originalIsCheckAll
                    this.$emit('update', this.list)
                    this.statistics()
                    this.$message(this.$refs.toast).error('网络异常，请重试')
                }
            }
        },
        //统计
        statistics() {
            let total = 0
            let count = 0
            this.list.forEach(c => {
                if (c.checked !== 1) {
                    return
                }
                if (c.goods.price > 0 && c.goods.price < c.goods.costPrice) {
                    total += c.goods.price * c.num
                } else {
                    total += c.goods.costPrice * c.num
                }
                count += c.num
            })
            this.total = total.toFixed(2)
            this.selectedCount = count
        },
        cut() {
            this.isCut = !this.isCut
            this.statisticsIndex = true
            this.allCheck()
        },
        // 更新滚动和刷新状态
        updateScrollAndRefreshStatus() {
            // 只有当商品数量足以填满容器时才允许滚动
            this.shouldScroll = this.needScroll;

            // 只有在需要滚动且在顶部时才允许下拉刷新
            if (this.needScroll) {
                this.shouldEnableRefresh = this.isAtTop;
            } else {
                // 如果不需要滚动，只有当有商品时才允许下拉刷新
                this.shouldEnableRefresh = this.list.length > 0;
            }
        },
        // 处理滚动事件
        handleScroll(e) {
            // 获取滚动位置
            this.scrollTop = e.detail.scrollTop;
            // 当滚动位置小于等于20px时认为在顶部
            this.isAtTop = this.scrollTop <= 20;
            // 更新刷新状态
            this.updateScrollAndRefreshStatus();
        },
        // 刷新
        onRefresh() {
            this.$emit('onRefresh')
        },
        toHome() {
            uni.navigateTo({
                url: '/pages/index/index'
            })
        },
        toGoodsDetail(id) {
            uni.navigateTo({
                url: '/pages/goods/detail?id=' + id
            })
        }
    }
}
</script>

<style lang="scss" scoped>

// 现代化购物车样式
.modern-cart {
    width: 100%;
    background: #f8f9fa;
    position: relative;
}

// 空购物车状态
.empty {
    display: flex;
    align-items: center;
    justify-content: center;
    min-height: 50vh;
    padding: 40px 20px;

    .empty-container {
        display: flex;
        flex-direction: column;
        align-items: center;
        text-align: center;
        max-width: 280px;
        width: 100%;
        animation: fadeIn 0.6s ease-out;

        .empty-icon {
            margin-bottom: 24px;
            opacity: 0.6;
            transition: all 0.3s ease;

            &:hover {
                opacity: 0.8;
                transform: scale(1.05);
            }
        }

        .empty-text {
            margin-bottom: 32px;

            .empty-title {
                display: block;
                font-size: 18px;
                font-weight: 600;
                color: #2c3e50;
                margin-bottom: 8px;
                line-height: 1.4;
            }

            .empty-subtitle {
                display: block;
                font-size: 14px;
                color: #7f8c8d;
                line-height: 1.5;
            }
        }

        .empty-button {
            animation: slideUp 0.6s ease-out 0.2s both;
            width: 100%;

            .u-button {
                transition: all 0.3s ease;
                box-shadow: 0 2px 8px rgba(60, 156, 255, 0.2);

                &:active {
                    transform: translateY(-1px);
                    box-shadow: 0 4px 12px rgba(60, 156, 255, 0.3);
                }
            }
        }
    }
}

// 购物车容器
.cart-container {
    display: flex;
    flex-direction: column;
    position: relative;

    .cart-items {
        display: flex;
        flex-direction: column;
        gap: 10px;
        padding: 8px 12px 12px;
    }
}

// 商品卡片
.cart-item {
    background: #ffffff;
    border-radius: 16px;
    box-shadow: 0 2px 8px rgba(0, 0, 0, 0.06);
    display: flex;
    padding: 14px 12px;
    transition: all 0.2s ease;

    &:active {
        transform: scale(0.99);
        box-shadow: 0 1px 4px rgba(0, 0, 0, 0.1);
    }

    .item-checkbox {
        margin-right: 12px;
        display: flex;
        align-items: center;
        justify-content: center;

        &.disabled {
            opacity: 0.5;
        }

        .checkbox-icon {
            width: 22px;
            height: 22px;
        }
    }

    .item-content {
        flex: 1;
        display: flex;
        align-items: flex-start;
        gap: 12px;
    }

    .item-image {
        position: relative;
        width: 85px;
        height: 85px;
        border-radius: 12px;
        overflow: hidden;
        flex-shrink: 0;

        .product-image {
            width: 100%;
            height: 100%;
            border-radius: 12px;
            object-fit: cover;
        }

        .stock-mask {
            position: absolute;
            top: 0;
            left: 0;
            right: 0;
            bottom: 0;
            background: rgba(0, 0, 0, 0.7);
            display: flex;
            align-items: center;
            justify-content: center;
            border-radius: 12px;

            .stock-badge {
                background: #ff4757;
                color: #ffffff;
                padding: 6px 12px;
                border-radius: 16px;
                font-size: 12px;
                font-weight: 600;
            }
        }
    }

    .item-info {
        flex: 1;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        min-height: 85px;

        .info-top {
            .product-name {
                font-size: 15px;
                font-weight: 600;
                color: #2c3e50;
                line-height: 1.4;
                display: -webkit-box;
                -webkit-line-clamp: 2;
                line-clamp: 2;
                -webkit-box-orient: vertical;
                overflow: hidden;
                margin-bottom: 6px;
            }

            .product-tags {
                display: flex;
                gap: 8px;
                flex-wrap: wrap;

                .unit-tag {
                    background: #ecf0f1;
                    color: #7f8c8d;
                    padding: 4px 8px;
                    border-radius: 12px;
                    font-size: 11px;
                    font-weight: 500;
                }

                .stock-tag {
                    background: #e8f5e9;
                    color: #27ae60;
                    padding: 4px 8px;
                    border-radius: 12px;
                    font-size: 11px;
                    font-weight: 500;

                    &.low-stock {
                        background: #fff3cd;
                        color: #f39c12;
                    }
                }
            }
        }

        .info-bottom {
            display: flex;
            justify-content: space-between;
            align-items: center;
            margin-top: 8px;

            .price-box {
                display: flex;
                align-items: baseline;

                .currency {
                    font-size: 14px;
                    color: #ff4757;
                    font-weight: 600;
                    margin-right: 2px;
                }

                .price {
                    font-size: 18px;
                    color: #ff4757;
                    font-weight: 700;
                }
            }

            .quantity-control {
                display: flex;
                align-items: center;
                background: #f8f9fa;
                border-radius: 18px;
                padding: 3px;
                border: 1px solid #e9ecef;

                .quantity-btn {
                    width: 28px;
                    height: 28px;
                    border-radius: 14px;
                    display: flex;
                    align-items: center;
                    justify-content: center;
                    transition: all 0.2s ease;
                    background: #ffffff;

                    &.minus {
                        margin-right: 4px;
                    }

                    &.plus {
                        margin-left: 4px;
                    }

                    &.disabled {
                        opacity: 0.4;
                        pointer-events: none;
                    }

                    &:active:not(.disabled) {
                        background: #3c9cff;
                        .btn-text {
                            color: #ffffff;
                        }
                    }

                    .btn-text {
                        font-size: 16px;
                        font-weight: 600;
                        color: #2c3e50;
                    }
                }

                .quantity-number {
                    min-width: 40px;
                    text-align: center;
                    font-size: 16px;
                    font-weight: 600;
                    color: #2c3e50;
                }
            }
        }
    }
}

// 底部结算栏
.checkout-bar {
    position: fixed;
    bottom: 50px;
    left: 0;
    right: 0;
    background: #ffffff;
    border-top: 1px solid #e9ecef;
    box-shadow: 0 -1px 6px rgba(0, 0, 0, 0.08);
    z-index: 100;

    .checkout-content {
        display: flex;
        align-items: center;
        justify-content: space-between;
        padding: 8px 16px;
        height: 50px;

        .select-all {
            display: flex;
            align-items: center;
            gap: 8px;

            .checkbox-icon {
                width: 20px;
                height: 20px;
            }

            .select-text {
                font-size: 14px;
                color: #2c3e50;
                font-weight: 500;
            }
        }

        .price-info {
            display: flex;
            align-items: center;
            gap: 4px;

            .price-label {
                font-size: 14px;
                color: #7f8c8d;
            }

            .total-price {
                display: flex;
                align-items: baseline;

                .price-currency {
                    font-size: 12px;
                    color: #ff4757;
                    font-weight: 500;
                }

                .price-amount {
                    font-size: 18px;
                    color: #ff4757;
                    font-weight: 600;
                }
            }
        }

        .checkout-button {
            background: #3c9cff;
            border-radius: 16px;
            padding: 8px 20px;
            display: flex;
            align-items: center;
            gap: 4px;
            transition: all 0.2s ease;

            &:active {
                transform: scale(0.95);
            }

            .button-text {
                font-size: 14px;
                font-weight: 600;
                color: #ffffff;
            }

            .button-count {
                font-size: 11px;
                color: rgba(255, 255, 255, 0.8);
            }
        }
    }
}

// 删除确认弹窗
.delete-modal {
    display: flex;
    flex-direction: column;
    align-items: center;
    padding: 20px;

    .delete-title {
        font-size: 16px;
        font-weight: 600;
        color: #2c3e50;
        margin-top: 12px;
        margin-bottom: 4px;
    }

    .delete-subtitle {
        font-size: 14px;
        color: #7f8c8d;
    }
}

// 动画效果
@keyframes fadeIn {
    from {
        opacity: 0;
        transform: translateY(15px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

@keyframes slideUp {
    from {
        opacity: 0;
        transform: translateY(10px);
    }
    to {
        opacity: 1;
        transform: translateY(0);
    }
}

// 响应式设计
@media (max-width: 375px) {
    .empty {
        .empty-container {
            max-width: 240px;

            .empty-text {
                .empty-title {
                    font-size: 16px;
                }

                .empty-subtitle {
                    font-size: 13px;
                }
            }
        }
    }
}
</style>
