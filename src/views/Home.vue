<template>
  <div class="home">
    <!-- 搜索栏头部 -->
    <div class="header">
      <!-- ！路由传递参数时可以这样配置，用name也可 -->
      <div class="address_map" @click="$router.push({name: 'address',params: {city: city}})">
        <i class="fa fa-map-marker"></i>
        <span>{{address}}</span>
        <i class="fa fa-sort-desc"></i>
      </div>
    </div>
    <!-- 商家搜索框 -->
    <div class="search_wrap" :class="{'fixedview': showFilter}"  @click="$router.push('./search')">
      <!-- :class="{'fixedview':true}" -->
      <div class="shop_search">
        <i class="fa fa-search"></i>
        搜索商家 商家名称
      </div>
    </div>
    <!-- 轮播图区域 -->
    <div id="container">
      <!-- auto的值表示几秒一轮播 -->
      <!-- 首页轮播 -->
      <mt-swipe :auto="4000" class="swiper">
        <mt-swipe-item v-for="(img,index) in swipeImgs" :key="index">
          <img :src="img" alt="图片未加载..." />
        </mt-swipe-item>
      </mt-swipe>
      <!-- 手动分类图轮播 -->
      <mt-swipe :auto="0" class="entries">
        <mt-swipe-item v-for="(entry,index) in entries" :key="index">
          <div class="foodentry" v-for="(item,index2) in entry" :key="index2">
            <div class="img_wrap">
              <img :src="item.image" alt="图片未加载..." />
            </div>
            <span>{{item.name}}</span>
          </div>
        </mt-swipe-item>
      </mt-swipe>
    </div>
    <!-- 推荐商家 -->
    <div class="shoplist-title">推荐商家</div>

    <!-- 综合排序导航栏 -->
    <FilterView :filterData="filterData" @searchFixed="showFilterView" @update="update"></FilterView>

    <!-- 商家信息 -->
    <mt-loadmore
      :top-method="loadData"
      :bottom-method="loadMore"
      :bottom-all-loaded="allLoaded"
      :auto-fill="false"
      :bottomPullText="bottomPullText"
      ref="loadmore"
    >
      <div class="shoplist">
        <IndexShop v-for="(item,index) in restaurants" :key="index" :restaurant="item.restaurant"></IndexShop>
      </div>
    </mt-loadmore>
  </div>
</template>

<script>
import { Swipe, SwipeItem, Loadmore } from "mint-ui";
import FilterView from "../components/FilterView";
import IndexShop from "../components/IndexShop";
export default {
  data() {
    return {
      swipeImgs: [],
      entries: [],
      filterData: null, //综合排序标题栏 信息
      showFilter: false, //搜索商家顶部提升控制符
      page: 1, //商家信息请求页数
      size: 5,
      restaurants: [], //商家信息
      allLoaded: false, //是否加载完
      bottomPullText: "上拉加载更多...", //上拉文字
      data: null //排序条件
    };
  },
  computed: {
    address() {
      return this.$store.getters.address;
    },
    city() {
      return (
        this.$store.getters.location.addressComponent.city ||
        this.$store.getters.location.addressComponent.province
      );
    }
  },
  created() {
    this.getData();
  },
  methods: {
    getData() {
      // 轮播图数据
      this.$axios("/api/profile/shopping").then(res => {
        // console.log(res.data);
        this.swipeImgs = res.data.swipeImgs;
        this.entries = res.data.entries;
      });
      // 排序和筛选信息
      this.$axios("/api/profile/filter").then(res => {
        // 打印出信息便于编程
        // console.log(res.data);
        this.filterData = res.data;
      });
      this.loadData();
      // 商家详情
      /* this.$axios.post(`/api/profile/restaurants/${this.page}/5`).then(res => {
        console.log(res.data);
        this.restaurants = res.data;
        // console.log(this.restaurants[1].restaurant);
      }); */
    },
    loadData() {
      // 下拉时重置数据
      this.page = 1;
      this.allLoaded = false;
      this.bottomPullText = "上拉加载更多";
      // 拉取商家信息
      this.$axios
        .post(`/api/profile/restaurants/${this.page}/${this.size}`,this.data)
        .then(res => {
          // console.log('---');
          console.log(res.data);
          this.$refs.loadmore.onTopLoaded();
          this.restaurants = res.data;
        });
    },
    // 下拉加载更多数据
    loadMore() {
      if (!this.allLoaded) {
        this.page++;
        // 拉取商家信息
        this.$axios
          .post(`/api/profile/restaurants/${this.page}/${this.size}`)
          .then(res => {
            //  加载完之后重新渲染
            this.$refs.loadmore.onBottomLoaded();
            if (res.data.length > 0) {
              res.data.forEach(item => {
                this.restaurants.push(item);
              });
              if (res.data < this.size) {
                this.allLoaded = true;
                this.bottomPullText = "没有更多了哦";
              }
            } else {
              // 数据为空
              this.allLoaded = true;
              this.bottomPullText = "没有更多了哦";
            }
          });
      }
    },

    // 搜索商家提升到顶部固定
    showFilterView(isShow) {
      this.showFilter = isShow;
    },
    update(condition) {
      // console.log(condition);
      this.data = condition;
      console.log(this.data);
      this.loadData();
      
    }
  },
  components: {
    FilterView,
    IndexShop
  }
};
</script>
<style scoped>
.home {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
}
.header,
.search_wrap {
  background-color: #009eef;
  padding: 10px 16px;
}
.header .address_map {
  color: #fff;
  font-weight: bold;
}
.address_map i {
  margin: 0 3px;
  font-size: 18px;
}
.address_map span {
  display: inline-block;
  width: 80%;
  overflow: hidden;
  white-space: nowrap;
  text-overflow: ellipsis;
}
.search_wrap .shop_search {
  /* margin-top: 10px; */
  background-color: #fff;
  padding: 10px 0;
  border-radius: 4px;
  text-align: center;
  color: #aaa;
}

.search_wrap {
  position: sticky;
  top: 0px;
  z-index: 999;
  box-sizing: border-box;
}
.swiper {
  height: 100px;
}
.swiper img {
  width: 100%;
  height: 100px;
}

.entries {
  background: #fff;
  height: 47.2vw;
  text-align: center;
  overflow: hidden;
}
.foodentry {
  width: 20%;
  float: left;
  position: relative;
  margin-top: 2.933333vw;
}
.foodentry .img_wrap {
  position: relative;
  display: inline-block;
  width: 12vw;
  height: 12vw;
}
.img_wrap img {
  width: 100%;
  height: 100%;
}
.foodentry span {
  display: block;
  color: #666;
  font-size: 0.32rem;
}
/* 推荐商家 */
.shoplist-title {
  display: flex;
  align-items: flex;
  justify-content: center;
  height: 9.6vw;
  line-height: 9.6vw;
  font-size: 16px;
  color: #333;
  background: #fff;
}
.shoplist-title:after,
.shoplist-title:before {
  display: block;
  content: "一";
  width: 5.333333vw;
  height: 0.266667vw;
  color: #999;
}
.shoplist-title:before {
  margin-right: 3.466667vw;
}
.shoplist-title:after {
  margin-left: 3.466667vw;
}

.fixedview {
  width: 100%;
  position: fixed;
  top: 0px;
  z-index: 999;
}

.mint-loadmore {
  height: calc(100% - 95px);
  overflow: auto;
}
/* 我改了active的原点颜色 */
.mint-swipe-indicator {
  background: #fff;
}
.mint-swipe-indicator is-active {
  background: #000;
}
</style>
