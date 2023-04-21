<template>
  <div class="search">
    <input type="text" placeholder="Search by Txn Hash/Block Height/Address" @keyup="getKeyUp" v-model="search" />
    <div class="search-btn" @click="getText">
      <img src="/images/index/search@2x.png" alt="" />
    </div>
  </div>
</template>

<script lang="ts">
import BigNumber from 'bignumber.js'
import { defineComponent, reactive, ref, watch } from 'vue';
import { useRoute } from 'vue-router';
export default defineComponent({
  setup() {
    const route = useRoute()
    const search = ref('')
    watch(() => route.path,() => { search.value = '' },{ immediate: true });
    return {
      search
    }
  },
  methods: {
    getKeyUp(val: KeyboardEvent) {
      if (val.code.toLocaleLowerCase() === 'enter') {
        this.getText()
      }
    },
    getText() {
      const hashLength = 64
      const addressLength = 44
      const blockHeight = (number: string) => !new BigNumber(number).isNaN()
      if (this.search.length === hashLength) {
        this.$router.push(`/tx/${this.search}`)
        return
      }
      if (this.search.length === addressLength) {
        this.$router.push(`/holders/${this.search}`)
        return
      }
      if (blockHeight(this.search)) {
        this.$router.push(`/block/${this.search}`)
      }
    },
  }
})
</script>

<style lang="scss">
.search {
  // margin: 22px auto 0;
  display: flex;
  justify-content: center;
  height: 44px;

  input {
    height: calc(100% + 1px);
    background: #ffffff;
    border-radius: 5px;
    width: 596px;
    outline: none;
    border: none;
    padding-left: 16px;
    font-size: 16px;
    font-family: 'hlt-55';
    position: relative;
    right: -6px;
    color: #16161d;

    &::placeholder {
      color: #ccc;
    }
  }

  .search-btn {
    border-radius: 5px;
    background: #5d61ff;
    width: 50px;
    height: 100%;
    display: flex;
    align-items: center;
    justify-content: center;
    cursor: pointer;

    img {
      width: 16px;
      height: 16px;
    }
  }
}

@media screen and (max-width: 768px) {

  .search {
    // margin: 22px auto 0;
    height: 32px;
    display: flex;
    justify-content: center;

    input {
      width: 80vw;
      height: calc(100% + 1px);
      padding-left: 8px;
      font-size: 12px;
    }

    .search-btn {
      width: 40px;
      height: 100%;

      img {
        width: 12px;
        height: 12px;
      }
    }
  }
}</style>