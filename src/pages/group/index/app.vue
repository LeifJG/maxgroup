<template>
  <div id="app">
    <section class="main">
      <div class="top">
        <img src="./images/top.png" width="100%" alt="">
      </div>
      <div class="warp group">
        <div class="group_title">
          您想加入
        </div>
        <div class="group_warp">
          <div class="group_item" v-for="item in groupList" :style="{backgroundImage: 'url(' + item.groupPicture + ')'}" :key="item.id">
            <a :href="'/group/country.html?eId='+item.id" class="herf">
              <!-- <div class="group_bg" :style="{backgroundImage: 'url(' + item.groupPicture + ')'}"> </div> -->
              <div class="group_name centerbox">
                {{item.groupName}}
              </div>
            </a>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
  import logo from 'assets/img/logo.jpg'
  //import modal from 'components/modal.vue'
  import Lib from "assets/js/Lib";
  export default {
    components: {
      //modal
    },
    mounted() {
      Lib.M.baseDomFn();
    },
    data () {
      return {
        logoImg: logo,
        groupList:null,
      }
    },
    created () {
      Lib.WX.is_weixin('加入移民同路人群','与持牌移民专家、移民同路人、海外华人交流互动！',undefined,'https://oss.meiyi.ai/upload/image/201712/d4cc072f-3165-4d67-9ced-d7442b8e5705.png');
      Lib.M.baseFn();
      this.getGroupList();
    },
    methods: {
      getGroupList(){
        Lib.M.ajax({
          url:`/group_list`,
          type:'post',
          success:res=>{
            console.log(res);
            this.groupList=res;
          }
        });
      },

    }
  }
</script>

<style lang="less">
  @import "../../../assets/css/m_base.less";
  body{
    background: #F9F9F9;
    input::-webkit-outer-spin-button,
    input::-webkit-inner-spin-button{
      -webkit-appearance: none !important;
      margin: 0;
    }
  }
  .group_warp{
    margin-top: 0.36rem;
    display: flex;
    display: -webkit-flex;
    flex-wrap: wrap;
    -webkit-flex-wrap: wrap;
    margin-right: -0.26rem;
    .group_item{
      flex: 0 0 2.3rem;
      width: 2.3rem;
      height: 2.3rem;
      background-size: cover;
      border-radius: 6px;
      position: relative;
      font-weight: 600;
      overflow: hidden;
      margin: 0 0.15rem 0.15rem 0;
      .group_bg{
        width: 100%;
        filter: blur(10px);
        position: absolute;
        border-radius: 0 0 6px 6px;
        bottom: 0;
        height: 0.8rem;
        background-size: 2.86rem 3.22rem;
        opacity: 0.8;
        background-position: 100% 100%;
      }
      .group_name{
        position: absolute;
        bottom: 0;
        height: 100%;
        line-height: 2.3rem;
        width: 100%;
        text-align: center;
        color:#fff;
        background: rgba(0, 0, 0, 0.24);
        
      }
    }
  }
  .main{
    max-width: 768px;
    margin: 0 auto;
    font-size: .36rem;
    position: relative;
  }
  .warp{
    background: #FFFFFF;
    //box-shadow: 2px 2px 3px 0 rgba(176,176,176,0.50);
    //border-radius: 6px;
    //margin: 0 0.16rem;
    //margin-bottom: 0.16rem;
  }
  .herf{
    display: block;
    height: 100%;
  }
  .group{
    position: relative;
    padding:0.36rem .15rem;
  }
  .group_title{
    text-align: center;
    letter-spacing:1px;
    &:after{
      content:'';
      display: inline-block;
      position: relative;
      height: 1px;
      width: 1.3rem;
      top: -0.1rem;
      //background-image: linear-gradient(to left, rgba(55,194,255,0.19), rgba(53,193,255,1));
      background: #c9c9c9;
    }
    &:before{
      content:'';
      display: inline-block;
      position: relative;
      height: 1px;
      top: -0.1rem;
      width: 1.3rem;
      //background-image: linear-gradient(to right, rgba(55,194,255,0.19), rgba(53,193,255,1));
      background: #c9c9c9;
    }
  }
</style>
