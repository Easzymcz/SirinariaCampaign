<template>
  <div class="theme-container" :class="pageClasses">
    <Navbar v-if="shouldShowNavbar" @toggle-sidebar="toggleSidebar" />
    <div class="sidebar-mask" @click="toggleSidebar(false)"></div>
    <Sidebar :items="sidebarItems" @toggle-sidebar="toggleSidebar">
      <slot name="sidebar-top" slot="top" />
      <slot name="sidebar-bottom" slot="bottom" />
    </Sidebar>
    <Home v-if="$page.frontmatter.home" />

    <Page v-else :sidebar-items="sidebarItems" @>
      <slot name="page-top" slot="top" />
      <slot name="page-bottom" slot="bottom">
      <div class="page-nav" v-if="prev || next">
        <p class="inner">
          <span v-if="prev" class="prev">
            ←
            <router-link v-if="prev" class="prev" :to="prev.path">
              {{ prev.title || prev.path }}
            </router-link>
          </span>

          <span v-if="next" class="next">
            <router-link v-if="next" :to="next.path">
              {{ next.title || next.path }}
            </router-link>
            →
          </span>
        </p>
      </div>
      </slot>
    </Page>

  </div>
</template>

<script>
  import Home from '@theme/components/Home.vue'
  import Navbar from '@theme/components/Navbar.vue';
  import Page from '@theme/components/Page.vue'
  import Sidebar from '@theme/components/Sidebar.vue'
  import {
    resolveSidebarItems
  } from '@theme/util'
  export default {
    props: ["page"],
    components: {
      Home,
      Page,
      Sidebar,
      Navbar
    },
    data() {
      return {
        isSidebarOpen: false
      }
    },
    computed: {
      prev() {
        let currentPage = this.page ? this.page : this.$page;
        let prev = this.$site.pages
          .filter(x => {
            return x.frontmatter.chapter === currentPage.frontmatter.chapter - 1;
          });
        return prev[0];
      },
      next() {
        let currentPage = this.page ? this.page : this.$page;
        let next = this.$site.pages
          .filter(x => {
            return x.frontmatter.chapter === currentPage.frontmatter.chapter + 1;
          });
        return next[0];
      },
      shouldShowNavbar() {
        const {
          themeConfig
        } = this.$site
        const {
          frontmatter
        } = this.$page
        if (
          frontmatter.navbar === false ||
          themeConfig.navbar === false) {
          return false
        }
        return (
          this.$title ||
          themeConfig.logo ||
          themeConfig.repo ||
          themeConfig.nav ||
          this.$themeLocaleConfig.nav
        )
      },
      shouldShowSidebar() {
        const {
          frontmatter
        } = this.$page
        return (
          !frontmatter.home &&
          frontmatter.sidebar !== false &&
          this.sidebarItems.length
        )
      },
      sidebarItems() {
        return resolveSidebarItems(
          this.$page,
          this.$page.regularPath,
          this.$site,
          this.$localePath
        )
      },
      pageClasses() {
        const userPageClass = this.$page.frontmatter.pageClass
        return [{
            'no-navbar': !this.shouldShowNavbar,
            'sidebar-open': this.isSidebarOpen,
            'no-sidebar': !this.shouldShowSidebar
          },
          userPageClass
        ]
      }
    },
    mounted() {
      this.$router.afterEach(() => {
        this.isSidebarOpen = false
      })
    },
    methods: {
      toggleSidebar(to) {
        this.isSidebarOpen = typeof to === 'boolean' ? to : !this.isSidebarOpen
      },
      // side swipe
      onTouchStart(e) {
        this.touchStart = {
          x: e.changedTouches[0].clientX,
          y: e.changedTouches[0].clientY
        }
      },
      onTouchEnd(e) {
        const dx = e.changedTouches[0].clientX - this.touchStart.x
        const dy = e.changedTouches[0].clientY - this.touchStart.y
        if (Math.abs(dx) > Math.abs(dy) && Math.abs(dx) > 40) {
          if (dx > 0 && this.touchStart.x <= 80) {
            this.toggleSidebar(true)
          } else {
            this.toggleSidebar(false)
          }
        }
      }
    }
  };
</script>
<style src="prismjs/themes/prism-tomorrow.css"></style>
<style src="@theme/styles/theme.styl" lang="stylus"></style>
<style lang="stylus">
.page-nav
  padding-top 1rem
  padding-bottom 0
  .inner
    min-height 2rem
    margin-top 0
    border-top 1px solid $borderColor
    padding-top 1rem
    overflow auto // clear float
  .next
    float right
</style>
