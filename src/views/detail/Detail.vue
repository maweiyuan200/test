<template>
  <div id="Details">
    <!-- 头部导航 -->
    <detail-tittle-navbar class="top-details"
                          @titleClick='titleClick'
                          ref="topNavBar"/>
    <!-- 滚动模块 -->
    <scroll
        class="d-wrapper"
        ref="scroll"
        @scroll="contentScroll"
        :probe-type="3">
      <!-- 轮播图组件 -->
      <detail-swiper :top-images='topImages'></detail-swiper>
      <!-- 基本信息组件 -->
      <detail-base-info :base-info='baseInfo'></detail-base-info>
      <!-- 店铺信息组件 -->
      <detail-shop-info :shop-info='shopInfo'></detail-shop-info>
      <!-- 商品图片组件 -->
      <detail-goods-info
          :goods-info='goodsInfo'
          @imageLoadOk='imageLoadOk'>
      </detail-goods-info>
      <!-- 商品尺寸大小 -->
      <detail-size
          :size-info='sizeInfo'
          ref="Size"></detail-size>
      <!-- 评论区域组件 -->
      <detail-comment-info
          :comment-info='commentInfo'
          ref="Comment1"></detail-comment-info>
      <!-- 推荐商品数据列表 -->
      <goods-list
          :goods='goodsInfoList'
          :is-detail-msg='isDetailMsg'
          ref="Recommend"
          class="goods-list"></goods-list>
    </scroll>
    <!-- .native实现监听组件内原生事件 -->
    <back-top @click.native="backClick" v-show="isShowBackTop"/>
    <!-- 底部工具栏 -->
    <detail-bottom-tab
        @addShopCar='addShopCar'
        @buyNow='buyNow'></detail-bottom-tab>
    <!-- toast组件 // 如果使用插件，这个就不需要了-->
    <!--
    <toast
        :message='msg'
        :show='show' />
    -->
  </div>
</template>

<script>
import DetailTittleNavbar from "./childComp/DetailTittleNavbar";
import {getDetails,BaseInfo,ShopInfo,SizeInfo,getDetailsGoodsList} from 'network/details'
// 轮播图组件
import DetailSwiper from './childComp/DetailSwiper'
// 基本信息组件
import DetailBaseInfo from './childComp/DetailBaseInfo'
// 店铺信息组件
import DetailShopInfo from './childComp/DetailShopInfo'
// 商品图片组件
import DetailGoodsInfo from './childComp/DetailGoodsInfo'
// 商品尺寸组件
import DetailSize from './childComp/DetailSize'
// 评论信息组件
import DetailCommentInfo from './childComp/DetailCommentInfo'
// 引入推荐商品数据列表模块
import GoodsList from 'components/content/goods/GoodsList'
// 引入底部导航模块
import DetailBottomTab from './childComp/DetailBottomTab'
// 导入滚动模块
import Scroll from 'components/common/scroll/Scroll'
// 返回首页导航按钮
import BackTop from 'components/content/backtop/BackTop'
// 导入混入Mixin
import {MixIn,backTopMixIn} from 'common/mixin'
// 导入变量名
import {ADDCART} from 'store/mutations-types'
// 导入vuex中的actions属性，将actions中的函数映射到组件中
import {mapActions} from 'vuex'

export default {
  name: "Detail",
  components: {
    DetailTittleNavbar,
    DetailSwiper,
    DetailBaseInfo,
    DetailShopInfo,
    DetailGoodsInfo,
    DetailSize,
    DetailCommentInfo,
    GoodsList,
    DetailBottomTab,
    BackTop,
    Scroll,
    //Toast   //自己封装的toast组件 ,引用插件的话就不用这个了
  },
  data() {
    return {
      iid: null,
      topImages: [],  //轮播图数据
      baseInfo: {},   //基本信息保存
      shopInfo: {},    //店铺信息保存
      goodsInfo: {},   //商品图片等信息
      sizeInfo: {},   // 保存商品尺寸信息
      commentInfo: {},   //品论数据保存
      goodsInfoList: [],  //推荐商品数据
      isDetailMsg: true,  //因为我们推荐数据的组件是用同一个的，所以用来区分数据
      themeTops: [],   //保存头部导航栏对应到的offsetTop值
      currentIndex: 0,   //防止滚动那里重复刷数据
      // isShowBackTop: false,  //默认设置是看不见的
      /*
      msg: '',   //保存toast数据
      show: false  //是否显示toast
      */
    }
  },
  created() {
    this.iid = this.$route.params.iid
    // 1、显示对应的数据
    getDetails(this.iid).then(res => {
      const data = res.result;
      //  2、保存详细里面的数据
      this.topImages = data.itemInfo.topImages


      // 3、保存基本信息数据
      // 重点二、把这个对象new出来，同时传递参数
      this.baseInfo = new BaseInfo(data.itemInfo,data.shopInfo.services,data.columns)
      // 4、保存店铺信息数据
      this.shopInfo = new ShopInfo(data.shopInfo);
      // 5、商品图片等信息
      this.goodsInfo = data.detailInfo;
      // 6、商品尺寸信息
      this.sizeInfo = new SizeInfo(data.itemParams.info,data.itemParams.rule);
      // 7、评论信息
      // 判断是否有评论，因为存在没有评论的情况
      if (data.rate.cRate !== 0) this.commentInfo = data.rate;
      /*this.$nextTick(() => {
         //$nextTick是等该组件创造完后再执行一个回调函数，
         //但是这个只是检测到你dom渲染出来就执行，而图片数据不一定渲染完了，所以高度还是有问题，所以用updated更新
         // 第二种方法就是再updated函数里写
         this.themeTops.push(0);
         this.themeTops.push(this.$refs.Size.$el.offsetTop);
         this.themeTops.push(this.$refs.comment.$el.offsetTop);
         this.themeTops.push(this.$refs.recommend.$el.offsetTop);
         console.log(this.themeTops)
       })*/
    }),
        // 推荐其它商品列表
        getDetailsGoodsList().then(res => {
          //  获取商品列表数据
          // console.log(res)
          this.goodsInfoList = res.data.list;
        })
  },
  // 不使用混入时
  /*mounted () {
     const refresh = debounce(this.$refs.scroll.refresh,500);  //刷新，让滚动框架重新计算滚动区域
    // 事件总线，接收传递过来的事件
    this.$bus.$on('imgLoadOk', () => {
        refresh();// 防抖动
    })
  },*/

  // 使用混入时
  mixins: [MixIn,backTopMixIn],
  methods: {
    ...mapActions({  //使用对象的形式映射进来
      'addCart': ADDCART
    }),
    imageLoadOk () {
      // 刷新
      this.newRefresh()
      // 获取高度，给我们头部导航栏定位锚点用，在这里监听是最好的，因为刷新完后，高度获取绝对正确
      this.themeTops.push(0);
      this.themeTops.push(this.$refs.Size.$el.offsetTop);
      this.themeTops.push(this.$refs.Comment1.$el.offsetTop);
      this.themeTops.push(this.$refs.Recommend.$el.offsetTop);
      // 空间换时间,加入最大值
      this.themeTops.push(Number.MAX_VALUE)
    },
    titleClick (index) { //头部导航栏
      this.$refs.scroll.scrollTo(0, -this.themeTops[index], 500)
    },

    contentScroll(position) {
      // 获取y值，为了方便我们对比。所以获取正值
      const positionY = -position.y
      // 保存数组长度
      let length = this.themeTops.length;
      // 滚动，这个值与themeTops对比
      for (let i = 0; i < length - 1; i ++) {  //获取的i是字符串
        if (this.currentIndex !== i && (positionY >= this.themeTops[i] && positionY < this.themeTops[i + 1])) {
          this.currentIndex = i;// 防止赋值过程过于频繁
          this.$refs.topNavBar.currentIndex = this.currentIndex;
        }
      }
      // 当达到1000时将按钮显示出来,注意，position.y是负数
      this.listenerBackTopShow(position)
      //  console.log(position.y)
    },

    // 加入购物车
    addShopCar () {
      // 1、先获取数据
      const product = {};
      product.title = this.baseInfo.title
      product.price = this.baseInfo.NowPrice
      product.desc = this.baseInfo.desc
      product.image = this.topImages[0]
      product.iid = this.iid

      // 2、发送这个数据给vuex管理,共享
      // this.$store.dispatch(ADDCART,product).then(res => {
      //   console.log(res)  //vuex中的actions可以接收一个promise执行异步操作
      // })
      // 使用映射关系，直接将action中的方法映射到组件内，直接调用
      this.addCart(product).then(res => {
        /*// 如果使用插件，这个就不需要了
        this.msg = res
        this.show = true
        // 1.5s之后消失
        setTimeout(() => {
          this.show = false
          this.msg = ' '
        },1500)
        */
        this.$toast.show(res)
      })
    },
    buyNow () {
      console.log('立即购买')
    }
  }
}
</script>

<style scoped>
  #Details {
    overflow-x: hidden;
    height: 100vh;
  }
  .d-wrapper {
    height: calc(100% - 93px);
    position: relative;
    z-index: 100;
    background-color: white;
  }
  .goods-list {
    margin-top: 8px;
  }
  .top-details {
    position: relative;
    z-index: 101;
    background-color: white;
  }
</style>
