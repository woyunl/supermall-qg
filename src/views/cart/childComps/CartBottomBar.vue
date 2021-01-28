<template>
  <div class="bottom-bar">
    <div class="check-content">
      <cart-check-button class="check-button" :is-checked="isSelectAll" @click.native="CheckClick"/>
      <span>全选</span>
    </div>
    <div class="price">
      合计：{{totalPrice}}
    </div>
    <div class="calculate" @click="calcClick">
      去计算({{checkLength}})
    </div>
  </div>
</template>

<script>
    import CartCheckButton from "./CartCheckButton";
    export default {
        name: "CartBottomBar",
        components:{
          CartCheckButton
        },
        computed:{
          totalPrice(){
            return '￥' + this.$store.state.cartList.filter(item => {
              return item.checked
            }).reduce((preValue,item) => {
              return preValue + item.price * item.count
            },0).toFixed(2)
          },
          checkLength() {
            return this.$store.state.cartList.filter(item => item.checked).length
          },
          isSelectAll(){
            if (this.$store.state.cartList.length === 0 ) return false
            for(let item of this.$store.state.cartList){
              if(!item.checked){
                return false
              }
            }
            return true
          }
        },
        methods:{
          CheckClick(){
             if(this.isSelectAll){
               this.$store.state.cartList.forEach(item => item.checked = false)
             }else{
               this.$store.state.cartList.forEach(item => item.checked = true)
             }
          },
          calcClick(){
            if(!this.isSelectAll){
              this.$toast.show('请选择购买的商品',2000)
            }
          }
        }
    }
</script>

<style scoped>
  .bottom-bar {
    height: 38px;
    line-height: 40px;
    background-color: #eee;
    position: relative;
    display: flex;
    font-size: 13px;
  }
  .check-content {
    display: flex;
    align-items: center;
    margin-left: 10px;
    width:60px;
  }
  .check-button {
    width: 20px;
    height: 20px;
    line-height: 20px;
    margin-right: 5px;
  }
  .price {
    margin-left: 20px;
    flex: 1;
  }
  .calculate{
    width: 90px;
    background: red;
    color:#fff;
    text-align: center;
  }
  span{
    font-size: 10px;
  }
</style>
