---
title: ztree的模糊搜索过滤功能
comments: true
toc: true
description: ztree的模糊搜索过滤功能
top_img: 'https://gitee.com/gsshy/picgo/raw/master/img/top.jpg'
categories:
  - vue
tags:
  - ztree
  - 模糊搜索
  - 工作
abbrlink: 5a57df06
date: 2020-10-21 16:00:00
---
# ztree的模糊搜索过滤功能

## 1.问题

加一个查询框可以模糊过滤

![image-20210910150846172](https://gitee.com/gsshy/picgo/raw/master/img/image-20210910150846172.png)

## 2.实现代码

```html
//搜索框
<el-input v-model="itemTypeName" placeholder="请输入设备类别名称" size="mini" clearable @keyup.enter.native="onSearch"/>
        <el-scrollbar style="height: 35vh">
          <tree :nodes="itemTypeTreeData" node-key="itemTypeId" :setting="settingFixType" @onClick="fixTreeClick"
                @onCreated="handleCreated"/>
        </el-scrollbar>
```

```js
handleCreated: function (ztreeObj) {
      this.ztreeObj = ztreeObj
      this.zTree = ztreeObj
	  
      this.allNodes = ztreeObj.transformToArray(ztreeObj.getNodes())
      ztreeObj.expandNode(ztreeObj.getNodes()[0], true, false, true)
    },
onSearch () {
      let that = this
      let zTreeObj = that.zTree
      if (that.itemTypeName === "") {
          //注意！！！showNodes时that.allNodes是getNodes转数组之后的，但是expandNode是直接用的getNodes()[0]
        zTreeObj.showNodes(that.allNodes)
        zTreeObj.expandAll(false)
        zTreeObj.expandNode(zTreeObj.getNodes()[0], true, false, true)
        return
      }
      that.nodeList = zTreeObj.getNodesByParamFuzzy("itemTypeName", that.itemTypeName, null)
      that.updateNodes(that.nodeList)
    },
    updateNodes (nodeList) {
      let that = this
      let zTreeObj = that.zTree
      let allNode = zTreeObj.transformToArray(zTreeObj.getNodes())
      zTreeObj.hideNodes(allNode)
      for (let n in nodeList) {
        that.findParent(zTreeObj, nodeList[n])
      }

      zTreeObj.showNodes(nodeList)
    },
    findParent (zTree, node) {
      let that = this
      zTree.expandNode(node, true, false, false)
      let pNode = node.getParentNode()
      if (pNode != null) {
        that.nodeList.push(pNode)
        that.findParent(zTree, pNode)
      }
    },
```

