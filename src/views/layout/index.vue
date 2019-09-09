
<template>
  <div v-if="curUserData.id" class="common-layout-index">
    <layout-header></layout-header>
    <div class="content-box">
      <navbar></navbar>
      <div class="content">
        <router-view></router-view>
      </div>
    </div>
    <layout-footer></layout-footer>
      <button class="query-btn" :class="{unread: hasUnRead}" @click="init">
        <i class="fi icon2zixunxiaoxi"></i>
        咨询消息
      </button>
      <div v-if="isShowMessage" class="mask">
      <div class="mesbox">
        <message ref="message"
        @hide="hide"
        @show="show"></message>
        <!-- 问题分类页面 -->
      </div>
    </div>
  </div>
</template>

<script>
import { getUnreadMsgCount } from '@/api/apis';
import header from './header.vue';
import footer from './footer.vue';
import navbar from './navbar.vue';
import message from '@/components/message.vue';

export default {
  name: 'landing-page',
  components: {
    message,
    layoutHeader: header,
    layoutFooter: footer,
    navbar,
  },
  data() {
    return {

      curMessageIndex: 0, // 当前聊天用户id
      isShowQuestion: false, // 显示问题页面
      params: {
        appkey: 'sfci50a7s3uzi',
        token: '',
        navi: '',

      },
      isShowMessage: false,
      hasUnRead: false, // 是否有未读消息
    };
  },
  mounted() {
    this.getUnreadMsgCount();
    setInterval(() => {
      this.getUnreadMsgCount();
    }, 30 * 1000);
  },
  methods: {
    // 获取未读消息
    async getUnreadMsgCount() {
      let res = await getUnreadMsgCount({ type: 'expert', userId: this.curUserData.id });
      if (res.data.data) {
        this.hasUnRead = true;
      } else {
        this.hasUnRead = false;
      }
    },
    async init() {
      if (!this.token) {
        this.$message('没有登录，去登录');
      } else {
        this.isShowMessage = true;
      }
      // this.$refs.message[0].init(this.params,this.userInfo);
      // this.$refs['message'].init();
    },
    hide() {
      this.isShowMessage = false;
    },
    show() {
      this.isShowMessage = true;
    },


  },
};
/* eslint-enable */
</script>
<style>

</style>
<style lang="scss" scoped>
  .mesbox{
    position: fixed;
    width: 100%;
    height: 100%;
    right: 0;
    bottom: 0;
    left: 50%;
    top:50%;
    transform: translate(-50%,-50%);
    overflow: hidden;
    max-width: 830px;
    max-height: 600px;
  }
  .content-box {
    position: relative;
    padding: 10px 10px 10px 230px;
    box-sizing: border-box;
    background-color: #F2F4F8;
    .content {
      min-height: 460px;
      @media(min-height: 800px) {
        min-height: calc(100vh - 251px);
      }
      background-color: #FFF;
    }
  }
  .query-btn {
    position: fixed;
    right: 0;
    bottom: 20%;
    text-align: center;
    background-color: rgba(51, 200, 223, .4);
    border-radius: 4px 0 0 4px;
    color: #fff;
    border: 0;
    outline: none;
    padding: 15px 10px;
    cursor: pointer;
    transition: all .3s;
    &:hover {
      background-color: rgba(51, 200, 223, .9)
    }
    &.unread {
      &:after {
        content: "";
        position: absolute;
        top: -5px;
        right: 0;
        width: 10px;
        height: 10px;
        border-radius: 50%;
        background-color: red;
      }
    }
    z-index: 9;
    i {
      display: block;
      font-size: 24px;
      text-align: center;
      margin-bottom: 2px;
    }
  }
</style>
