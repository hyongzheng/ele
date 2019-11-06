<template>
  <div class="search">
    <Header :isLeft="true" title="搜索" />
    <!-- 搜索框 -->
    <div class="search_header">
      <form class="search_wrap">
        <i class="fa fa-search"></i>
        <input type="text" v-model="key_word" placeholder="输入商家,商品信息" />
        <button @click.prevent="searchHandle">搜索</button>
      </form>
    </div>
    <!-- 搜索框下方显示 -->
    <div class="shop" v-if="result && !showShop">
      <!-- 如果搜索不到 -->
      <div class="empty_wrap" v-if="empty ">
        <img src="https://fuss10.elemecdn.com/d/60/70008646170d1f654e926a2aaa3afpng.png" alt />
        <div class="empty_txt">
          <h4>附近没有搜索结果</h4>
          <span>换个关键词试试吧</span>
        </div>
      </div>
      <!-- 搜索得到时 -->
      <div v-else>
        <SearchIndex @myclick="shopItemClick" :data="result.restaurants" />
        <SearchIndex @myclick="shopItemClick" :data="result.words" />
      </div>
    </div>
    <!-- 点击跳转商家信息 -->
    <div class="container" v-if="showShop">
      <FilterView :filterData="filterData" @update="update"></FilterView>
      <div class="shoplist" v-infinite-scroll="loadMore" :infinite-scroll-disabled="loading">
        <IndexShop v-for="(item,index) in restaurants" :key="index" :restaurant="item.restaurant"></IndexShop>
      </div>
    </div>
  </div>
</template>

<script>
import Header from "../components/Header";
import SearchIndex from "../components/SearchIndex";
import FilterView from "../components/FilterView";
import IndexShop from "../components/IndexShop";
import { InfiniteScroll } from "mint-ui";
export default {
  name: "Search",
  data() {
    return {
      key_word: "", //搜索框内容绑定
      result: null, //接受搜索结果
      empty: false,
      showShop: false,
      filterData: null,
      restaurants: [],
      page: 0,
      size: 7,
      loading: false,
      data: null
    };
  },
  watch: {
    key_word() {
      this.empty = false; //将‘搜索为空’清除
      this.showShop = false; //将‘搜索详情’清除
      this.keyWordChange();
    }
  },
  created() {
    this.$axios("/api/profile/filter").then(res => {
      // console.log(res.data);
      // 这是‘距离最近、筛选...’的标题栏信息
      this.filterData = res.data;
    });
  },
  methods: {
    keyWordChange() {
      // console.log(this.key_word);
      this.$axios(`/api/profile/typeahead/${this.key_word}`)
        .then(res => {
          console.log(res.data);
          this.result = res.data;
        })
        .catch(err => {
          this.result = null;
        });
    },
    searchHandle() {
      // 搜索内容为空时
      if (!this.key_word) return;
      // 搜索内容有显示时
      if (
        this.result &&
        (this.result.restaurants.length > 0 || this.result.words.length)
      ) {
        console.log("有内容");
        this.shopItemClick();
      } else {
        this.empty = true;
      }
    },
    shopItemClick() {
      // 再次点击搜索时，将先前的数据清空
      this.page = 0;
      this.restaurants = [];
      this.showShop = true;
      // console.log(" 点击了");
    },
    update(condation) {
      console.log(condation);
      this.data = condation;
      this.shopItemClick();
    },
    loadMore() {
      this.page++;
      this.$axios
        .post(`/api/profile/restaurants/${this.page}/${this.size}`, this.data)
        .then(res => {
          // this.restaurants = res.data;
          if (res.data.length > 0) {
            res.data.forEach(item => {
              this.restaurants.push(item);
            });
          } else {
            this.loading = true;
          }
        });
    }
  },
  components: {
    Header,
    SearchIndex,
    FilterView,
    IndexShop
  }
};
</script>

<style scoped>
.search {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
}
.search_header {
  margin-top: 45px;
  background: #fff;
  padding: 0 4.266667vw;
}
.search_header .search_wrap {
  padding: 2.933333vw 2.933333vw 2.933333vw 0;
  display: flex;
  align-items: center;
  position: relative;
}
.search_wrap .fa-search {
  width: 2.933333vw;
  height: 2.933333vw;
  color: #888;
  position: absolute;
  top: 4.6666666vw;
  left: 2.933333vw;
}
.search_wrap input {
  flex-grow: 1;
  border-radius: 0.266667vw;
  background-color: #f8f8f8;
  padding: 1.733333vw 4vw 1.733333vw 8.8vw;
  color: #666;
  outline: none;
  border: none;
}
.search_wrap button {
  outline: none;
  border: none;
  color: #333;
  font-size: 0.426667rem;
  background: #fff;
  font-weight: 700;
  margin-left: 2.933333vw;
  font-size: 14px;
}

.shop {
  width: 100%;
  height: calc(100% - 95px);
  overflow: auto;
}

.empty_wrap {
  width: 100%;
  height: 100%;
  overflow: hidden;
  background: #fff;
  display: flex;
  justify-content: center;
}
.empty_wrap img {
  width: 35vw;
  height: 35vw;
}
.empty_txt h4 {
  color: #666;
  font-size: 1rem;
  margin: 12vw 0 2vw 0;
}
.empty_txt span {
  color: #999;
  font-size: 0.8rem;
}
</style>
