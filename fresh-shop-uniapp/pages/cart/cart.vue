<template>
    <pageWrapper>
        <!-- 未登录状态 -->
        <view class="login-container" v-if="!token">
            <view class="login-card">
                <view class="login-icon">
                    <u-icon name="account-circle" color="#3c9cff" size="80"></u-icon>
                </view>
                <view class="login-text">
                    <text class="login-title">登录后查看购物车</text>
                    <text class="login-subtitle">登录即可享受专属优惠和便捷购物体验</text>
                </view>
                <view class="login-button">
                    <u-button type="primary" :customStyle="toLoginStyle" text="立即登录" @click="showLogin" />
                </view>
            </view>
        </view>

        <!-- 已登录状态 -->
        <view class="cart-container" v-else>
            <!-- 页面标题 -->
            <view class="page-header">
                <text class="page-title">我的购物车</text>
                <view class="cart-count" v-if="list.length > 0">
                    <text class="count-text">{{ list.length }}件商品</text>
                </view>
            </view>

            <!-- 购物车内容 -->
            <view class="cart-content">
                <!-- 购物车列表 -->
                <shopCart :list="list" :height="scrollViewHeight" :triggered="triggered"
                          @onRefresh="onRefresh" @delect="delectCart" @update="updateCart" @accounts="accounts" @deleteCart="deleteCartByIndex"/>
            </view>
        </view>

        <!-- 底部导航 -->
        <Tabbar :tabsId="3"/>

        <!-- 登录弹窗 -->
        <loginPop :show="showLoginDialog" @close="hideLogin" @success="loginSuccess"/>

        <!-- 提示信息 -->
        <u-toast ref="toast" style="z-index: 9999"></u-toast>
    </pageWrapper>
</template>

<script>
import Tabbar from '@/components/tabbar/tabbar.vue'
import shopCart from '@/components/shopCart/shopCart.vue'
import loginPop from '@/components/loginPop/loginPop.vue'
import {getToken} from '@/store/storage.js'
import {getCartList} from '@/api/cart';
import config from '@/config/config.js'

export default {
    components: {
        Tabbar,
        loginPop,
        shopCart
    },
    data() {
        return {
            token: '',
            tabsIndex: 0,
            list: [], // 购物车列表
            showLoginDialog: false, // 登录
            scrollViewHeight: 0,
            triggered: false,
            toLoginStyle: {
                width: '160px',
                height: '48px',
                borderRadius: '24px',
                background: '#3c9cff',
                border: 'none',
                color: '#ffffff',
                fontSize: '16px',
                fontWeight: '500',
            },
            tabsList: [{
                name: '购物车',
            },
                // {
                //     name: '快速下单',
                // }
            ]
        };
    },
    onLoad() {
        this.token = getToken()
    },
    onShow() {
        // 登录的情况下获取
        if (this.token) {
            this.getCartListData()
        }
    },
    mounted() {
        // 设置商品列表高度，确保不被底部tabbar遮挡
        uni.getSystemInfo({
            success: (res) => {
                const windowHeight = res.windowHeight;
                const statusBarHeight = res.statusBarHeight || 0;
                // 减去状态栏、固定标题高度、底部结算栏高度和tabbar高度
                // 标题高度约70px，底部结算栏约50px，tabbar约50px
                this.scrollViewHeight = windowHeight - statusBarHeight - 70 - 50;
                console.log('计算后的scroll-view高度:', this.scrollViewHeight);
            },
        });
    },
    methods: {
        // 更新购物车解决微信小程序选中问题
        updateCart(list) {
            this.list = list
        },
        deleteCartByIndex(index) {
            const l = JSON.parse(JSON.stringify(this.list))
            l.splice(index, 1)
            this.list = l
        },
        async getCartListData() {
            const res = await getCartList(this.$refs.toast)
            // 授权过期
            if (res.code === 401) {
                this.token = ''
                return false
            }
            res.data.list.forEach(item => {
                if (item.goods.images && item.goods.images.length > 0 && item.goods.images[0].url.slice(0, 4) !== 'http') {
                    item.goods.images[0].url = config.baseUrl + "/" + item.goods.images[0].url
                }
            })
            this.list = res.data.list
            console.log(this.list)
            return true
        },
        // 登录成功
        loginSuccess(u) {
            this.hideLogin();
            this.token = getToken()
            this.$message(this.$refs.toast).success("登录成功")
            this.getCartListData()
        },
        // 显示登录框
        showLogin() {
            this.showLoginDialog = true
        },
        // 隐藏登录框
        hideLogin() {
            this.showLoginDialog = false
        },
        tabsChange(e) {
            this.tabsIndex = e.index
        },
        delectCart(e) {
            console.log('delectCart', e)
        },
        accounts(e) {
            console.log('accounts', e);
        },
        // 刷新
        async onRefresh() {
            this.triggered = true;
            const b = await this.getCartListData()
            if (b) {
                this.$message(this.$refs.toast).success('刷新成功')
            } else {
                this.$message(this.$refs.toast).error('刷新失败')
            }
            this.triggered = false;
        },
    }
}
</script>

<style lang="scss">
// 页面基础样式
page {
  background: #f8f9fa;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

// 登录提示容器
.login-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: 70vh;
  padding: 20px;

  .login-card {
    background: #ffffff;
    border-radius: 20px;
    padding: 40px 30px;
    box-shadow: 0 8px 32px rgba(0, 0, 0, 0.1);
    text-align: center;
    max-width: 320px;
    width: 100%;

    .login-icon {
      margin-bottom: 24px;
      animation: float 3s ease-in-out infinite;
    }

    .login-text {
      margin-bottom: 32px;

      .login-title {
        display: block;
        font-size: 20px;
        font-weight: 600;
        color: #2c3e50;
        margin-bottom: 8px;
      }

      .login-subtitle {
        display: block;
        font-size: 14px;
        color: #7f8c8d;
        line-height: 1.5;
      }
    }

    .login-button {
      animation: slideUp 0.6s ease-out;
    }
  }
}

// 购物车容器
.cart-container {
  display: flex;
  flex-direction: column;
  background: #f8f9fa;

  // 页面标题 - 固定定位
  .page-header {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    background: #ffffff;
    padding: 10px 16px;
    display: flex;
    justify-content: space-between;
    align-items: center;
    box-shadow: 0 1px 4px rgba(0, 0, 0, 0.05);
    z-index: 100;

    .page-title {
      font-size: 20px;
      font-weight: 600;
      color: #2c3e50;
    }

    .cart-count {
      background: #3c9cff;
      color: #ffffff;
      padding: 4px 10px;
      border-radius: 12px;
      font-size: 11px;
      font-weight: 500;
    }
  }

  // 购物车内容
  .cart-content {
    flex: 1;
    padding-top: 50px; // 为固定标题留出空间
    padding-bottom: 70px;
  }
}

// 动画效果
@keyframes float {
  0%, 100% { transform: translateY(0px); }
  50% { transform: translateY(-10px); }
}

@keyframes slideUp {
  from {
    opacity: 0;
    transform: translateY(20px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

// 响应式设计
@media (max-width: 375px) {
  .login-container {
    .login-card {
      padding: 30px 20px;

      .login-text {
        .login-title {
          font-size: 18px;
        }

        .login-subtitle {
          font-size: 13px;
        }
      }
    }
  }

  .cart-container {
    .page-header {
      padding: 10px 12px;

      .page-title {
        font-size: 20px;
      }
    }

    .cart-content {
      padding: 12px;
    }
  }
}
</style>
