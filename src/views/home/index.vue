<template>
  <div class="home-container">
    <!-- 导航栏 -->
    <van-nav-bar class="page-nav-bar" fixed>
      <van-button
        class="search-btn"
        slot="title"
        type="info"
        size="small"
        round
        icon="search"
        to="/search"
        >搜索</van-button
      >
     <template #right>
    <van-icon name="search" size="18" @click="isChennelEditShow = true,left='right'"/>
  </template>
       <template #left>
    <van-icon name="wap-nav" size="18" @click="isChennelEditShow = true,left='left'"/>
  </template>
    </van-nav-bar>
    <!-- /导航栏 -->

    <!-- 频道列表 -->
    <!--
      通过 v-model 绑定当前激活标签对应的索引值，默认情况下启用第一个标签
      通过 animated 属性可以开启切换标签内容时的转场动画
      通过 swipeable 属性可以开启滑动切换标签页
     -->
    <van-tabs class="channel-tabs" v-model="active" animated swipeable>
      <van-tab
        :title="channel.name"
        v-for="channel in channels"
        :key="channel.id"
      >
        <!-- {{channel.name}} -->
        <!-- 文章列表 -->
        <article-list ref="article-list" :channel="channel" />
        <!-- 文章列表 -->
      </van-tab>
      <div slot="nav-right" class="placeholder"></div>
      <div
        slot="nav-right"
        class="hamburger-btn"
        @click="isChennelEditShow = true ,left='bottom'"
      >
        <i class="toutiao toutiao-gengduo"></i>
      </div>
    </van-tabs>
    <!-- /频道列表 -->

    <!-- 频道编辑弹出层 -->
    <van-popup
      v-model="isChennelEditShow"
      closeable
      close-icon-position="top-right"
      :position="left"
      :style="{ height: '100%' }"
    >
      <channel-edit
        :my-channels="channels"
        :active="active"
        @update-active="onUpdateActive"
      />
    </van-popup>
    <!-- /频道编辑弹出层 -->
  </div>
</template>

<script>
import { getUserChannels } from "@/api/user";
import ArticleList from "./components/article-list";
import ChannelEdit from "./components/channel-edit";
import { mapState } from "vuex";
import { getItem, setItem } from "@/utils/storage";

export default {
  name: "HomeIndex",
  components: {
    ArticleList,
    ChannelEdit,
  },
  props: {},
  data() {
    return {
      active: 0,
      channels: [], // 频道列表
      isChennelEditShow: false, // 控制编辑频道弹出层的显示状态
      left:"bottom"
    };
  },
  computed: {
    ...mapState(["user"]),
  },
  watch: {},
  created() {
    this.loadChannels();
  },
  mounted() {},
  methods: {
    async loadChannels() {
      try {
        // const { data } = await getUserChannels()
        // this.channels = data.data.channels
        let channels = [];

        // 如果登陆过返回带有token的关注频道，没登录过就返回不带token的默认频道
        if (this.user) {
          // 已登录，请求获取用户频道列表
          const localChannels = getItem("USER_TOUTIAO_CHANNELS");
          //    有，拿来使用
          if (localChannels) {
            channels = localChannels;
          } else {
            //    没有，请求获取用户频道列表
            const { data } = await getUserChannels();
            channels = data.data.channels;
            setItem("USER_TOUTIAO_CHANNELS", channels);
          }
        } else {
          // 未登录，判断是否有本地的频道列表数据
          const localChannels = getItem("TOUTIAO_CHANNELS");
          //    有，拿来使用
          if (localChannels) {
            channels = localChannels;
          } else {
            //    没有，请求获取默认频道列表
            const { data } = await getUserChannels();
            channels = data.data.channels;
            setItem("TOUTIAO_CHANNELS", channels);
          }
        }

        this.channels = channels;
      } catch (err) {
        this.$toast("获取频道数据失败");
      }
    },

    onUpdateActive(index, isChennelEditShow = true) {
      // 更新激活的频道项
      this.active = index;

      // 关闭编辑频道弹层
      this.isChennelEditShow = isChennelEditShow;
    },
  },
};
</script>

<style scoped lang="less">
.home-container {
  padding-top: 174px;
  padding-bottom: 100px;
  /deep/ .van-nav-bar__title {
    min-width: 60%;
  }
  .search-btn {
    width: 100%;
    height: 64px;
    background-color: #5babfb;
    border: none;
    font-size: 28px;
    .van-icon {
      font-size: 32px;
    }
  }

  /deep/ .channel-tabs {
    .van-tabs__wrap {
      position: fixed;
      top: 92px;
      z-index: 1;
      left: 0;
      right: 0;
      height: 82px;
    }

    .van-tab {
      border-right: 1px solid #edeff3;
      min-width: 200px;
      font-size: 30px;
      color: #777777;
    }

    .van-tab--active {
      color: #333333;
    }

    .van-tabs__nav {
      padding-bottom: 0;
    }

    .van-tabs__line {
      bottom: 8px;
      width: 31px !important;
      height: 6px;
      background-color: #3296fa;
    }

    .placeholder {
      flex-shrink: 0;
      width: 66px;
      height: 82px;
    }

    .hamburger-btn {
      position: fixed;
      right: 0;
      display: flex;
      justify-content: center;
      align-items: center;
      width: 66px;
      height: 82px;
      background-color: #fff;
      background-color: rgba(255, 255, 255, 0.902);
      i.toutiao {
        font-size: 33px;
      }
      &:before {
        content: "";
        position: absolute;
        left: 0;
        width: 1px;
        height: 58px;
        background-image: url(~@/assets/gradient-gray-line.png);
        background-size: contain;
      }
    }
  }
}
</style>