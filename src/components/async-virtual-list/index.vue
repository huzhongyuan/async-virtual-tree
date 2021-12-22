<template>
  <div ref="list" class="infinite-list-container" :style="{height: height + 'px'}" @scroll="scrollEvent($event)">
    <div class="infinite-list-phantom" :style="{ height: listHeight + 'px' }"></div>
    <div class="infinite-list" :style="{ transform: getTransform }">
      <div ref="items"
        class="infinite-list-item" 
        v-for="(item, index) in visibleData" 
        :key="item.id"
        @click.stop="lineSelect(item, index)"
        :style="{ height: itemSize + 'px',lineHeight: itemSize + 'px', paddingLeft: itemPadding(item.id) + 'px' }"
      >
        <div v-if="isLowerLevel" @click.stop="toggleExpand(item, index)" class="tree-expand-wrapper">
          <div v-if="!item.isLeaf">
            <div
                v-if="!item.loadNode"
                :class="{ 'tree-icon-expand': true , 'rotate90': item.expand,'clickAble': !item.expand }"
                icon
                pointer
            ></div>
            <div v-else class="spinner">
            </div>
          </div>
          <div v-else class="no-expand"></div>
        </div>
        <div v-if="isCheckbox" @click.stop="toggleCheck(item, index)" :class="{
            'no-check': item.selType === 1 || !item.selType,
            'check': item.selType === 2,
            'half-check': item.selType === 3 
          }">
          <div class="check-graphical" v-if="item.selType === 2"></div>
          <div class="half-check-graphical" v-if="item.selType === 3"></div>
        </div>
        <span>{{ item.name }}</span>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name:'VirtualList',
  props: {
    // 列表高度
    height: {
      type: Number,
      default: 300
    },
    //所有列表数据
    list:{
      type:Array,
      default:()=>[]
    },
    //每项高度
    itemSize: {
      type: Number,
      default: 28
    },
    // id分隔符
    separator: {
      type: String,
      default: '-'
    },
    // 是否有checkbox
    isCheckbox: {
      type: Boolean,
      default: false
    },
    // 是否展示下级
    isLowerLevel: {
      type: Boolean,
      default: true
    },
    // 是否通过点击展开下层节点
    isClickSelectNode: {
      type: Boolean,
      default: true
    },
    // 异步加载
    loadMore: Function
  },
  computed:{
    //列表总高度
    listHeight(){
      return this.listData.length * this.itemSize;
    },
    //可显示的列表项数
    visibleCount(){
      return Math.ceil(this.screenHeight / this.itemSize)
    },
    //偏移量对应的style
    getTransform(){
      return `translate3d(0,${this.startOffset}px,0)`;
    },
    // listData: {
    //   get:function () {
    //     return this.list;
    //   },
    //   set: function (v) {
    //     return v;
    //   }
    // },  
    //获取真实显示列表数据
    visibleData(){
      return this.listData.slice(this.start, Math.min(this.end,this.listData.length));
    },
  },
  watch: {
    list: function (newVal, oldVal) {
      if (newVal.length !== 0 && typeof(oldVal) === 'object' && oldVal.length === 0) {
        newVal.forEach((item) => {
          this.originNodeMap.set(item.id, item);
        })
        this.listData = [].concat(newVal);
      }
    }
  },
  mounted() {
    this.start = 0;
    this.end = this.start + this.visibleCount;
  },
  data() {
    return {
      //可视区域高度
      screenHeight: document.body.clientHeight,
      //偏移量
      startOffset:0,
      //起始索引
      start:0,
      //结束索引
      end:null,
      listData: [],
      originNodeMap: new Map(),
      currentNodeMap: {}
    };
  },
  methods: {
    scrollEvent() {
      //当前滚动位置
      let scrollTop = this.$refs.list.scrollTop;
      //此时的开始索引
      this.start = Math.floor(scrollTop / this.itemSize);
      //此时的结束索引
      this.end = this.start + this.visibleCount;
      //此时的偏移量
      this.startOffset = scrollTop - (scrollTop % this.itemSize);
    },
    // 每一列padding
    itemPadding(id) {
      return id.split(this.separator).length * 10
    },
    // 展开某一行
    lineSelect(item, index) {
      if (this.isClickSelectNode) {
        this.toggleExpand(item, index);
      }
    },
    // 点击节点
    toggleExpand(item, index) {
      if (item.expand) {
        // 收纳节点
        this.storageNode(item, index);
      } else {
        // 展开节点
        this.unfoldNode(item, index);
      }
    },
    // 收纳节点
    storageNode(item, index){
      console.log(index);
      // 设置所有子节点expand为false
      const childList = this.listData.filter(node => node.id.startsWith(item.id) && node.id[item.id.length] === this.separator);
      childList.forEach(childItem => {
        this.changeAttributes(this.originNodeMap, childItem.id, 'expand', false);
      })
      this.listData = [].concat(this.listData.filter(node => node.id === item.id || !node.id.startsWith(item.id) || node.id[item.id.length] !== this.separator));
      this.$set(item, 'expand', !item.expand && true);
    },
    // 展开节点
    unfoldNode(item, index) {
      // 如果该节点正在展开/或者属于叶子结点直接返回
      if (item.loadNode || item.isLeaf) return;
      if (("expanded" in this.originNodeMap.get(item.id))) {
        const startIndex = this.start;
        const cacheArrStart = this.listData.slice(0, startIndex + index + 1);
        const cacheArrEnd = this.listData.slice(startIndex + index + 1);
        // 如果已经展开过子节点
        const keysList = [...this.originNodeMap.keys()];
        const childKeys = keysList.filter(keyItem => keyItem.startsWith(item.id) && keyItem[item.id.length] === this.separator && (keyItem.split(this.separator).length - item.id.split(this.separator).length === 1));
        let childrenList = [];
        childKeys.forEach((childKeysItem) => {
          childrenList.push(this.originNodeMap.get(childKeysItem))
        })
        this.listData = []
          .concat(cacheArrStart)
          .concat(childrenList)
          .concat(cacheArrEnd);
        this.$set(item, 'expand', !item.expand && true);
      } else {
        // 记录是否已经展开过
        // 进入loading状态
        this.$set(item, 'loadNode', !item.loadNode && true);  
        const itemData = item;
        this.loadMore(itemData).then((nodeChildList) => {
          // 设置为此id已经展开
          this.changeAttributes(this.originNodeMap, itemData.id, 'expanded', true);
          nodeChildList.forEach(nodeChildItem => { nodeChildItem.parentId = item.id })
          if (nodeChildList.length === 0) {
            // 是否叶子节点
            this.$set(itemData, 'isLeaf', true)
          } else {
            // 1、如果该节点为半勾选状态则子节点的勾选状态是根据返回的数据来 2、如果该节点为勾选状态则返回的子节点全为勾选状态  3、如果改节点为为勾选状态则子节点全部未勾选
            if (item.selType === 1 || !item.selType) {
              // 该节点未勾选-所有子节点均不勾选
              nodeChildList.forEach((nodeChildItem) => {
                nodeChildItem.selType = 1;
              })
            } else if (item.selType === 2) {
              // 该节点勾选-所有子节点均勾选
              nodeChildList.forEach((nodeChildItem) => {
                nodeChildItem.selType = 2;
              })
            }
            // 将新节点加入map
            nodeChildList.forEach(childrenItem => {
              this.originNodeMap.set(childrenItem.id, childrenItem);
            })
            // 找到索引 如果该节点已经收起则不更新数据
            if (this.listData.filter(childrenItem => childrenItem.id === itemData.id).length !== 0) {
              const currentIndex = this.listData.map(idItem => idItem.id).indexOf(itemData.id);
              const cacheArrStart = this.listData.slice(0, currentIndex + 1);
              const cacheArrEnd = this.listData.slice(currentIndex + 1);
              this.listData = []
              .concat(cacheArrStart)
              .concat(nodeChildList)
              .concat(cacheArrEnd);
              itemData.expand = !itemData.expand && true;
            }
          }
        }).finally(() => {
          this.$set(itemData, 'loadNode', false);
          // this.$forceUpdate();
        })
      }
    },
    // 修改某个属性的值 map的值和list的item指向同一个堆所以只用更改一次
    changeAttributes(map, mapkey, key, value){
      let data = map.get(mapkey);
      data[key] = value;
      // map.set(mapkey, data);
    },
    // 勾选 selType: 1未勾选 2勾选 3半勾选
    toggleCheck(item) {
      const setChechFn = (selType) => {
        this.$set(item, 'selType', selType);
        // 处理子节点勾选状态
        this.setChildToggleCheck(item, selType);
        // 处理父节点勾选状态
        this.setParentToggleCheck(item);
      };
      const actions = {
        '1': () => {
          // 未勾选变为勾选
          setChechFn(2);
        },
        '2': () => {
          // 勾选变为未勾选
          setChechFn(1);
        },
        '3': () => {
          // 半勾选变为勾选
          setChechFn(2);
        }
      }
      const action = actions[item.selType] || actions['1'];
      action();
    },
    // 改变所有子节点的勾选状态
    setChildToggleCheck(item, selType) {
      // 获取所有子节点key
      const keysList = [...this.originNodeMap.keys()];
      const childKeys = keysList.filter(keyItem => keyItem.startsWith(item.id) && keyItem[item.id.length] === this.separator);
      childKeys.forEach((childKeysItem) => {
        this.changeAttributes(this.originNodeMap, childKeysItem, 'selType', selType);
      })
    },
    // 设置父节点的勾选状态
    setParentToggleCheck(item) {
      let pid = item.id;
      const number = pid.split(this.separator).length - 1;
      for (let i = 0; i <  number; i ++) {
        if (pid.lastIndexOf(this.separator) > -1);
          // 截取每次父节点的值
          pid = pid.slice(0, pid.lastIndexOf(this.separator));
          // 获取该节点下级子节点
          const keysList = [...this.originNodeMap.keys()];
          const childKeys = keysList.filter(keyItem => keyItem.startsWith(pid) && keyItem.split(this.separator).length - pid.split(this.separator).length === 1);
          let [ checkNumber, halfCheckNumber ] = [0, 0];
          childKeys.forEach(childKeysItem => {
            if (this.originNodeMap.get(childKeysItem).selType === 2 ) {
              checkNumber = checkNumber + 1;
            } else if (this.originNodeMap.get(childKeysItem).selType === 3) {
              halfCheckNumber = halfCheckNumber + 1;
            }
          })
          if (childKeys.length === checkNumber) {
            // 全选
            this.changeAttributes(this.originNodeMap, pid, 'selType', 2);
          } else if (checkNumber === 0 && halfCheckNumber === 0) {
            // 全没选
            this.changeAttributes(this.originNodeMap, pid, 'selType', 1);
          } else {
            this.changeAttributes(this.originNodeMap, pid, 'selType', 3);
          }
      }
    },
    // 获取所有叶子结点选择状态
    getLeafStatus() {
      // 筛选叶子结点
      console.time();
      const keysList = [...this.originNodeMap.keys()];
      const list = new Set();
      keysList.forEach(key => {
        if (this.originNodeMap.get(key).parentId) {
          list.add(this.originNodeMap.get(key).parentId);
        }
      })
      const parentIdList = Array.from(list);
      const leafKeyList = keysList.concat(parentIdList).filter(v => !parentIdList.includes(v));
      let [chechKeyList, halfCheckKeyList] = [[], []];
      leafKeyList.forEach(key => {
        const obj = this.originNodeMap.get(key);
        if (obj.selType === 2) {
          chechKeyList.push(key);
        } else if (obj.selType === 3) {
          halfCheckKeyList.push(key);
        }
      })
      console.timeEnd();
      return {
        chechKeyList,
        halfCheckKeyList
      };
    }
  }
};
</script>


<style scoped >
.infinite-list-container {
  overflow: auto;
  position: relative;
  -webkit-overflow-scrolling: touch;
}

.infinite-list-phantom {
  position: absolute;
  left: 0;
  top: 0;
  right: 0;
  height: 100%;
  z-index: -1;
}

.infinite-list {
  left: 0;
  right: 0;
  top: 0;
  /* position: absolute; */
  text-align: left;
}

.infinite-list-item {
  color: #555;
  box-sizing: border-box;
  display: flex;
  align-items: center;
  /* border-bottom: 1px solid #999; */
}
.infinite-list-item:hover {
  background-color: rgb(242, 243, 245);
  cursor: pointer;
  border-radius: 2px;
}
.infinite-list-item .tree-expand-wrapper {
    display: inline-block;
    padding-right: 0px;
    cursor: pointer;
    width: 10px;
    margin-right: 5px;
}

.infinite-list-item .tree-expand-wrapper .tree-icon-expand {
    vertical-align: middle;
    transition-duration: 0.3s;
    width: 0;
    height: 0;
    border: 5px solid transparent;
    border-left: 5px solid #4E5969;
    color: rgb(29, 33, 41);
}

.infinite-list-item .tree-expand-wrapper .no-expand {
    width: 10px;
    border: 0;
    display: inherit;
}
.infinite-list-item .tree-expand-wrapper .clickAble {
    cursor: pointer;
}
.infinite-list-item .tree-expand-wrapper .rotate90 {
    transform: rotate(90deg) translate(3px,3px);
}
.infinite-list-item .tree-expand-wrapper .no-expand {
  
}
.infinite-list-item .tree-expand-wrapper .spinner {
  width: 8px;
  height: 8px;
  background-color: rgb(29, 33, 41);
  margin: 0px auto;
  -webkit-animation: sk-rotateplane 1.2s infinite ease-in-out;
  animation: sk-rotateplane 1.2s infinite ease-in-out;
  margin-right: 12px;
  margin-top: 0px;
}

@-webkit-keyframes sk-rotateplane {
  0% { -webkit-transform: perspective(12px) }
  50% { -webkit-transform: perspective(12px) rotateY(180deg) }
  100% { -webkit-transform: perspective(12px) rotateY(180deg)  rotateX(180deg) }
}

@keyframes sk-rotateplane {
  0% { 
    transform: perspective(12px) rotateX(0deg) rotateY(0deg);
    -webkit-transform: perspective(12px) rotateX(0deg) rotateY(0deg) 
  } 50% { 
    transform: perspective(12px) rotateX(-180.1deg) rotateY(0deg);
    -webkit-transform: perspective(12px) rotateX(-180.1deg) rotateY(0deg) 
  } 100% { 
    transform: perspective(12px) rotateX(-180deg) rotateY(-179.9deg);
    -webkit-transform: perspective(12px) rotateX(-180deg) rotateY(-179.9deg);
  }
}
.infinite-list-item >div {
  display: inline-block;
  margin-right: 8px;
  width: 12px;
  height: 12px;
  border-radius: 2px;
  display: flex;
  align-items: center;
  justify-content: center;
}

.infinite-list-item .no-check {
  border: 1px solid rgb(229, 230, 235);
  background-color: rgb(255, 255, 255);
}

.infinite-list-item .check {
  width: 14px;
  height: 14px;
  background-color: rgb(22, 93, 255);
}
.infinite-list-item .check .check-graphical {
  width: 3px;
  height: 6px;
  border-right: 2px solid white;
  border-bottom: 2px solid white;
  transform: rotate(40deg);
}
.infinite-list-item .half-check {
  width: 14px;
  height: 14px;
  background-color: rgb(22, 93, 255);
}

.infinite-list-item .half-check .half-check-graphical {
  width: 7px;
  height: 2px;
  border-radius: 5px;
  background-color: white;
}
</style>