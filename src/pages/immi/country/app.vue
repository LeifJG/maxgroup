<template>
  <div id="app">
    <div class="main" v-if="details&&!resultCode">
      <div class="top">
        <img src="./images/top.png" alt="" width="100%">
        <div class="flag" :style="{backgroundImage: 'url(' + details.groupPicture + ')'}"></div>
      </div>
      <div class="warps">
        <div v-for="item in details.inviteQuestions" class="warp" :key="item.orders">
          <div v-if="item.type!='source'">
            <p class="question_title">{{item.orders}}.{{item.content}} <span class="error_tips" v-show="item.errTips">（{{item.errTips}}）</span></p>
            <div class="option_selelt_warp">
              <span class="icon-Shape"></span>
              <select v-model="item.selectedOption" class="option_selelt" :class="{'black':item.selectedOption}">
                <option :value="false">请选择{{item.content}}</option>
                <option v-for="option in item.inviteQuestionOptions" :value="option.optionValue" v-text="option.optionValue+'年'" :key="option.orders"></option>
              </select>
            </div>
          </div>
          <div v-else>
            <p class="question_title">{{item.orders}}.{{item.content}} <span class="error_tips" v-show="item.errTips">（{{item.errTips}}）</span></p>
            <div class="option_warp">
              <span class="option_item" :class="{'selected':item.selectedOptionIndex==optionItem.orders}" v-for="optionItem in item.inviteQuestionOptions" @click="selectOption(item,optionItem)" :key="optionItem.orders">{{optionItem.optionValue}}</span>
              <input type="text" name="other_text" placeholder="请补充说明" class="other_text" ref="other_text" v-if="item.selectedOption&&item.selectedOption.explanation" v-model="otherText">
            </div>
          </div>
        </div>
        <div class="warp input_warp">
          <p class="input_title">验证入圈</p>
          <p class="desc">为保证一个优质移民社交圈,所有成员均需验证入圈</p>
          <div class="input">
            <input type="tel" name="phone" v-model="phone" class="phone text" placeholder="请输入手机号码">
            <p class="error_tips" v-show="phoneErr">{{phoneErr}}</p>
            <div class="vcode_warp">
              <input type="tel" name="vcode" v-model="vcode" class="vcode text" placeholder="请输入验证码">
              <input type="button" name="getvcode" class="btn getvcode_btn" value="获取验证码" @click="getCode($event)">
            </div>
            <p class="error_tips" v-show="vcodeErr">{{vcodeErr}}</p>
          </div>
          <input type="button" name="" value="获取邀请码" class="sumbit_btn" @click="checkResult">
        </div>
      </div>
    </div>
    <div class="main result_warp" v-if="details&&resultCode">
      <div class="top">
        <img src="./images/top.png" alt="" width="100%">
        <div class="flag" :style="{backgroundImage: 'url(' + details.countryFlag + ')'}"></div>
      </div>
      <div class="warps">
        <div class="warp">
          <div class="tips">
            <p>您已成功递交入圈申请，还<span class="red">差2步</span>就可以和移民同路人交流!</p>
            <!-- <span class="icon-pass"></span> -->
          </div>
        </div>
        <div class="warp">
          <div class="step_warp">
            <div class="step">
              <div class="group_title">
                <span class="step_num">1</span>
                将带有验证码的本页面截图
              </div>
              <div class="code_warp">
                <p class="code">#{{resultCode}}#</p>
              </div>
              <div class="step_title">
                <span class="step_num">2</span>
                长按识别二维码,将截图发给群主
              </div>
              <img :src="details.ownerQRCode" alt="" width="40%" class="code_img">

              <div class="step_title step_desc">
                <p>或加群主：{{details.ownerAccount}}</p>
                <p>群主收到截图确认身份后拉您入圈</p>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    <div class="shadow loading_hook" v-show="loading">
      <div class="centerbox">
        <div class="lds-spinner">
          <div></div><div></div><div></div><div></div><div></div><div></div><div></div><div></div><div></div><div></div><div></div><div></div>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
  import logo from 'assets/img/logo.png'
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
        details:null,
        phone:'',
        phoneErr:'',
        vcode:'',
        vcodeErr:'',
        countdown:60,
        resultCode:'',
        loading:false,
        ageArr:'',
        otherText:'',
        miniprogram:Lib.M.checkMiniprogram,
      }
    },
    created () {
      Lib.WX.is_weixin();
      Lib.M.baseFn();
      this.ageArr=this.getAgeArr();
      this.eId=Lib.M.getUrlQuery('eId');
      this.getDetails();
    },
    methods: {
      selectOption(option,item){
        option.selectedOption=item;
        option.selectedOptionIndex=item.orders;
      },
      getAgeArr(){
        let date=new Date();
        let year=date.getFullYear();
        let ageArr=[];
        for (let i = 0; i<78; i++) {
          let age=year-i;
          ageArr.push({
            optionValue:age
          });
        };
        return ageArr;
      },
      checkResult(){
        if(this.phone==''){
          alert('请输入手机号！');
          return ;
        }
        let phone = this.phone;
        let vcode = this.vcode;
        let jsonStr = [];
        let errNum=0;
        this.details.inviteQuestions.forEach((option)=>{ //获取用户题目的答案。
          option.errTips=false;
          if(!option.selectedOption){
            option.errTips='您漏了这题';
            errNum++;
            return false;
          }
          if(typeof option.selectedOption == 'object'){
            jsonStr.push({
              content:option.content,
              optionValue:option.selectedOption.explanation?this.otherText||false:option.selectedOption.optionValue,
              type:option.type,
            });
          }else{
            jsonStr.push({
              content:option.content,
              optionValue:option.selectedOption,
              type:option.type,
            });
          }
        });
        if(errNum===0){
          this.loading=true;
          Lib.M.ajax({//检测验证码
            type:'post',
            url:'/check_code',
            data:{
              phone,
              vcode
            },
            success:res=>{
              if(res.type=='success'){
                let result={
                  phone,
                  vcode,
                  jsonStr:JSON.stringify(jsonStr),
                  groupId:this.eId,
                  sid:Lib.M.getSource(),
                  terminal:Lib.M.checkTerminal(),
                };
                this.sumbitFn(result);
              }else{
                this.loading=false;
                alert(res.content);
              }
            }
          });
        }
      },
      sumbitFn(result){
        Lib.M.ajax({
          type:'post',
          url:'/submit_group',
          data:result,
          success:res=>{
            this.loading=false;
            if(res.type=='success'){
              this.resultCode=res.content;
            }else{
              alert(res.content);
            }
          }
        })
      },
      getDetails(){
        Lib.M.ajax({
          url:`/group_detail/${this.eId}`,
          type:'post',
          success:res=>{
            this.details=res;
            //`邀请您加入${this.details.groupName}`,'移民行业专家在线答疑、移民同路人互助交流、海外华人分享见闻',
            Lib.WX.is_weixin(`加入${this.details.groupName}`,'移民行业专家在线答疑、移民同路人互助交流、海外华人分享见闻',undefined,this.details.sharePicture);
            Lib.M.setSeoFn(`加入${this.details.groupName}`,undefined,'移民行业专家在线答疑、移民同路人互助交流、海外华人分享见闻');
            this.details.inviteQuestions.map((option)=>{
              this.$set(option,'selectedOption',false);
              this.$set(option,'errTips',false);
              if(option.type=='age'){
                option.inviteQuestionOptions=this.ageArr;
              }else if(option.type=='source'){
                this.$set(option,'selectedOptionIndex',false);
              }
              //option.selectedOption=false;
            });
          }
        });
      },
      settime:function (btn) {
        if (this.countdown == 0) {
          btn.disabled=false;
          btn.value="发送验证码";
          this.countdown = 60;
        } else {
          btn.disabled=true;
          btn.value=`重新发送(${this.countdown})`;
          this.countdown--;
          setTimeout( () => {
            this.settime(btn);
          },1000);
        };
      },
      getCode:function (btn) {
        setTimeout( () => {
          Lib.M.ajax({
            url:'/send_code',
            type:'post',
            data: {
              phone:this.phone,
            },
            success:res => {
              if(res.type=='success'){
                this.settime(btn.target);
              }
              else{
                alert(res.content);
              }
            },
          });
        },500);
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
  .main{
    max-width: 768px;
    margin: 0 auto;
    font-size: 14px;
    position: relative;
    .top{
      position: relative;
      .flag{
        width: 1.5rem;
        height: 1.5rem;
        position: absolute;
        bottom: 0;
        left: 50%;
        transform:translateX(-0.75rem);
        background-size: cover;
        background-position: center;
        border-radius: 50%;
        z-index: 100;
      }
    }
    .option_warp{
      margin-top: 0.3rem;
      display: flex;
      display: -webkit-flex;
      flex-wrap: wrap;
      -webkit-flex-wrap: wrap;
      .option_item{
        flex:0 0 1.68rem;
        margin-right: 0.5rem;
        margin-bottom: 0.24rem;
        border: 1px solid #DCDCDC;
        border-radius: 4px;
        font-size: 14px;
        color: #898989;
        height: 0.64rem;
        text-align: center;
        line-height: 0.64rem;
        box-sizing: border-box;
      }
      .black{
        color:#000;
      }
      .selected{
        border: 1px solid #1CA9FF;
        color:#1CA9FF;
      }
      .other_text{
        flex: 0 0 6rem;
        border: 1px solid #1CA9FF;
        border-radius: 4px;
        text-indent: 15px;
        height: 0.64rem;
        line-height: 0.64rem;
      }
    }
    .option_selelt_warp{
      position: relative;
      .option_selelt{
        height: 40px;
        line-height: 40px;
        text-indent: 10px;
        margin-top: 10px;
        width: 100%;
        border: 1px solid #DCDCDC;
        border-radius: 6px;
        background: #fff;
        color:#898989;
      }
      .icon-Shape{
        position: absolute;
        right: 16px;
        color: #C3C3C3;
        font-size: 10px;
        bottom: 14px;
      }
      .black{
        color:#000;
      }
    }
    .input_warp{
      text-align: center;
      .input_title{
        font-weight: 600;
        font-size: 16px;
      }
      .desc{
        font-size: 12px;
        margin:10px 0;
      }
      .input{
        .error_tips{
          position: relative;
          top:-8px;
        }
        .phone{
          width: 100%;
          border: 1px solid #DCDCDC;
          border-radius: 6px;
          height: 30px;
          line-height: 30px;
          text-indent: 10px;
          margin: 5px 0 16px 0;
          outline: none;
        }
        .vcode_warp{
          display: flex;
          margin-bottom: 16px;
          border: 1px solid #DCDCDC;
          border-radius: 6px;
          padding: 7px 0;
          .vcode{
            outline: none;
            text-indent: 10px;
            border:none;
            flex:1;
          }
          .getvcode_btn{
            flex:0 0 1rem;
            color: #42C4FE;
            background: #fff;
            border-left: 1px solid #DCDCDC;
          }
        }
      }
      .sumbit_btn{
        outline: none;
        background-image: linear-gradient(223deg, #4FD5FF 0%, #2AB8FF 45%, #139EFF 100%);
        border-radius: 20px;
        width: 4rem;
        height: 1rem;
        line-height: 1rem;
        text-align: center;
        color:#fff;
        display: inline-block;
        margin: 0 auto;
        border: none;
        font-size: 20px;
      }
    }
  }
  .warps{
    background: #FFF;
    position: relative;
    top: -38px;
    //padding-top: 40px;
    margin: 0 0.16rem;
    border-radius: 6px;
  }
  .warp:first-child{
    padding-top: 45px;
  }
  .warp{
    padding: 0.22rem 0.3rem 0.3rem 0.3rem;
    background: #FFF;
    box-shadow: 2px 2px 3px 0 rgba(176,176,176,0.50);
    border-radius: 6px;
    margin-bottom: 0.16rem;
  }
  .herf{
    display: block;
    height: 100%;
  }
  .group{
    position: relative;
    top:-40px;
    padding: 0.36rem 0.6rem 0.44rem 0.6rem;
  }
  .error_tips{
    text-align: left;
    color:#DF3854;
  }
  .result_warp{
    .step_warp{
      padding: 0.6rem 0;
      .step_num{
        display: inline-block;
        background: #42C4FE;
        font-size: 14px;
        color: #FFF;
        position: absolute;
        top: -3px;
        height: 20px;
        width: 20px;
        text-align: center;
        line-height: 20px;
        border-radius: 50%;
        left: 5px;
      }
      .code_warp{
        font-size: 26px;
        color: #DF3854;
        text-align: center;
        position: relative;
        height: 75px;
        box-sizing: border-box;
        padding: 12px 0;
        &:after{
          content: '';
          position: absolute;
          left: 13px;
          top: 12px;
          bottom: 12px;
          width: 1px;
          border-left: 4px dotted #42C4FE;
        }
      }
      .step_title{
        text-align: center;
        font-size: 16px;
        position: relative;
        color: #222222;
      }
      .step_desc{
        font-size: 14px;
        color:#868686;
        line-height: 20px;
      }
      .code_img{
        display: block;
        margin: 0.3rem auto;
      }
    }
    .top{
      .flag{
        bottom:-10px;
      }
    }
    .warps{
      top: 0;
    }
    .warp:first-child{
      padding: 0;
      margin-bottom: 0.3rem;
    }
    .tips{
      padding: 0 0.3rem;
      display: flex;
      font-size: 0.32rem;
      color: #222222;
      box-sizing: border-box;
      height: 1.28rem;
      line-height: 0.6rem;
      .icon-pass{
        color:#09BB07;
        font-size: 0.6rem;
        display: inline-block;
        //vertical-align: top;
        position: relative;
        top: 0.12rem;
        margin-right: 8px;
      }
    }
  }
  .group_title{
    position: relative;
    text-align: center;
    letter-spacing:1px;
    font-size: 16px;
    &:after{
      content:'';
      display: inline-block;
      position: relative;
      height: 1px;
      width: 1rem;
      top: -5px;
      background-image: linear-gradient(to left, rgba(55,194,255,0.19), rgba(53,193,255,1));
    }
    &:before{
      content:'';
      display: inline-block;
      position: relative;
      height: 1px;
      top: -5px;
      width: 1rem;
      background-image: linear-gradient(to right, rgba(55,194,255,0.19), rgba(53,193,255,1));
    }
  }
  .red{
    color: #DF3854;;
  }
</style>
