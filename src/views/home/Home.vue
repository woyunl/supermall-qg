<template>
  <div id="home">
    <nav-bar class="home-nav"><div slot="center">购物街</div></nav-bar>
    <tab-control :titles="['流行','新款','精选']"
                 @tabClick="tabClick"
                 ref="tabControl1" class="tab-control" v-show="isTabFixed"/>
    <scroll class="content"
            ref="scroll"
            :probe-type="3"
            @scroll="contentScroll"
            :pull-up-load="true"
            @pullingUp="loadMore">
      <home-swiper :banners="banners" @swiperImageLoad="swiperImageLoad"/>
      <RecommendView :recommends="recommends"/>
      <feature-view/>
      <tab-control :titles="['流行','新款','精选']"
                   @tabClick="tabClick"
                   ref="tabControl2"/>
      <goods-list :goods="showGoods"/>
    </scroll>
    <back-top @click.native = "backClick" v-show="isShowBackTop"/>

  </div>
</template>

<script>
    import HomeSwiper from './childComps/HomeSwiper'
    import RecommendView from "./childComps/RecommendView";
    import FeatureView from "./childComps/FeatureView";

    import NavBar from "components/common/navbar/NavBar";
    import TabControl from "components/contents/tabControl/TabControl";
    import GoodsList from 'components/contents/goods/GoodsList';
    import Scroll from "../../components/common/scroll/Scroll";
    import BackTop from "components/contents/backTop/BackTop";

    import { getHomeMultidata, getHomeGoods } from "network/home";


    export default {
        name: "Home",
        components:{
          RecommendView,
            NavBar,
            HomeSwiper,
            FeatureView,
            TabControl,
            GoodsList,
            Scroll,
            BackTop
        },
        data() {
          return {
            banners:[],
            recommends:[],
            goods:{
              'pop':{page:0,list:[]},
              'new':{page:0,list:[]},
              'sell':{page:0,list:[]},
            },
            currentType:'pop',
            isShowBackTop:false,
            tabOffsetTop:0,
            isTabFixed: false,
            saveY:0
          }
        },
        computed: {
          showGoods() {
            return this.goods[this.currentType].list
          }
        },
        destroyed(){
          console.log('home destroyed');
        },
        activated(){
          this.$refs.scroll.scrollTo(0,this.saveY,0)
          this.$refs.scroll.refresh()
        },
        deactivated(){
          this.saveY = this.$refs.scroll.getScrollY()
        },
        created() {
          this.getHomeMultidata()
          this.getHomeGoods('pop')
          this.getHomeGoods('new')
          this.getHomeGoods('sell')

        },
        mounted() {
          //1.图片加载完成的事件监听
          const refresh = this.debounce(this.$refs.scroll.refresh)
          this.$bus.$on('itemImageLoad',() => {
            refresh()
          })
        },
      methods:{

        /**
         * 事件监听的方法
         */
        debounce(func,delay){
          let timer = null
          return function (...args){
            if (timer) clearTimeout(timer)
            timer = setTimeout(() => {
              func.apply(this,args)
            },delay)
          }
        },

        tabClick(index){
          switch (index) {
             case 0:
               this.currentType = 'pop'
               break
             case 1:
               this.currentType = 'new'
               break
             case 2:
               this.currentType = 'sell'
               break
          }
          this.$refs.tabControl1.currentIndex = index;
          this.$refs.tabControl2.currentIndex = index;
        },
        backClick (){
          this.$refs.scroll.scrollTo(0,0)
        },
        contentScroll(position) {
          //1.判断BackTop是否显示
          this.isShowBackTop = (-position.y) > 1000

          //2.决定tabControl是否吸顶(position:fixed)
          this.isTabFixed = (-position.y) > this.tabOffsetTop

        },
        loadMore(){
            this.getHomeGoods(this.currentType)
          },
        swiperImageLoad(){
          this.tabOffsetTop = this.$refs.tabControl2.$el.offsetTop;
        },

        /**
         * 网络请求相关的方法
         */
        getHomeMultidata() {
            getHomeMultidata().then(res => {
              this.banners = res.data.banner.list;
              this.recommends = res.data.recommend.list;
            })
          },
          getHomeGoods(type){
            const page = this.goods[type].page +1
            getHomeGoods(type,page).then(res => {
              this.goods[type].list.push(...res.data.list)
              this.goods[type].page += 1

              this.$refs.scroll.finishPullUp()
            })
          }
      }
    }
</script>

<style scoped>
  #home{
    height: 100vh;
    position: relative;
  }

  .home-nav{
    background-color: var(--color-tint);
    color:#fff;

   /* position: fixed;
    left: 0;
    right:0;
    top: 0;
    z-index: 9;*/

  }

  .content {
    overflow: hidden;
    position: absolute;
    top: 44px;
    bottom: 49px;
    left: 0;
    right: 0;
  }

  .tab-control{
    position: relative;
    z-index: 9;
  }

</style>
