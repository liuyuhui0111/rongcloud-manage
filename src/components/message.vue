<template>
  <div v-if="isShowMessageBox"
  @click="isShowEmoji=false;isShowquickmesBox=false;closeCofirmBox=false;"
  class="message common-message">
  <!-- 用户列表 -->
  <div class="userlistbox">
  <div class="userlist">
    <div class="search-box">
     <el-popover
          placement="right"
          trigger="click">
            <!-- 个人信息 -->
            <div class="useraccount">
              <span class="title">{{curUserData.name}}</span>
              <p>{{curUserData.account}}</p>
            </div>
      <img slot="reference" :src="userInfo.icon" alt="">
      </el-popover>
      <div @keyup.enter="SearchPageFn" class="inpbox">
        <span @click="SearchPageFn" class="iconfont iconsousuo"></span>
        <el-input v-model="searchVal"
        placeholder=""></el-input>
      </div>

    </div>
    <div class="list common-scroll-bar">
      <div v-for="(item,index) in userlist"
      @click="checkUser(item,index)"
      :data="item.newmes"
      class="item"
      :class="[
      {on:curMessageIndex==index},
      {stop:item.type==1},
      {newmes:item.newmes==1},
      {group:item.exchange == 1 && item.toExpertId != curUserData.id},
      {gray1:item.online==1 || item.status==2}
      ]"
      :key="index">
        <div class="imgbox">
        <span class="qtype" v-if="item.qtype == '0' && item.status==2">问题未分类</span>
        <div :class="{gray:item.online==1 || item.status==2}">
          <img :src="item.icon" alt="">
          <img v-if="item.exchange == 1
          && item.toExpertId != curUserData.id"
          :src="item.fromIcon"
          alt="" class="expertfrom">
          <img v-if="item.exchange == 1
          && item.toExpertId != curUserData.id"
          :src="item.toIcon"
          alt="" class="expertto">
        </div>
        </div>
        <div :class="{gray:item.online==1 || item.status==2}" class="intro">
          <p class="namebox">
            <span class="name">{{encryptionFn(item.name)}}</span>
            <span class="time">{{getTimeFn(item,'2')}}</span>
          </p>
          <p class="idbox">
            <span class="mesid">咨询单id:{{item.code}}</span>
            <span v-if="item.exchange == 0
            || (item.exchange == 1
            && item.fromExpertId != curUserData.id)"
            class="iconfont iconzixunzhong"></span>
            <span v-if="item.exchange == 1
            && item.fromExpertId == curUserData.id"
            class="iconfont iconzixunzhong"></span>
          </p>
        </div>
      </div >
    </div>

  </div>
  </div>
  <!-- 聊天窗口 -->
  <template v-if="userlist.length>0">
    <div v-for="(uitem,uindex) in userlist"
    :key="uindex"

      v-show="uindex === curMessageIndex"
      class="messagebox">


      <div class="headerbox">
      <template v-if="curTargetUserData.userType == 'P'
      || curTargetUserData.userType == 'C'">
      <el-popover
          placement="bottom"
          trigger="click">
            <!-- 个人信息 -->
            <div class="useraccount">
              <span v-if="curTargetUserData.userType == 'P'" class="title">个人账户</span>
              <span v-if="curTargetUserData.userType == 'C'" class="title">企业账户</span>
              <p v-if="curTargetUserData.userType == 'C'">
              所属企业：{{curTargetUserData.userCompany}}
              </p>
              <p v-if="curTargetUserData.userType == 'C'">
              所属行业：{{curTargetUserData.companyTrade}}</p>
              <p v-if="curTargetUserData.userType == 'P'">
              所属地区：{{curTargetUserData.userCity}}</p>
              <p v-if="curTargetUserData.userType == 'C'">
              所属地区：{{curTargetUserData.companyLocation}}</p>
            </div>
          <div slot="reference" class="title">

            <span class="icon"></span>

          <span class="name">
          {{curTargetUserData.name}}咨询中
          </span>
        </div>
        </el-popover>
        </template>
        <template v-else>
          <div class="title">

            <span class="icon"></span>

          <span class="name">
          {{curTargetUserData.name}}咨询中
          </span>
        </div>
        </template>
        <div class="time">已咨询{{getTimeFn(curTargetUserData.duration,'1')}}</div>
        <div class="btns">
        <span @click="hideMessage" class="close">-</span>
        </div>
      </div>
      <!-- 聊天展示区域 -->
      <div :id="'meslist'+uindex"
      class="meslist common-scroll-bar">

        <div
        v-for="(item,index) in meslist[uitem.mesid]"
        class="item"
        :class="[
          {on:item.senderUserId==curUserData.id || item.fromUserId==curUserData.id},
          {system:item.objectName == 'RC:InfoNtf'
          },
          {create:item.content.type == '0001'},
        ]"
        :key="index"
        :ref="index">
          <div v-if="item.content.extra
          && item.objectName != 'RC:InfoNtf'
          && item.objectName !=='RC:GrpNtf'
          && item.objectName != 'RC:DxhyMsg'" class="user">
            <img :src="item.content.extra.icon" alt="">
          </div>
          <div v-if="item.content.contentType =='MSG_LOG'" class="user">
            <img :src="item.content.extra.icon" alt="">
          </div>

          <template v-if="item.objectName == 'RC:DxhyMsg'">
              <div class="msg-user-source msgbox"
              v-if="item.content.contentType =='MSG_USER_SOURCE'">
                <!-- 来源 -->
                <div class="mesitem">用户来源：{{item.content.content.fromName}}</div>
                <div @click="openWindow(item.content.content.fromAddress)"
                class="mesitem pointer">当前正在浏览的页面：{{item.content.content.fromContent}}</div>
              </div>

               <div class="msg-evaluate msgbox"
               v-if="item.content.contentType =='MSG_EVALUATE'">
                <!-- 查看评价 -->
                <span @click="showEvaluateFn(item)" class="btn-sub">查看评价</span>
              </div>
              <!-- eslint-disable -->
              <div class="mes meslog" @click="showMesLog(item.content)" 
              v-if="item.content.contentType =='MSG_LOG'">
               <div class="logtitle"> {{item.content.title}}</div>
                <p class="ellipsis"
                v-for="(contentItem,contentIndex) in item.content.content.slice(1,4)"
                :key="contentIndex">
                {{contentItem.content.extra.name}} : {{contentItem.content.content}}
                </p>
                <div class="pointer">查看聊天记录</div>
                <span class="after"></span>
              </div>
              <!-- eslint-enable -->
          </template>


          <template v-if="item.objectName == 'RC:InfoNtf'">
          <!-- 系统消息 -->
            <div class="systembox">
              <p v-text="getSystemTip(item)"></p>
              <span class="after"></span>
            </div>

          </template>
          <div class="mesbox">
          <!-- <div v-if="item.content.user &&
          item.content.messageName != 'InformationNotificationMessage'
          && item.objectName != 'RC:DxhyMsg'"
          class="name">{{item.content.user.name}}</div> -->

          <template v-if="item.objectName == 'RC:TxtMsg'">
          <!-- 文本消息 -->
            <div class="mes">
            <p v-html="emojiToHtml(item.content.content)"></p>
            <span class="after"></span>
            </div>
          </template>


          <template v-if="item.objectName == 'RC:ImgMsg'">
            <!-- 图文消息 -->
            <div class="imgbox mes"
            @click="imgClick(item)"
            :class="{finish:!item.uploading}">
              <img :id="item.id" :src="item.content.content">
              <span class="after"></span>
            </div>
          </template>

          <template v-if="item.objectName == 'RC:FileMsg'">
            <!-- 文件消息 -->
            <div class="filebox imgbox mes"
            @click="imgClick(item)"
            :class="{finish:!item.uploading}">
              <a href="javascript:">{{item.content.name}}</a>
              <span class="after"></span>
            </div>
          </template>


          </div>
        </div >
        <div class="h34"></div>
      </div>

      <!-- 快捷回复 -->
      <div v-show="isShowquickmesBox"
      class="quickmes-box common-scroll-bar">
        <div v-for="(item,index) in getCommonPhrasesListRes"
        class="item"
        @click.stop="quickmes(item)"
        :key="index">
        {{item.name}}
        </div >
      </div>

      <!-- toast提示框 已转单 问题已分类 -->
      <div v-show="isShowToast" class="toast">
       <p>{{toaststr}}</p>
      </div>

      <!-- 发送区域 -->
      <div class="sendbox">
        <div v-if="(userlist[curMessageIndex].exchange == 1 &&
        userlist[curMessageIndex].fromExpertId == curUserData.id)
        || userlist[curMessageIndex].status==2" class="hidemask">
          <!-- 禁用聊天遮罩层 -->
          <div
          @click="showQuestionFn()"
          v-if="userlist[curMessageIndex].status==2
          && userlist[curMessageIndex].qtype == '0'"
          class="toast pointer"><p>已结束咨询，点击去分类</p></div>

          <div v-if="userlist[curMessageIndex].status==2
          && userlist[curMessageIndex].qtype == '1'"
          class="toast"><p>已结束咨询</p></div>

          <div v-if="userlist[curMessageIndex].exchange == 1
          &&userlist[curMessageIndex].fromExpertId == curUserData.id" class="toast">
          <!-- 转单标志为1 且当前用户id 等与转单id  -->
           <p>已转单给{{userlist[curMessageIndex].consultExpert.name}}</p>
          </div>

        </div>
        <div class="btns">
          <span @click.stop="isShowquickmesBox=false;
          closeCofirmBox=false;
          emojiClick();" class="iconfont iconbiaoqing"></span>

          <label for="picture">
          <span title="上传图片"
          class="iconfont iconfasongtupian"></span></label>

          <label for="file">
          <span title="上传文件"
          class="iconfont iconfasongwenjian"></span>
        </label>

      <!--     <span @click="upload('img')" class="iconfont iconfasongtupian"></span>


          <span @click="upload('file')" class="iconfont iconfasongwenjian"></span> -->


          <!-- 快捷回复 -->

          <span title="快捷回复"
          @click.stop="isShowEmoji=false;
          closeCofirmBox=false;
          isShowquickmesBox = !isShowquickmesBox"
          class="iconfont iconkuaijiehuifu"></span>

          <!-- 问题分类 -->
          <span
          @click="showQuestionFn()"
          title="问题分类"
          class="iconfont iconwentifenlei"></span>

          <!-- 转单 -->
          <span @click="changeExpert()"
          title="转单"
          class="iconfont iconzhuandan1"></span>

           <!-- 结束 -->


          <div class="re">
            <div v-show="closeCofirmBox" class="close-confirm-box dialog-box-base">
              <div class="box">
                <p>询问用户是否可以结束咨询？</p>
                <div class="btnbox">
                  <span @click.stop="closeCofirmBox=false;" class="cancel">取消</span>
                  <span @click.stop="proposeEndConsultFn()" class="sub">确定</span>
                </div>
              </div>
            </div>
          <span @click.stop="isShowEmoji=false;
          isShowquickmesBox=false;
          closeCofirmBox=!closeCofirmBox"
          class="iconfont iconjieshuzixun"></span>
          </div>

        </div>
        <div class="inp">
          <div v-if="isShowEmoji && emojiList.length>0"
          @click.stop
          class="emojibox">
             <div class="item" v-for="(item,index) in emojiList"
             @click="emojiClickItem(item)"
             :key="index" v-html="item.node.outerHTML">
             </div >
          </div>
          <div @keyup.enter="startSendMes(1)">
            <el-input

            type="textarea"
            placeholder="说点什么吧"
            v-model="mesData"
            rows="3"
            resize="none"
            show-word-limit
            ></el-input>
          </div>
          <!-- <textarea v-model="mesData" placeholder="说点什么吧"></textarea> -->
        </div>
      </div>
      <div class="btn-sub sub" @click="startSendMes(1)">发送</div>


      <!-- 结束会话 -->


      <!-- 上传压缩相关隐藏dom -->
      <input style="display:none;"
          @change="fileChange($event,'img')"
          accept="image/jpeg,image/jpg,image/gif,image/png,image/bmp"
          ref="picture"
          type="file" id="picture" name="picture">
          <input
          @change="fileChange($event,'file')"
          ref="file"
          style="display:none;" type="file" id="file" name="file">
      <canvas id="canvas" style="display:none;"></canvas>

    </div>
  </template>
  <template v-else>
    <div class="messagebox">
      <!-- 还没有用 -->
       <div class="headerbox">

          <div class="title">


          <span class="name">
          暂无咨询消息
          </span>
        </div>
        <div class="time"></div>
        <div class="btns">
        <span @click="hideMessage" class="close">-</span>
        </div>
      </div>
    </div>
  </template>

    <!-- 问题分类弹窗 -->
    <template v-if="questionStr">
      <question @back="isShowQuestion=false;"
      :questionStr="questionStr"
      :mesid="curid"
      @success="questionSuccess"
      v-show="isShowQuestion"></question>
    </template>

    <!-- 聊天记录 -->
    <template v-if="isShowMeslog">
      <meslog @back="isShowMeslog=false;"
      :mesid="curid"
      :meslogList = "meslogList"></meslog>
    </template>

    <!-- 转单专家列表弹窗 -->
    <template v-if="isShowExpect">
    <expertlist
    :mesid="curid"
    :expertid="userInfo.id"
    :targetId="targetId"
    @success="changeExpertSuccess"
    @back="isShowExpect=false;"></expertlist>
    </template>

    <!-- 评价弹窗 -->
      <div v-if="dialogCompentVisible" class="dialog-mask">
        <!-- 评价弹窗弹窗 -->
        <div class="dialog-box-componet">
        <el-dialog title=""
        :modal-append-to-body="boolFalse"
        :modal="boolFalse"
        :visible.sync="dialogCompentVisible">
           <div class="title">此次咨询服务用户对您的评价是{{rateParam.rateVal}}分</div>
           <div class="ratebox">
              <div class="label">{{rateMap[rateParam.rateVal]}}</div>
              <div class="rate">
                <el-rate
                  v-model="rateParam.rateVal"
                  disabled
                  :colors="colors">
                </el-rate>
              </div>
           </div>
           <div class="tagbox">
             <div v-for="(item,index) in rateTagMap"
             class="item"
             v-show="!isShowTag(item)"
             :class="{on:item.active}"
             :key="index">
              {{item.text}}
             </div >
           </div>
           <div class="inp">
            评价内容：{{rateParam.rateMes}}

           </div>

        </el-dialog>
        </div>
      </div>
  </div>
</template>
<script>

import message from './message';

export default message;

</script>
<style>
  .el-popover{
    min-width: 50px;
  }
  .common-message .inp .el-textarea__inner{
    border: none;
  }
  .common-message .closeCofirmBox{
    position: absolute;
    bottom: 100px;
    left: 120px;
  }
  .el-message-box__message p{
    text-align: center;
    font-size: 14px;
    margin-bottom: 10px;
    color: #333333;
  }
  .el-message-box__btns{
    text-align: center;
    margin-bottom: 10px;
  }
  .el-input__inner:active,
  .el-input__inner:focus,
  .el-input__inner:hover{
    border-color: #33C8DF;
  }
  .el-message-box__btns .el-button--primary:active,
  .el-message-box__btns .el-button--primary:focus,
  .el-message-box__btns .el-button--primary:hover,
  .el-message-box__btns .el-button--primary{
    color: #FFF;
    background-color: #33C8DF;
    border-color: #33C8DF;
  }
  .el-message-box__btns button:nth-child(2){
    margin-left: 20px;
  }
  .common-dialog-mes .icon-auth{
    padding-left: 18px;
  }
  .common-dialog-mes .icon-auth:before{
    content: "";
    position: relative;
    width: 10px;
    height: 10px;
    border-radius: 10px;
    background: #33C8DF;
    display: inline-block;
    margin-right: 8px;
  }
  .common-message .dialog-box .el-dialog{
    width: 400px;
    /*box-sizing:border-box;*/
  }
  .common-message .dialog-box-componet .el-dialog{
    width: 500px;
    box-sizing:border-box;
  }
  .el-dialog__body{
    padding: 10px 32px;
    color: #333;
    font-size: 14px;
  }
  .el-dialog__wrapper{
    overflow: hidden;
  }
   .common-layout-index .el-input__inner{
    background: none;
color: #333;
border: none;
outline: none;
height: 28px;
padding: 0 0 0 0px;
  }

.common-message .messagebox .headerbox .title .icon{
  width: 15px;
  height: 15px;
  background: url('./img/icon-user.png') no-repeat center center;
  -webkit-background-size: 100% 100%;
  background-size: 100% 100%;
  border-radius: 100%;
  margin-right: 9px;
  cursor: pointer;
  display: block;
}

</style>
<style scoped>
/*个人信息*/
.useraccount{
  padding: 10px 20px;
}
/*自定义弹框白色三角*/
.dialog-box-base .box{
border: 1px solid #D8D8D8;
border-radius: 2px;
display: block;
width: 100%;
height: 100%;
z-index: 0;
}
.dialog-box-base:after{
  content: "";
  position: absolute;
   width: 6px;
  height: 6px;
  background: #fff;
  z-index: 2;
  bottom: -5px;
  transform: rotate(-45deg);
  border-left: 1px solid #D8D8D8;
  border-bottom: 1px solid #D8D8D8;
  left: 50%;
}
/*自定义消息*/
.msgbox {
  width: 100%;
}
.msgbox .mesitem{
  display: block;
  width: 85%;
  margin: 12px auto;
  background: #EDEDED;
border-radius: 4px;
font-size: 12px;
color: #333333;
text-align: center;
padding: 8px;
line-height: 18px;
}
/*toast提示信息*/
.toast{
  position: absolute;
  z-index: 10;
  background: none;
  display: flex;
  align-items: center;
  justify-content: center;
  width: 100%;
  bottom: 110px;
}
.toast p{
  display: inline-block;
  padding: 8px 23px 8px 40px;
  font-size: 12px;
color: #333333;
line-height: 16px;

border-radius: 4px;
background: url('./img/success.png') no-repeat 23px center;
background-size: 14px 14px;
background-color: #EBF9E4;
}
/*消息列表*/
.quickmes-box{
  position: absolute;
  z-index: 11;
  background: #fff;
  width: 360px;
  height: 146px;
  overflow-y: auto;
  bottom: 101px;
  left: 30px;
  background: #F9F9F9;
border: 1px solid #D9D9D9;
border-radius: 4px 4px 0 0;
box-sizing:border-box;
padding: 20px 16px 0 16px;
font-size: 14px;
/*display: flex;*/
/*flex-wrap: wrap;*/
}
.quickmes-box .item{
  line-height: 20px;
  padding-bottom: 10px;
  cursor: pointer;
  color: #666666;
  cursor: pointer;
}
.quickmes-box .item:hover{
  color: #33C8DF;
}
/*用户列表*/
/*已分类*/
.userlistbox .qtype{
  position: absolute;
  font-size: 10px;
color: #F56C6C;
background: #FFEAEA;
border-radius: 4px;
z-index: 2;
padding: 0 2px;
left: 0px;
right: 0;
text-align: center;
}

 .userlistbox .iconsousuo{
    color: #0898ad;
    cursor: pointer;
    font-size: 16px;
  }
  .userlistbox .iconzixunzhong{
    color: #09bb0c;
  }
  .userlistbox .search-box .inpbox{
    background: #86E3F1;
    border-radius: 2px;
    border-radius: 2px;
    display: flex;
    box-sizing:border-box;
    align-items: center;
    justify-content: space-between;
    padding-left: 10px;
  }

  .userlistbox .userlist{
    width: 280px;
    height: 100%;
    position: absolute;
    display: block;
    box-sizing:border-box;
    left: 0;
    top: 0;
    background: #fff;
    border-right:1px solid #e7e7e7;
  }
  .userlistbox .userlist .item{
    display: block;
    display: flex;
    align-items: center;
    justify-content: space-between;
    height: 62px;
    box-sizing:border-box;
    padding: 10px;
    cursor: pointer;
  }
  .userlistbox .userlist .item.gray1{
    background: rgba(197,197,197,1);
  }
  .userlistbox .userlist .item.on{
    background: #EDEDED;
  }
  .userlistbox .userlist .item .imgbox{
    width: 74px;
    height: 44px;
    overflow: hidden;
    margin-right: 10px;
    position: relative;
  }
   .userlistbox .userlist .item .imgbox img{
    width: 44px;
    border-radius: 100%;
    position: absolute;
    left: 10px;
   }
   .userlistbox .userlist .item.group .imgbox img{
    width: 20px;
    height: 20px;
    position: absolute;
    border-radius: 100%;
    left: 20px;
   }
   .userlistbox .userlist .item.group .imgbox img.expertto{
    top: 20px;
    left: 8px;
   }
   .userlistbox .userlist .item.group .imgbox img.expertfrom{
    top: 20px;
    left: 34px;
   }
  .userlistbox .userlist .item .intro{
    width: 206px;
    position: relative;
  }
  .userlistbox .userlist .item.newmes .intro:after{
    content: "";
    position: absolute;
    width: 8px;
    height: 8px;
    background: #f00;
    border-radius: 100%;
    right: 0px;
    bottom: 10px;
  }
  .userlistbox .userlist .item .intro p{
    width: 100%;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 12px;
    color: #333;
  }
  .userlistbox .userlist .item .intro .name{
    font-size: 14px;
  }
  .userlistbox .userlist .item .intro .time{
    color: #C0C0C0;
  }
  .userlistbox .userlist .item .intro .mesid{
    color: #666666;
  }
  .userlistbox .userlist .list{
    position: absolute;
    top: 50px;
    bottom: 0;
    left: 0;
    right: 0;
    overflow-y: auto;
  }
  .userlistbox .iframe-box{
    height: 100%;
    left: 280px;
    top: 0;
    right: 0;
    display: block;
    box-sizing:border-box;
    position: absolute;
  }
  .userlistbox .search-box{
    position: absolute;
    width: 100%;
    height: 50px;
    display: flex;
    align-items: center;
    justify-content: space-between;
    padding: 10px;
    box-sizing:border-box;
    background: #33C8DF;
  }
  .userlistbox .search-box img{
    width: 36px;
    height: 36px;
    border-radius: 100%;
    display: block;
    margin-right: 12px;
  }
/*是否可以结束咨询提示*/
.closeQuestion{
  background: #FFFFFF;
border: 1px solid #D8D8D8;
border-radius: 4px;
border-radius: 4px;
box-sizing:border-box;
padding: 16px;
text-align: left;
font-size: 13px;
color: #333;
}
.closeQuestion p{
  margin-bottom: 16px;
}
.closeQuestion .btns{
  display: flex;
  align-items: center;
  justify-content: center;
}
.closeQuestion .btns span{
  padding: 5px 20px;
  display: block;
  border: 1px solid #33C8DF;
border-radius: 4px;
border-radius: 4px;
text-align: center;
font-size: 14px;
color: #33C8DF;
box-sizing:border-box;
cursor: pointer;
}
.closeQuestion .btns .yes{
  margin-right: 10px;
}
/*评价*/
.dialog-box-componet .title{
  font-size: 18px;
color: #333333;
font-weight: bold;
margin-bottom: 24px;
text-align: center;
}
.dialog-box-componet .ratebox{
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 14px;
  margin-bottom: 16px;
}
.dialog-box-componet .rate{
  margin-right: 25px;
}
.dialog-box-componet .tagbox{
  display: flex;
  align-items: center;
  justify-content: center;
}
.dialog-box-componet .tagbox .item{
  display: block;
   height: 25px;
  line-height: 25px;
  text-align: center;
  border: 1px solid #D8D8D8;
border-radius: 4px;
border-radius: 4px;
font-size: 12px;
color: #666666;
margin-right: 8px;
cursor: pointer;
padding: 0 10px;
}
.dialog-box-componet .tagbox .item.on{
  color: #33C8DF;
  border-color: #33C8DF;
}
.dialog-box-componet .inp{
margin-top: 16px;
margin-bottom: 32px;
}

/*没有权益提交用户信息*/
  .dialog-box .title{
    text-align: left;
    margin-bottom: 16px;
  }
  .dialog-box .tip{
    font-size: 12px;
    margin-bottom: 6px;
  }
  .dialog-box .inpbox{
    display: flex;
    align-items: center;
    justify-content: space-between;
  }
  .dialog-box .inpbox .name{
    width: 33%;
  }
  .dialog-box .inpbox .tel{
    width: 65%;
  }
/*弹窗*/
.dialog-mask{
  position: fixed;
  background: rgba(0,0,0,0.6);
  top: 0;
  left: -10px;
  right: -10px;
  bottom: -10%;
  height: 110%;
  z-index: 100;
}
.dialog-box{
  background: #fff;
}
/*问题弹窗*/
.dialog-question{
  position: absolute;
  width: 100%;
  height: 100%;
  overflow-y: auto;
  overflow-x: hidden;
  background: #fff;
  left: 0;
  top: 0;
  z-index: 9;
}
.dialog-header{
  height: 50px;
  line-height: 50px;
  padding: 0 15px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  box-shadow: 0 1px 0 0 rgba(0,0,0,0.15);
  font-size: 14px;
color: #333333;
font-weight: bold;
}
.dialog-content{
  width: 85%;
  margin: 0 auto;
}
.dialog-content p{
  padding-top: 72px;
  padding-bottom: 16px;
  font-size: 14px;
  color: #333333;
}


.popover{
  font-size: 12px;
  color: #333;
  text-align: center;
  display: block;
  height: 22px;
  line-height: 22px;
}
/*系统提示*/

.systemtips{
  position: fixed;
  width: 100%;
  height: 32px;
  line-height: 32px;
  text-align: center;
  font-size: 12px;
  color: #333333;
  background: #FFFBE6;
  z-index: 10;
}
.h34{
  height: 32px;
}
.systemtips span{
  color: #e6a23c;
  font-weight: bold;
}

/*禁用聊天*/
.hidemask{
  position: absolute;
  z-index: 99;
  background: rgba(222,222,222,0.6);
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  color: #000;
  text-align: center;
  padding-top: 40px;
}
.emojibox{
    position: absolute;
    width: 100%;
    z-index: 111;
    background: #fff;
    bottom: 100px;
}
.emojibox div.item{
  display: inline-block;
}
/*确认关闭*/
.re{
  position: relative;
}

.close-confirm-box{
  width: 204px;
  height: 98px;
border-radius: 2px;
text-align: center;
font-size: 12px;
color: #333;
position: absolute;
left: -90px;
bottom: 35px;
background: #fff;
}
.close-confirm-box p{
  padding-top: 24px;
}
.close-confirm-box .btnbox{
  display: flex;
  align-items: center;
  justify-content: center;
  margin-top: 16px;
  font-size: 12px;
}
.close-confirm-box .btnbox span{
  display: block;
  width: 48px;
  height: 25px;
  text-align: center;
  line-height: 25px;
  border: 1px solid #33C8DF;
border-radius: 4px;
border-radius: 4px;
cursor: pointer;
}
.close-confirm-box .btnbox span.cancel{
  color: #33C8DF;
  background: #fff;
  margin-right: 20px;
}
.messagebox .sendbox .btns .close-confirm-box span:hover,
.messagebox .sendbox .btns .close-confirm-box span{
  font-size: 12px;
  padding: 2px 12px;
}
.messagebox .sendbox .btns .close-confirm-box span.sub:hover{
  color:#fff;
}
.close-confirm-box .btnbox span.sub{
  background: #33C8DF;
  color: #fff;
}
.messagebox{
  position: absolute;
  bottom: 0;
  right: 0;
  left: 280px;
  background: #fff;
  height: 100%;
  box-shadow: 0 1px 0 0 rgba(0,0,0,0.15);
  z-index: 99;
}
.messagebox .headerbox{
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: flex-start;
  background: #FFFFFF;
  box-shadow: 0 1px 0 0 rgba(0,0,0,0.15);
  font-size: 14px;
  color: #333333;
  padding: 0 16px;
  font-weight: bold;
  border-bottom: 1px solid #D8D8D8;
  box-sizing:border-box;
}
.messagebox .headerbox .btns{
  width: 30px;
  height: 30px;
  display: flex;
  justify-content: flex-end;
  align-items: center;
  position: absolute;
  right: 16px;
}
.dialog-header .close,
.messagebox .headerbox .btns span{
  width: 16px;
  height: 16px;
  line-height: 14px;
  font-size: 14px;
  text-align: center;
  cursor: pointer;
  background: #33C8DF;
  color: #fff;
  border-radius: 100%;
}
.messagebox .headerbox .title{
  display: flex;
  align-items: center;
  justify-content: flex-start;
  margin-right: 62px;
  cursor: pointer;
}
.messagebox .headerbox .time{
  font-size: 12px;
color: #666666;
font-weight: normal;
display: flex;
align-items: center;
}


.messagebox .sendbox{
  height: 100px;
  bottom: 0;
  right: 0;
  position: absolute;
  width: 100%;
  border-top:1px solid #ccc;
}
.messagebox .sendbox .inp{
  position: relative;
}
.messagebox .sendbox .btns{
  height: 30px;
  display: flex;
}
.messagebox .sendbox .btns span{
  display: block;
  font-size: 20px;
  width: 30px;
  text-align: center;
  cursor: pointer;
  line-height: 30px;
}
.messagebox .sendbox .btns span.iconwentifenlei,
.messagebox .sendbox .btns span.iconzixunzhong,
.messagebox .sendbox .btns span.iconzhuandan1{
  font-size: 17px;
}
.messagebox .sendbox .btns span:hover{
  color: #33C8DF;
  font-size: 17px;
}
.messagebox textarea{
  width: 100%;
  border: none;
  height: 70px;
  outline: none;
  box-sizing:border-box;
  padding: 0 15px 15px 15px;
  resize: none;
}
.messagebox .btn-sub{
  margin: 0 auto;
  display: block;
   width: 68px;
  height: 30px;
   line-height: 30px;
  color: #fff;
  text-align: center;
  font-size: 14px;
  background: #33C8DF;
  cursor: pointer;
  border-radius: 4px;
}
.dialog-content .btn-sub{
  margin-top: 40px;
}
.common-message .dialog-box .btn-sub{
  margin: 24px auto;
}
.common-message .dialog-box-componet .btn-sub{
  margin-bottom: 24px;
}
.common-message .dialog-box .custom-tel{
  font-size: 12px;
color: #666666;
text-align: center;
}
.messagebox .btn-sub.sub{
  position: absolute;
  z-index: 9;
  right: 10px;
  bottom: 10px;
}
.messagebox .meslist{
  position: absolute;
  width: 100%;
  top: 50px;
  bottom:100px;
  overflow-y: auto;
  background: #fff;
}
.messagebox .meslist .item{
  overflow: hidden;
  position: relative;
  width: 100%;
  padding: 10px 50px;
  box-sizing:border-box;
  display: flex;
  align-items: center;
  justify-content: flex-start;
}
.messagebox .meslist .item .user{
  position: absolute;
  width: 32px;
  height: 32px;
  border-radius: 4px;
  overflow: hidden;
  left: 10px;
  top: 10px;
}
.messagebox .meslist .item.on .user{
  left: inherit;
  right: 10px;
}
.messagebox .meslist .item .user img{
  width: 100%;
  height: 100%;
}
.messagebox .meslist .item .mesbox{
  max-width: 100%;
  font-size: 13px;
  line-height: 22px;
  position: relative;
}
.messagebox .meslist .item .mesbox .name{
  font-size: 12px;
  color: #666;
  text-align: left;
}
.messagebox .meslist .item.on .mesbox .name{
text-align: right;
display: none;
}
.messagebox .meslist .item.on .imgbox{
  position: relative;
}
.messagebox .meslist .item.on .imgbox:after{
  content: "上传中...";
  position: absolute;
  width: 100%;
  height: 100%;
  z-index: 2;
  background: rgba(0,0,0,0.6);
  left: 0;
  top: 0;
  display: flex;
  align-items: center;
  justify-content: center;
  text-align: center;
}
.messagebox .meslist .item.on .finish:after{
  display: none;
}
.messagebox .meslist .item.on .imgbox img{
  width: 100%;
  opacity: 1;
}
.messagebox .meslist .item .mes{
  display: inline-block;
  padding: 5px 15px;
  max-width: 100%;
  color: #333;
  background: #F5F5F5;
  border-radius: 4px;
  word-break: break-all;
  position: relative;
  min-height: 22px;
}
.messagebox .meslist .item .meslog p{
  font-size: 13px;
  line-height: 18px;
color: #999999;
}
.messagebox .meslist .item .meslog .logtitle{
  font-size: 15px;
color: #333333;
line-height: 21px;
margin-bottom: 5px;
}
.messagebox .meslist .item .meslog .pointer{
  padding: 7px 0;
  margin-top: 7px;
  border-top: 1px solid #E7E7E7;
  color:#666;
}
.messagebox .meslist .item .mes.filebox{
  padding: 40px 20px 20px 20px;
  text-align: center;
  background: url('./img/file.png') no-repeat center 8px;
  background-size: 30px auto;
  cursor: pointer;
}
.messagebox .meslist .item.on{
  justify-content: flex-end;
}

.messagebox .meslist .item.system .mes{
  padding: 0;
  background-color: #E7E7E7;
  font-size: 12px;
color: #777777;
letter-spacing: 0;
text-align: center;
margin: 0 auto;
}
.messagebox .meslist .item.system .mes span.after{
  display: none;
}
.messagebox .meslist .item.system .user{
  display: none;
}
.messagebox .meslist .item.system{
  justify-content: center;
}
.messagebox .meslist .item.system .systembox p{
  background: #E7E7E7;
border-radius: 4px;
border-radius: 4px;
padding: 5px 6px;
word-break: break-all;
font-size: 12px;
}

.messagebox .meslist .item.on .mes{
  background-color: #C5F7FF;
  border-color: #fb683c;
}


.messagebox .meslist .item .mes span.after{
  display: inline-block;
  width: 0;
  height: 0;
  position: absolute;
  top: 2px;
  border-left: 6px solid transparent;
  border-right: 10px solid #F5F5F5;
  border-top: 6px solid transparent;
  border-bottom: 6px solid transparent;
  right: inherit;
  left: -12px;
  right: inherit;
}

.messagebox .meslist .item.on .mes span.after{
  border-right: 6px solid transparent;
  border-left: 10px solid #C5F7FF;
  border-top: 6px solid transparent;
  border-bottom: 6px solid transparent;
  right: -12px;
  left: inherit;
  top: 10px;
}

</style>
