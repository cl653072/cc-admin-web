<template>
  <q-layout view="lhh lpR lff">
    <q-header class="bg-primary text-white">
      <q-toolbar>
        <q-btn flat icon="menu" @click="drawer=!drawer" />
        <q-toolbar-title class="text-caption">欢迎进入 cc-admin 企业级快速开发平台</q-toolbar-title>
        <q-btn
          flat
          size="md"
          label="QQ群：965940297"
          type="a"
          icon="mdi-qqchat"
          @click="copyNum"
          target="__blank" >
          <q-tooltip>点击复制QQ群号，交流反馈！</q-tooltip>
        </q-btn>
        <q-btn
          flat
          size="md"
          label="Sika Design"
          type="a"
          icon="img:icons/logo/sikacode-logo.png"
          href="http://quasar.sikacode.com/"
          target="__blank" >
          <q-tooltip>Sika Design精美漂亮的Quasar后台</q-tooltip>
        </q-btn>
        <q-btn
          flat
          size="md"
          type="a"
          label="Quasar中文网"
          href="http://www.quasarchs.com/"
          icon="img:icons/logo/quasar-logo.svg"
          target="__blank" >
          <q-tooltip>最好的quasar中文帮助文档</q-tooltip>
        </q-btn>

        <q-btn flat round size="sm" icon="mdi-card-search-outline" @click="drawer=!drawer" >
          <q-tooltip>全局搜索</q-tooltip>
        </q-btn>
        <q-btn flat round size="sm" icon="mdi-help-circle-outline" @click="drawer=!drawer" >
          <q-tooltip>帮助文档</q-tooltip>
        </q-btn>
        <q-btn flat round size="sm" icon="mdi-bell-outline" @click="drawer=!drawer" >
          <q-badge
            color="negative"
            style="padding: 2px 4px"
            title-color="white"
            floating
          >8
          </q-badge>
          <q-tooltip>未读消息</q-tooltip>
        </q-btn>
        <q-btn icon="mdi-badge-account-horizontal" flat :label="welcomeInfo">
          <q-menu transition-show="rotate" transition-hide="rotate">
            <q-list style="min-width: 100px">
              <q-item clickable>
                <q-item-section>个人中心</q-item-section>
              </q-item>
              <q-item clickable>
                <q-item-section>账户设置</q-item-section>
              </q-item>
              <q-separator />
              <q-item clickable>
                <q-item-section>密码修改</q-item-section>
              </q-item>
            </q-list>
          </q-menu>
        </q-btn>
        <q-btn flat icon="mdi-power-standby" label="退出系统" @click="out" />
      </q-toolbar>
      <mainTabs :tabs="$store.state.Rule.routeTabs" @selectPanel="setDesTab" />
    </q-header>
    <q-drawer elevated v-model="drawer" behavior="desktop" :width="220" content-class="column">
      <q-btn flat class="ly-icon-btn bg-primary text-white">
        <q-icon name="mdi-bike-fast" class="q-mr-md" />cc-admin
        <q-tooltip anchor="center right" self="center left">欢迎使用</q-tooltip>
      </q-btn>
      <q-scroll-area class="col main-menu">
        <q-list v-for="u in $store.state.Rule.routes" :key="u.id">
          <q-expansion-item
            expand-separator
            v-if="u.children&&u.children.filter(v=>!v.hidden)"
            :icon="u.meta.icon"
            :label="u.meta.title"
            :header-style="headerStyleActive(u)"
            :value="openMenu(u)"
            group="mainitem"
          >
            <q-expansion-item
              :header-inset-level="1.2"
              :label="s.meta.title"
              expand-icon-class="hide-icon"
              v-for="s in u.children&&u.children.filter(v=>!v.hidden)"
              :key="s.id"
              @click="go(s)"
              clickable
              :style="itemStyleActive(s)"
            />
          </q-expansion-item>
          <q-expansion-item
            v-else
            :icon="u.meta.icon"
            :label="u.meta.title"
            expand-icon-class="hide-icon"
            @click="go(u)"
            :style="itemStyleActive(u)"
            group="mainitem"
          />
        </q-list>
      </q-scroll-area>
    </q-drawer>

    <q-page-container>
      <keep-alive exclude="ServerInfo">
        <router-view />
      </keep-alive>
    </q-page-container>
  </q-layout>
</template>

<script>
import { openURL, copyToClipboard } from 'quasar';
import { goLogin } from 'boot/api';
import mainTabs from 'components/tabpanel/index.vue';

export default {
  name: 'IndexLayout',
  components: {
    mainTabs,
  },
  data() {
    return {
      selectItem: '',
      link: 'inbox',
      drawer: true,
      list: [],
    };
  },
  methods: {
    copyNum() {
      copyToClipboard('965940297')
        .then(() => {
          this.$info('复制成功，欢迎进群');
        })
        .catch(() => {
          this.$error('复制失败！群号是：965940297，手工复制吧!');
        });
    },
    setDesTab(val) {
      this.selectItem = val;
    },
    out() {
      this.$axios.get('/sys/logout').then(() => {
      }).finally(() => {
        goLogin();
      });
    },
    go(u) {
      if (u.internalOrExternal) {
        openURL(u.path);
      } else if (u.path !== this.$route.path) { this.$router.push(u); }
    },
    headerStyleActive(item) {
      if (this.$route.path.startsWith(item.path)) {
        return {
          color: '#1890ff',
        };
      }
      return {};
    },
    openMenu(item) {
      if (this.$route.path.startsWith(item.path)) {
        return true;
      }
      return false;
    },
    itemStyleActive(item) {
      if (item.path === this.$route.path) {
        return {
          color: '#1890ff',
          backgroundColor: '#e6f7ff',
          borderRight: '0.2em solid #1890ff',
        };
      }
      return {};
    },
  },
  watch: {
    $route(to) {
      if (!this.$store.state.Rule.routeTabs.some((r) => r.path === to.path)) {
        this.$store.commit('Rule/addRouteTabs', { name: to.meta.title, path: to.path });
      }
    },
  },
  computed: {
    welcomeInfo() {
      return `欢迎您，${this.$store.state.User.info.realname}`;
    },
  },
};
</script>

<style lang="stylus">
.cc-active-menu
  border-right '0.2em solid #1890ff'
.hide-icon {
  display: none;
}

.ly-icon-btn {
  line-height: 50px;
  height: 50px;
  border-bottom: 1px $sky solid;
  border-radius: 0;
}

.q-item__section--main ~ .q-item__section--side {
  padding-left: 0px;
}

.main-menu {
  .q-item {
    min-height: 35px;
  }
}
</style>
