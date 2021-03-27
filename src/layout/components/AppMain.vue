<template>
  <section class="app-main">
    <transition name="component-fade" mode="out-in">
      <router-view :key="key" />
    </transition>
  </section>
</template>

<script>
export default {
  name: 'AppMain',
  computed: {
    key() {
      return this.$route.path
    }
  }
}
</script>

<style lang="scss" scoped>
@import "~@/styles/variables.scss";

.app-main {
  /*107 = navbar 50 + topbar 57  */
  height: calc(100vh - 107px);
  width: 100%;
  position: relative;
  overflow-y: auto;
}
.fixed-header+.app-main {
  padding-top: 50px;
}
</style>

<style lang="scss">
// fix css style bug in open el-dialog
.el-popup-parent--hidden {
  .fixed-header {
    padding-right: 15px;
  }
}
</style>
