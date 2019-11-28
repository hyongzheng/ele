<template>
<!-- 商品推荐'加号'的加减操作框 -->
  <div class="cartcontroll">
    <span @click.stop="decreaseCount" class="cartbutton button-minus" v-if="food.count>0">
      <i class="fa fa-minus"></i>
    </span>
    <span class="cartcount" v-if="food.count>0">{{food.count}}</span>
    <span @click.stop="increaseCount" class="cartbutton">
      <i class="fa fa-plus-circle"></i>
    </span>
  </div>
</template>

<script>
export default {
  name: "CartControll",
  props: {
    food: Object
  },
  methods: {
    // 我觉得这两个方法不应该直接修改数据，应该用emit提交到父级修改
    decreaseCount() {
      this.food.count--;
    },
    increaseCount() {
      this.food.count++;
    }
  }
};
</script>

<style scoped>
.cartcontroll {
  display: inline-flex;
  align-items: center;
}
.cartbutton {
  display: inline-block;
  vertical-align: middle;
}
.cartbutton i {
  color: rgb(35, 149, 255);
  vertical-align: middle;
  font-size: 1.5rem;
}
.button-minus {
  width: 4.866667vw;
  height: 4.866667vw;
  border: 1px solid rgb(35, 149, 255);
  border-radius: 50%;
  text-align: center;
}
.button-minus i {
  font-size: 1rem;
}
.cartcount {
  display: inline-block;
  text-align: center;
  color: rgba(0, 0, 0, 0.87);
  vertical-align: middle;
  font-size: 0.8rem;
  width: 6.933333vw;
  overflow: hidden;
}
</style>
