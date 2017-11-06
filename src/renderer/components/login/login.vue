<template>
  <div class="login-wrap">
    <div class="top">
      <span class="img"></span>
      <span class="name">{{name}}</span>
    </div>
    <div class="middle">
      
    </div>
    <div class="foot" @click="close">
      关闭
    </div>
  </div>
</template>

<script>
  export default {
    name: 'Login',
    data () {
      return {
        name: '自来也',
      }
    },
    methods: {
      close () {
        this.$emit('closeLogin')
      }
    }
  }
</script>

<style scoped>
.login-wrap{   
  position: absolute;
  top: 0;
  left: 60%;
  width: 40%;
  height: 100%;  
  background-color: #fff;
}
.top{
  height: 50px;
  background-color: deepskyblue;
  display: flex;
  justify-content: space-around;
}
.top span {
  height: 50px;
  width: 50px;
}
.img {
  background: url('./ccc.jpg') no-repeat center;
  background-size: 50px 50px;
  border-radius: 50%;
}
.name{
  line-height: 50px;
  text-align: center;
  color: white; 
}
.foot {
  position: absolute;
  left: 0;
  bottom: 0;
  width: 100%;
  height: 50px;
  background-color: dodgerblue;
  line-height: 50px;
  color: white;
  text-align: center;
}
</style>