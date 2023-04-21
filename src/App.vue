<template>
  <div class="home">
    <div class="fixed">
      <div class="searchBox"  v-show="showSearch">
        <mises-search/>
      </div>
      <SpNavbar :links="navbarLinks" :active-route="router.currentRoute.value.path" />
    </div>
    <div class="page" :class="{'hasSearch': showSearch}">
      <router-view />
    </div>
    <MisesFooter />
  </div>
</template>

<script lang="ts">
import './scss/app.scss'
import { computed, onBeforeMount } from 'vue'
import { useRoute, useRouter } from 'vue-router'
import { useStore } from 'vuex'
import SpNavbar from './components/SpNav'
import MisesFooter from './components/MisesFooter'
import MisesSearch from './components/MisesSearch/MisesSearch.vue'
export default {
  components: { SpNavbar,MisesFooter, MisesSearch },
  setup() {
    // store
    let $s = useStore()
    // router
    let router = useRouter()
    const route = useRoute()
    // state
    let navbarLinks = [
      { name: 'Home', url: '/portfolio' },
      {
        name: 'BlockChain',
        children: [
          {
            name: 'View Holders',
            url: '/holders'
          },
          {
            name: 'View Blocks',
            url: '/blocks'
          },
          {
            name: 'View Txns',
            url: '/tx'
          }
        ]
      },
      {
        name: 'Validators',
        url: '/validators'
      }
    ]
    // computed
    let address = computed(() => $s.getters['common/wallet/address'])
    // lh
    onBeforeMount(async () => {
      await $s.dispatch('common/env/init')
      //router.push('portfolio')
    })
    const showSearch = computed(()=>{
      return !['/', '/portfolio'].includes(route.path)
    })
    return {
      navbarLinks,
      // router
      router,
      // computed
      address,
      showSearch
    }
  }
}
</script>

<style scoped lang="scss">
body {
  margin: 0;
}
.home{
  display: flex;
  flex-direction: column;
  min-height: 100vh;
  .page{
    flex: 1;
  }
}
.fixed{
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  z-index: 55;
}
.page{
  padding-top: 80px;
}

.searchBox{
  display: none;
  background: white;
}
@media (max-width: 768px) {
  .page{
    padding-top: 66px;
    &.hasSearch{
      padding-top: 120px;
    }
  }
  .searchBox{
    display: block;
    background: white;
    padding: 10px 0;
    border-bottom: 1px solid #eee;
  }
}
</style>
<style lang="scss">
.searchBox{
  .search input{
    border:  1px solid #eee;
  }
}
</style>