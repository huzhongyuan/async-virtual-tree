<!--  -->
<template>
  <div class="">
    <div @click="getLeafStatus">
      获取子节点的状态
    </div>
    <asyncVirtualList
      ref="asyncVirtualList"
      :list="list"
      :loadMore="loadMore"
      isCheckbox
    ></asyncVirtualList>
  </div>
</template>

<script>
import asyncVirtualList from './async-virtual-list/index.vue';
export default {
  name: '',
  components: {
    asyncVirtualList
  },
  props: {},
  data () {
    return {
      list: [],
      separator: '-'
    }
  },
  mounted () {
    this.list = [];
    for (let i = 0; i < 10000; i++) {
      this.list.push({ id: i + '', name: i + '',  selType: i % 3 + 1});
    }
  },
  methods: {
    loadMore(nodeData) {
      return new Promise((resolve) => {
        setTimeout(() => {
            const list = [];
            for (let i = 0; i < 10000; i ++) {
              list.push({
                id: nodeData.id + '-' + (i + 1) + '', 
                name: nodeData.id + '-' + (i + 1), 
                isLeaf: Math.random(0, 1) > 0.8 ? true : false, 
                selType: i % 3 + 1
              })
            }
            resolve(list);
        }, 2000)
      })
    },

    // 提交数据
    getLeafStatus() {
      const leafStatus = this.$refs.asyncVirtualList.getLeafStatus();
      console.log(leafStatus);
    }
  },
}

</script>

<style>
html{
  height: 100%;
}
body{
  height: 100%;
  margin:0;
}
#app{
  height:100%;
}
</style>
