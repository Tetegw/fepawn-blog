<template>
	<header>
		<div class="header" :class="{BAM: isBAM }" id="headerRippleWrap">
			<h1 @click="changeHash('blog')" class="iconfont icon-fepawn"></h1>
      <div class="headerR">
				<div class="search" v-show="searchIsShow" @mouseenter="showSearch" :class="{active: slideDown}">
					<i class="iconfont icon-sousuo"></i>
					<input type="text" placeholder="请输入搜索的内容" v-model="searchInfo" @keydown.stop.enter="searchSubmit" @blur="hideSearch">
				</div>
				<div class="logout ripple" @click="logout">
					<i class="iconfont icon-git46"></i>
					<div class="rippleWrap">
						<span></span>
					</div>
					<span class="Tooltip">退出</span>
				</div>
				<div class="person" :class="{activeType: activeRoute.indexOf('/BAM') > -1}" @click="changeHash('BAM')">
					<div class="info">
						<div class="avatar"><img :src="avatar" alt=""></div>
					</div>
					<span class="Tooltip">个人中心</span>
				</div>
			</div>
			<nav class="nav">
				<ul v-show="navListIsShow">
					<li class="ripple" :class="{activeType:(activeRoute ==='/blog' || activeRoute ==='/article')}" @click="changeHash('blog')">
						<i class="iconfont icon-home1149866easyiconnet"></i>
						<span class="info">博客</span>
						<div class="rippleWrap">
							<span></span>
						</div>
					</li>
					<li class="ripple" :class="{activeType:activeRoute ==='/time'}" @click="changeHash('time')">
						<i class="iconfont icon-emizhifeiji"></i>
						<span class="info">归档</span>
						<div class="rippleWrap">
							<span></span>
						</div>
					</li>
          <li class="ripple" :class="{activeType:activeRoute ==='/code'}" @click="changeHash('code')">
						<i class="iconfont icon-insert_tag_field"></i>
						<span class="info">代码段</span>
						<div class="rippleWrap">
							<span></span>
						</div>
					</li>
				</ul>
			</nav>
		</div>
		<v-Message :messageShow="messageShow" :sendMessage="sendMessage"></v-Message>
	</header>
</template>

<script>
import defaultAvatar from "@/assets/img/defaultIcon.gif";
import { ripple } from "@/assets/script/common";
import Message from "@/components/common/Message/Message";
import { bmobLogout, currentUser } from "@/bmob";

export default {
  data() {
    return {
      activeRoute: "/blog",
      searchInfo: "",
      isBAM: false,
      navListIsShow: true,
      searchIsShow: true,
      messageShow: false,
      sendMessage: "",
      slideDown: false,
      avatar: defaultAvatar,
      userID: ""
    };
  },
  watch: {
    $route(to, from) {
      this.activeRoute = to.path;
      // 判断是否是后台管理界面
      if (to.path.indexOf("/BAM") > -1) {
        this._initUserInfo();
        this.isBAM = true;
        this.navListIsShow = false;
        this.searchIsShow = false;
      } else {
        this.isBAM = false;
        this.navListIsShow = true;
        this.searchIsShow = true;
      }
      // 判断是否是login界面
      if (to.path.indexOf("/login") !== -1) {
        this.activeRoute = "/BAM";
      }
      // 判断是否是article界面，和博客相同，不需要修改
    }
  },
  methods: {
    _initUserInfo() {
      currentUser().then(
        result => {
          this.avatar = result.get("avatar");
          this.userID = result.id;
        },
        res => {}
      );
    },
    logout() {
      bmobLogout().then(
        result => {
          if (result.code === "000") {
            (this.avatar = defaultAvatar), (this.userID = "");
            let _this = this;
            let routeQuery = this.$route.query;
            let routePath = this.$route.path;
            if (routePath.indexOf("/BAM") > -1) {
              this.$router.push({ path: "/blog", query: routeQuery });
            }
            if (!this.messageShow) {
              this.messageShow = true;
              this.sendMessage = result.message;
              setTimeout(function() {
                _this.messageShow = false;
              }, 1500);
            }
          }
        },
        res => {
          if (res.code === "001") {
            let _this = this;
            if (!this.messageShow) {
              this.messageShow = true;
              this.sendMessage = res.message;
              setTimeout(function() {
                _this.messageShow = false;
              }, 1500);
            }
          }
        }
      );
    },
    changeHash(type) {
      if (type === "BAM") {
        this.$router.push({ path: "/" + type });
        return;
      }
      // BAM后台时点击logo，则添加上用户id
      if (this.isBAM && type === "blog") {
        this.$router.push({ path: "/blog", query: { userId: this.userID } });
        return;
      }
      // 登录外，根据url中userId处理
      const userId = this.$route.query.userId;
      if (userId === undefined) {
        this.$router.push({ path: "/" + type });
      } else {
        this.$router.push({ path: "/" + type, query: { userId: userId } });
      }
    },
    showSearch(e) {
      e.target.children[1].focus();
      this.slideDown = true;
    },
    hideSearch() {
      this.searchInfo = "";
      this.slideDown = false;
    },
    searchSubmit() {
      // 并且跳转至blog
      const userId = this.$route.query.userId;
      if (userId === undefined) {
        this.$router.push({ path: "/blog" });
      } else {
        this.$router.push({ path: "/blog", query: { userId: userId } });
      }
      // 把关键字传递给blog
      this.searchInfo = this.searchInfo.trim();
      this.$emit("searchInfo", this.searchInfo);
    }
  },
  mounted() {
    console.log("header mounted");
    this._initUserInfo();
    // 判断是否是BAM界面
    if (this.$route.path.indexOf("/BAM") > -1) {
      this.activeRoute = this.$route.path;
      this.isBAM = true;
      this.navListIsShow = false;
      this.searchIsShow = false;
    }
    // 判断是否是login界面
    if (this.$route.path.indexOf("/login") !== -1) {
      this.activeRoute = "/BAM";
    }
    // 判断是否是time界面
    if (this.$route.path.indexOf("/time") !== -1) {
      this.activeRoute = "/time";
    }
    if (this.$route.path.indexOf("/code") !== -1) {
      this.activeRoute = "/code";
    }
    ripple("headerRippleWrap");
  },
  components: {
    "v-Message": Message
  }
};
</script>

<style lang="less">
@import "../../../assets/style/common.less";
@m26a69a : #26a69a;
@m009688 : #009688;
.header {
  position: fixed;
  z-index: 990;
  top: 0;
  left: 0;
  right: 0;
  background-color: @m26a69a;
  height: 50px;
  color: #fff;
  box-shadow: 0 1px 5px rgba(0, 0, 0, 0.1);
  &.BAM {
    left: 200px;
    right: 0;
    background-color: #fff;
    color: #2f4050;
  }
  h1 {
    margin-left: 26px;
    float: left;
    font-size: 30px;
    line-height: 50px;
    width: 160px;
    cursor: pointer;
  }
  .nav {
    max-width: 1000px;
    margin: 0 auto;
    .rippleMixin(20px, 50px, rgba(255, 255, 255, 0.2), 200, nav);
    ul {
      .clearfixMixin();
      li {
        float: left;
        line-height: 50px;
        &.activeType,
        &:hover {
          background-color: rgba(0, 0, 0, 0.1);
        }
        i {
          display: inline-block;
          margin-right: 3px;
          font-size: 17px;
          line-height: 21px;
          vertical-align: middle;
        }
      }
    }
  }
  .headerR {
    float: right;
    margin-right: 20px;
    .search {
      float: left;
      width: 44px;
      height: 50px;
      overflow: hidden;
      background-color: @m26a69a;
      font-size: 0;
      padding: 12px 10px 0 10px;
      box-sizing: border-box;
      transition: all 0.5s;
      i {
        float: left;
        line-height: 26px;
        font-size: 18px;
      }
      .placeholder(#fff);
      input {
        float: right;
        width: 0;
        border: 0;
        font-size: 14px;
        height: 26px;
        box-sizing: border-box;
        color: #fff;
        background-color: transparent;
        transition: all 0.5s;
      }
      &.active {
        width: 200px;
        background-color: @m009688;
        input {
          width: 155px;
          padding-left: 10px;
        }
      }
    }
    .person {
      float: right;
      height: 50%;
      cursor: pointer;
      position: relative;
      transition: all 1s;
      .Tooltip {
        display: none;
      }
      &:hover {
        .TooltipMixin(70px, -45px, 0, 6px, 6px, 7px);
        .Tooltip {
          display: block;
        }
      }
      .info {
        width: 50px;
        height: 50px;
        padding: 8px;
        display: inline-block;
        box-sizing: border-box;
        vertical-align: top;
        .avatar {
          overflow: hidden;
        }
        img {
          height: 34px;
        }
      }
      &.activeType,
      &:hover {
        background-color: rgba(0, 0, 0, 0.1);
      }
    }
    .rippleMixin(0, 50px, rgba(255, 255, 255, 0.2), 200, logout);
    .logout {
      float: left;
      width: 50px;
      height: 50px;
      text-align: center;
      cursor: pointer;
      .Tooltip {
        display: none;
      }
      &:hover {
        .Tooltip {
          display: block;
        }
        background-color: rgba(0, 0, 0, 0.1);
      }
      .iconfont {
        font-size: 27px;
        line-height: 50px;
      }
      .TooltipMixin(50px, -45px, 0, 6px, 6px, 7px);
    }
  }
}

@media (max-width: 1000px) {
  .header {
    &.BAM {
      left: 0;
    }
    h1 {
      width: auto;
      margin-left: 15px;
    }
    .nav {
      position: inherit;
      left: 0;
      margin: 0 0 0 60px;
      ul {
        li.ripple {
          padding: 0 12px;
          i {
            margin-right: 0;
          }
        }
      }
    }
    .search {
      display: none;
    }
    .headerR {
      margin-right: 10px;
      .person {
        &:hover {
          .Tooltip {
            display: none;
          }
        }
        .info {
          width: auto;
        }
      }
      .logout {
        &:hover {
          .Tooltip {
            display: none;
          }
        }
      }
    }
  }
}
</style>
