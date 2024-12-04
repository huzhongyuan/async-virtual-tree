
## Project setup
```
npm install
```

### Compiles and hot-reloads for development
```
npm run serve
```

### Compiles and minifies for production
```
npm run build
```

# async-virtual-list
1. 虚拟列表（只传入list参数）
2. 异步树不带checkbox（传入参数list、loadMore）
3. 异步树带checkbox（传入参数list、loadMore、isCheckbox）

# 前言
为啥自己写？目前使用较多的ElementUI等组件库均不支持大量数据的异步加载方式，致力于寻求一个更合适的大数据树的解决方案，用多了复杂全面的组件反而更偏爱单一性、轻量化的组件，初案方至。

[代码仓库](https://github.com/huzhongyuan/async-virtual-list)
***

## 代码

```
<asyncVirtualList
  ref="asyncVirtualList"
  :list="list"
  :loadMore="loadMore"
  isCheckbox
></asyncVirtualList>

// selType(非必填, checkbox开启时使用 1未勾选 2勾选 3半勾) 、 isLeaf(非必填、设置是否该节点可以展开)
list: [{ id: '1', name: '1',  selType: 1, isLeaf: true }],

// 展开加载数据
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

// isCheckbox（是否开启checkbox）

// 获取勾选结点数据
getLeafStatus() {
  const leafStatus = this.$refs.asyncVirtualList.getLeafStatus();
  console.log(leafStatus);
}

```

## 问题
### 1、异步树如何能获取到所有选择的节点？

所有勾选状态的叶子结点+半勾选状态的叶子结点的集合。后端通过半勾选状态的叶子结点可查询到该节点下的所有勾选的结点id。

getLeafStatus方法获取勾选数据格式如下
![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f85453ebe0d648d78ec12ee65b1282b6~tplv-k3u1fbpfcp-watermark.image?)

- chechKeyList-所有勾选的叶子结点的id集合
- halfCheckKeyList-所有半勾选状态的叶子结点id集合

### 2、后端如何返回数据
loadMore方法返回的数据需要后端计算子节点如下字段：
```
 {  
    id: id
    name: 名字
    isLeaf: 是否为叶子结点
    selType: 勾选状态 1 未勾选 2勾选 3半勾
 }

```
### 3、可使用功能
1. 虚拟列表（只传入list参数）
2. 异步树不带checkbox（传入参数list、loadMore）
3. 异步树带checkbox（传入参数list、loadMore、isCheckbox）

### 4、性能测试（主要为获取勾选节点数据的时间）
- 11w个结点 10w勾选

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/26d6b6baa9ad488b86ef1e26b2ecc78f~tplv-k3u1fbpfcp-watermark.image?)

感谢阅读、有优化或问题欢迎指出～




