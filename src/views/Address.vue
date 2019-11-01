<template>
  <div class="address">
    <!-- 带返回的头部栏 -->
    <Header title="选择收货地址" :isLeft="true"></Header>

    <!-- 定位检索框 -->
    <div class="city_search">
      <div class="search">
        <span class="city" @click="$router.push('/city')">
          {{city}}
          <!-- 这是一个下箭头 -->
          <i class="fa fa-angle-down"></i>
        </span>
        <!-- 这是一个搜索镜 -->
        <i class="fa fa-search"></i>
        <input type="text" v-model="search_val" placeholder="小区/写字楼/学校等" />
      </div>
      <!-- 地址下拉显示栏 -->
      <!-- <p>{{search_val}}</p> -->
      <Location :address="address" />
    </div>
    <div class="area">
      <ul class="area_list" v-for="(item,index) in areaList" :key="index">
        <li @click="selectAddress(item.district,item.address,item.name)">
          <h4>{{item.name}}</h4>
          <!-- ！把空数组过滤掉 -->
          <p>{{item.district}}{{item.address.toString()}}</p>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
import Header from '../components/Header';
import Location from '../components/Location';
export default {
  data () {
    return {
      city: this.$route.params.city || '广州', // 当前的城市 this.$route.params.city 这个方法有弊端。
      search_val: "",
      areaList: []
    }
  },
  created() {
    console.log(this);
    // if(!localStorage.getItem("mycity")){
    //   console.log(33)
    //   localStorage.setItem("mycity",this.$route.params.city);
    // }
  },
  methods: {
    searchPlace() {
      const self = this;
      // console.log(this.search_val);
      AMap.plugin("AMap.Autocomplete", function() {
        // 实例化Autocomplete
        var autoOptions = {
          //city 限定城市，默认全国
          city: self.city
        };
        var autoComplete = new AMap.Autocomplete(autoOptions);
        autoComplete.search(self.search_val, function(status, result) {
          // 搜索成功时，result即是对应的匹配数据
          console.log(result);
          self.areaList = result.tips;
        });
      });
    },
    selectAddress(district,address,name){
      this.$store.dispatch("setAddress", district+address.toString()+name);
      this.$router.push("/home");
    }
  },
  computed: {
    address(){
      return this.$store.getters.location.formattedAddress;
    }
  },
  watch: {
    search_val(){
      this.searchPlace();
    }
  },
  components:{
    Header,
    Location
  },

  // beforeRouteEnter(to, from, next){
  //   next(vm =>{
  //     vm.city = to.params.city;
  //   })
  // }
}

</script>
<style scoped>
.address {
  width: 100%;
  height: 100%;
  overflow: auto;
  box-sizing: border-box;
  padding-top: 45px;
}

.city_search {
  background-color: #fff;
  padding: 10px 20px;
  color: #333;
}

.search {
  background-color: #eee;
  height: 40px;
  border-radius: 10px;
  box-sizing: border-box;
  line-height: 40px;
}
.search .city {
  padding: 0 10px;
}
.city i {
  margin-right: 10px;
}
.search input {
  margin-left: 5px;
  background-color: #eee;
  outline: none;
  border: none;
}

.area {
  margin-top: 16px;
  background: #fff;
}
.area li {
  border-bottom: 1px solid #eee;
  padding: 8px 16px;
  color: #aaa;
}
.area li h4 {
  font-weight: bold;
  color: #333;
  margin-bottom: 5px;
}
</style>
