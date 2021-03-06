<template>
  <div>
    <div class="shopcart">
      <div class="content" @click="toggleList()">
        <div class="content-left">
          <div class="logo-wrapper">
            <div class="logo" :class="{'highlight':totalCount>0}">
              <i class="icon-shopping_cart" :class="{'highlight':totalCount>0}"></i>
            </div>
            <div class="num" v-show="totalCount>0">{{totalCount}}</div>
          </div>
          <div class="price" :class="{'highlight':totalCount>0}">¥{{totalPrice}}</div>
          <div class="desc">另需配送费¥{{deliveryPrice}}元</div>
        </div>
        <div class="content-right" @click.stop.prevent="pay">
          <div class="pay" :class="payClass">
            {{payDesc}}
          </div>
        </div>
      </div>
      <div class="ball-container">
        <!--eslint-disable-next-line-->
        <div v-for="ball in balls">
          <transition name="drop" @before-enter="beforeDrop" @enter="dropping" @after-enter="afterDrop">
            <div class="ball" v-show="ball.show">
              <div class="inner inner-hook"></div>
            </div>
          </transition>
        </div>
      </div>
      <transition name="fold">
        <div class="shopcart-list" v-show="listShow">
          <div class="list-header">
            <h1 class="title">购物车</h1>
            <span class="empty" @click="empty">清空</span>
          </div>
          <div class="list-content" ref="listcontent">
            <ul>
              <!--eslint-disable-next-line-->
              <li class="food" v-for="food in selectFoods">
                <span class="name">{{food.name}}</span>
                <div class="price">
                  <span>¥{{food.price*food.count}}</span>
                </div>
                <div class="cartcontrol-wrapper">
                  <cartcontrol :food="food"></cartcontrol>
                </div>
              </li>
            </ul>
          </div>
        </div>
      </transition>
    </div>
    <transition name="fade">
      <div class="list-mask" v-show="listShow" @click="hideList"></div>
    </transition>
  </div>

</template>

<script type="text/ecmascript-6">
  import BScroll from 'better-scroll';
  import cartcontrol from 'components/cartcontrol/cartcontrol';
  export default {
    props: {
      selectFoods: {
        type: Array,
        default() {
          return [
            // {
            //   price: 10,
            //   count: 1
            // }
          ];
        }
      },
      deliveryPrice: {
        type: Number,
        default: 0
      },
      minPrice: {
        type: Number,
        default: 0
      }
    },
    data() {
      return {
        balls: [
          {
            show: false
          },
          {
            show: false
          },
          {
            show: false
          },
          {
            show: false
          },
          {
            show: false
          }
        ],
        dropBalls: [],
        fold: true
      };
    },
    computed: {
      totalPrice() {
        let total = 0;
        this.selectFoods.forEach((food) => {
          total += food.price * food.count;
        });
        return total;
      },
      totalCount() {
        let count = 0;
        this.selectFoods.forEach((food) => {
          count += food.count;
        });
        return count;
      },
      payDesc() {
        if (this.totalPrice === 0) {
          return `¥${this.minPrice}元起送`;
        } else if (this.totalPrice < this.minPrice) {
          let diff = this.minPrice - this.totalPrice;
          return `还差¥${diff}元起送`;
        } else {
          return `去结算`;
        }
      },
      payClass() {
        if (this.totalPrice < this.minPrice) {
          return 'not-enough';
        } else {
          return 'enough';
        }
      },
      listShow() {
        var _this = this;
        if (!this.totalCount) {
          _this.fold = true;
          return false;
        }
        let show = !this.fold;
        if (show) {
          this.$nextTick(() => {
            if (!this.scroll) {
              _this.scroll = new BScroll(this.$refs.listcontent, {
                click: true
              });
            } else {
              this.scroll.refresh();
            }
          });
        }
        return show;
      }
    },
    methods: {
      drop(el) {
        // console.log(el);
        for (let i = 0; i < this.balls.length; i++) {
          let ball = this.balls[i];
          if (!ball.show) {
            ball.show = true;
            ball.el = el;
            this.dropBalls.push(ball);
            return;
          }
        }
      },
      beforeDrop(el) {
        let count = this.balls.length;
        while (count--) {
          let ball = this.balls[count];
          if (ball.show) {
            let rect = ball.el.getBoundingClientRect();
            let x = rect.left - 32;
            let y = -(window.innerHeight - rect.top - 22);
            el.style.display = '';
            el.style.webkitTransform = `translate3d(0,${y}px,0)`;
            el.style.transform = `translate3d(0,${y}px,0)`;
            let inner = el.getElementsByClassName('inner-hook')[0];
            inner.style.webkitTransform = `translate3d(${x}px,0,0)`;
            inner.style.transform = `translate3d(${x}px,0,0)`;
          }
        }
      },
      dropping(el, done) {
        /* eslint-disable no-unused-vars */
        let rf = el.offsetHeight;
        this.$nextTick(() => {
          el.style.display = '';
          el.style.webkitTransform = 'translate3d(0,0,0)';
          el.style.transform = 'translate3d(0,0,0)';
          let inner = el.getElementsByClassName('inner-hook')[0];
          inner.style.webkitTransform = 'translate3d(0,0,0)';
          inner.style.transform = 'translate3d(0,0,0)';
          el.addEventListener('transitionend', done);
        });
      },
      afterDrop(el) {
        let ball = this.dropBalls.shift();
        if (ball) {
          ball.show = false;
          el.style.display = 'none';
        }
      },
      toggleList() {
        if (!this.totalCount) {
          return;
        }
        this.fold = !this.fold;
      },
      empty() {
        this.selectFoods.forEach((food) => {
          food.count = 0;
        });
      },
      hideList() {
        this.fold = true;
      },
      pay() {
        if (this.totalPrice < this.minPrice) {
          return;
        }
        window.alert(`支付${this.totalPrice}元`);
      }
    },
    components: {
      cartcontrol
    }
  };
</script>

<style lang="less" rel="stylesheet/less">
  @import "../../common/less/mixin";
  .shopcart{
    position: fixed;
    left: 0;
    bottom: 0;
    z-index: 50;
    width: 100%;
    height: 48/@num;
    .content{
      display: flex;
      background: #141d27;
      .content-left{
        flex: 1;
        font-size: 0;
        .logo-wrapper{
          display: inline-block;
          position: relative;
          top: -10/@num;
          margin: 0 12/@num;
          padding: 6/@num;
          width: 56/@num;
          height: 56/@num;
          box-sizing: border-box;
          vertical-align: top;
          border-radius: 50%;
          background: #141d27;
          .logo{
            width: 100%;
            height: 100%;
            border-radius: 50%;
            text-align: center;
            background: #2b343c;
            &.highlight{
              background: rgb(0,160,220);
            }
            .icon-shopping_cart{
              line-height: 44/@num;
              font-size: 24/@num;
              color: #80858a;
              &.highlight{
                color: #fff;
              }
            }
          }
          .num{
            position: absolute;
            top: 0;
            right: 0;
            width: 24/@num;
            height: 16/@num;
            line-height: 16/@num;
            text-align: center;
            border-radius: 16/@num;
            font-size: 9/@num;
            font-weight: 700;
            color: #fff;
            background: rgb(240,20,20);
            box-shadow: 0 4/@num 8px 0 rgba(0,0,0,.4);
          }
        }
        .price{
          display: inline-block;
          vertical-align: top;
          line-height: 24/@num;
          margin-top: 12/@num;
          padding-right: 12/@num;
          box-sizing: border-box;
          border-right: 1px solid rgba(255,255,255,.1);
          font-size: 16/@num;
          font-weight: 700;
          color: rgba(255,255,255,.4);
          &.highlight{
            color: #fff;
          }
        }
        .desc{
          display: inline-block;
          vertical-align: top;
          line-height: 24/@num;
          margin: 12/@num 0 0 12/@num;
          font-size: 10/@num;
          color: rgba(255,255,255,.4);
        }
      }
      .content-right{
        flex: 0 0 105/@num;
        width: 105/@num;
        .pay{
          height: 48/@num;
          line-height: 48/@num;
          text-align: center;
          font-size: 12/@num;
          font-weight: 700;
          color: rgba(255,255,255,.4);
          background: #2b333b;
          &.not-enough{
            background: #2b333b;
          }
          &.enough{
            background: #00b43c;
            color: #fff;
          }
        }
      }
    }
    .ball-container{
      .ball{
        position: fixed;
        left: 32/@num;
        bottom: 22/@num;
        z-index: 200;
        transition: all 0.4s cubic-bezier(0.49, -0.29, 0.75, 0.41);
        .inner{
          width: 16/@num;
          height: 16/@num;
          border-radius: 50%;
          background: rgb(0,160,220);
          transition: all .4s linear;
        }
      }
    }
    .shopcart-list{
      position: absolute;
      top:0;
      left: 0;
      z-index: -1;
      width: 100%;
      transform: translate3d(0,-100%,0);
      &.fold-enter-active,&.fold-leave-active{
        transition: all .5s;
      }
      &.fold-enter,&.fold-leave-active{
        transform: translate3d(0,0,0);
      }
      .list-header{
        height: 40/@num;
        line-height: 40/@num;
        padding: 0 18/@num;
        background: #f3f5f7;
        border-bottom: 1px solid rgba(7,17,27,.1);
        .title{
          float: left;
          font-size: 14/@num;
          color: rgb(7,17,27);
        }
        .empty{
          float: right;
          font-size: 12/@num;
          color: rgb(0,160,220);
        }
      }
      .list-content{
        padding: 0 18/@num;
        max-height: 217/@num;
        background: #fff;
        overflow: hidden;
        .food{
          position: relative;
          padding: 10/@num 0;
          box-sizing: border-box;
          .border-1px(rgba(7,17,27,.1));
          .name{
            line-height: 24/@num;
            font-size: 14/@num;
            color: rgb(7,17,27);
            vertical-align: middle;
          }
          .price{
            position: absolute;
            right: 90/@num;
            bottom: 16/@num;
            line-height: 24/@num;
            font-size: 14/@num;
            font-weight: 700;
            color: rgb(240,20,20);
          }
          .cartcontrol-wrapper{
            position: absolute;
            right: 0;
            bottom: 10/@num;
          }
        }
      }
    }
  }
  .list-mask{
    position: fixed;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    z-index: 40;
    opacity: 1;
    backdrop-filter:blur(10px);
    background: rgba(7,17,27,.6);
    &.fade-enter-active,&.fade-leave-active{
      transition: all .5s;
    }
    &.fade-enter,&.fade-leave-active{
      opacity: 0;
      background: rgba(7,17,27,0);
    }
  }
</style>
