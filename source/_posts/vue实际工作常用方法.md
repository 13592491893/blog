---
title: vue实际工作常用方法
comments: true
toc: true
description: vue实际工作常用方法
top_img: https://gitee.com/gsshy/picgo/raw/master/img/top.jpg
categories:
  - vue
tags:
  - 前端
  - vue
  - 工作
date: 2020-10-25 16:00:00
---
# 1.select选择框获取key和value

``` vue
<el-select v-model="form.deviceType" value-key="value"  @change="getDeviceTypeName" 			:disabled="this.form.status == 'change'? true:false">
    <el-option v-for="item in deviceTypeList" :key="item.listCode" :label="item.listDesc" 			:value="item.listCode">
    </el-option>
</el-select>
```

``` vue
getDeviceTypeName(value) {
      let obj = this.deviceTypeList.find(item => {
        //这里的oneData就是上面遍历的数据源
        return item.listCode === value; //筛选出匹配数据
      });
      this.form.deviceTypeName = obj.listDesc;
      console.log(this.form.deviceTypeName);
    },
```

1. select标签绑定change事件
2. change事件里获取key和value的值

# 2.格式化列表字段

``` vue
<el-table-column prop="isChange" label="新增/更换" :show-overflow-tooltip="true" :formatter="isChangeFormatter" align="center">
</el-table-column>
```

``` 
isChangeFormatter(row, column, cellValue) {
      if (row.isChange == 'Y') {
        return '更换';
      }
      return '新增';
},
```

# 3.webpack中alias配置中的“@”是什么意思

如题所示，build文件夹下的webpack.base.conf.js

``` js
resolve: {
    extensions: ['.js', '.vue', '.json'],
    alias: {
      'vue$': 'vue/dist/vue.esm.js',
      '@': resolve('src')
    }
  }
```

其中的@的意思是：
只是一个别名而已。这里设置别名是为了让后续引用的地方减少路径的复杂度

``` js
//例如
src
- components
  - a.vue
- router
  - home
    - index.vue

//index.vue 里，正常引用 A 组件：
import A from '../../components/a.vue'
//如果设置了 alias 后。
alias: {
 'vue$': 'vue/dist/vue.esm.js',
 '@': resolve('src')
}
//引用的地方路径就可以这样了
import A from '@/components/a.vue'
//这里的 @ 就起到了【resolve('src')】路径的作用。
```

# 4.webpack proxyTable 代理跨域

webpack 开发环境可以使用proxyTable 来代理跨域，生产环境的话可以根据各自的服务器进行配置代理跨域就行了。在我们的项目config/index.js 文件下可以看到有一个proxyTable的属性，我们对其简单的改写

``` js
proxyTable: {
      '/api': {
        target: 'http://api.douban.com/v2',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    }
```

这样当我们访问localhost:8080/api/movie的时候 其实我们访问的是[http://api.douban.com/v2/movi...](http://api.douban.com/v2/movie这样便达到了一种跨域请求的方案)。

当然我们也可以根据具体的接口的后缀来匹配代理，如后缀为.shtml，代码如下：

``` js
proxyTable: {
    '**/*.shtml': {
        target: 'http://192.168.198.111:8080/abc',
        changeOrigin: true
    }
}
```

可参考地址：
1.[webpack 前后端分离开发接口调试解决方案，proxyTable解决方案](http://www.cnblogs.com/coolslider/p/7076191.html)
2.[http-proxy-middleware](https://github.com/chimurai/http-proxy-middleware)

# 5.vue2.x父子组件以及非父子组件之间的通信

### 1.父组件传递数据给子组件

父组件数据如何传递给子组件呢？可以通过`props`属性来实现

父组件：

``` js
<parent>
    <child :child-msg="msg"></child>//这里必须要用 - 代替驼峰
</parent>

data(){
    return {
        msg: [1,2,3]
    };
}
```

子组件通过`props`来接收数据:

方式1：

``` js
props: ['childMsg']
```

方式2 :

``` js
props: {
    childMsg: Array //这样可以指定传入的类型，如果类型不对，会警告
}
```

方式3：

``` js
props: {
    childMsg: {
        type: Array,
        default: [0,0,0] //这样可以指定默认的值
    }
}
```

这样呢，就实现了父组件向子组件传递数据.

### 2.子组件与父组件通信

子组件：

``` js
<template>
    <div @click="up"></div>
</template>

methods: {
    up() {
        this.$emit('fun','这是一段内容'); //主动触发fun方法，'这是一段内容'为向父组件传递的数据
    }
}
```

父组件:

``` js
<div>
    <child @fun="change" :msg="msg"></child> //监听子组件触发的fun事件,然后调用change方法
</div>
methods: {
    change(msg) {
        this.msg = msg; 
    }
}
```

### 3.非父子组件通信

如果2个组件不是父子组件那么如何通信呢？这时可以通过`eventHub`来实现通信.
所谓eventHub就是创建一个事件中心，相当于中转站，可以用它来传递事件和接收事件.

``` js
let Hub = new Vue(); //创建事件中心
```

组件1触发：

```js
<div @click="eve"></div>
methods: {
    eve() {
        Hub.$emit('change','hehe'); //Hub触发事件
    }
}
```

组件2接收:

``` js
<div></div>
created() {
    Hub.$on('change', () => { //Hub接收事件
        this.msg = 'hehe';
    });
}
```

可参考：[vue非父子组件怎么进行通信](https://segmentfault.com/a/1190000008042320)

# 6.select选择器回显问题

![image-20210311142652420](https://gitee.com/gsshy/picgo/raw/master/img/image-20210311142652420.png)

单位应该显示为**千克** ,但是回显的后unitId是2,默认成string类型了,要手动设置一下:

``` vue
<el-form-item label="单位" prop="unitId">
              <el-select v-model="equimentEditeForm.unitId" value-key="value" size="small" v-bind:disabled="diasabledInput">
                <el-option v-for="item in itemUnit" :key="parseInt(item.unitId)" :label="item.unitName" :value="parseInt(item.unitId)">
                </el-option>
              </el-select>
</el-form-item>
```

可参考:https://blog.csdn.net/qq_43779703/article/details/100693565

# 7.父子组件弹窗返回值问题

- 仪器参数定义开发时遇到问题:打断点时弹窗关闭

  解决办法:vue点击弹框之外部位弹框消失,dialog加上属性：`:close-on-click-modal="false"`

  ![image-20210312145135193](https://gitee.com/gsshy/picgo/raw/master/img/image-20210312145135193.png)

- 弹出框按**x**或者**关闭**的时候，下次再点击放大镜不能弹出

  分析：弹窗是使用watch监听`showInstrumentDefineSelectDialog`值变化去控制窗口的打开和关闭，如果关闭的时候直接使用`this.dialogTableVisible = false`,`showInstrumentDefineSelectDialog`的值就不会变化(为true)，下次再点放大镜`showInstrumentDefineSelectDialog`的值还是true，没有变化，所以watch不会生效，正确关闭的时候应该使用`this.$emit("update:showInstrumentDefineSelectDialog", false`,通过watch方法再改变`dialogTableVisible`的值

  子组件关闭方法修改：

  ```java
  // 关闭弹框
  OnClose () {
    // this.dialogTableVisible = false
    this.$emit("update:showInstrumentDefineSelectDialog", false)
  },
  ```

  **正确的父子页面模板:**

  ​	父页面：

  ​		

  ```vue
  <template>
    <div class="app-container el-main cardshadow">
      <el-form :model="queryParams" ref="queryForm" :inline="true" v-show="showSearch">
        <el-form-item label="工具/仪器类型" prop="deviceType">
          <el-input v-model="queryParams.deviceType" placeholder="请输入仪器类型" clearable size="small" @keyup.enter.native="handleQuery" />
        </el-form-item>
        <el-form-item label="类型代码" prop="paraTypeCode">
          <el-input v-model="queryParams.paraTypeCode" placeholder="请输入类型代码" clearable size="small" @keyup.enter.native="handleQuery" />
        </el-form-item>
        <el-form-item label="类型名称" prop="paraTypeName">
          <el-input v-model="queryParams.paraTypeName" placeholder="请输入参数类型名称" clearable size="small" @keyup.enter.native="handleQuery" />
        </el-form-item>
        <el-form-item label="参数代码" prop="posParaCode">
          <el-input v-model="queryParams.posParaCode" placeholder="请输入参数代码" clearable size="small" @keyup.enter.native="handleQuery" />
        </el-form-item>
        <el-form-item>
          <el-button type="primary" size="mini" @click="handleQuery">查询</el-button>
          <el-button type="primary" size="mini" @click="resetQuery">重置</el-button>
        </el-form-item>
      </el-form>
  
      <el-row :gutter="10" class="mb8">
        <el-col :span="1.5">
          <el-button type="primary" icon="el-icon-plus" size="mini" @click="handleAdd">新增
          </el-button>
        </el-col>
      </el-row>
  
      <el-table v-loading="loading" border :data="posParaList" size="mini">
        <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
          <template slot-scope="scope">
            <el-button size="mini" type="text" icon="el-icon-edit" @click="handleUpdate(scope.row)">修改</el-button>
            <el-button size="mini" type="text" icon="el-icon-delete" @click="handleDelete(scope.row)">删除</el-button>
          </template>
        </el-table-column>
        <el-table-column prop="deviceType" label="工具/仪器类型" :show-overflow-tooltip="true" align="center"></el-table-column>
        <el-table-column prop="deviceTypeName" label="工具/仪器类型名称" :show-overflow-tooltip="true" align="center"></el-table-column>
        <el-table-column prop="paraTypeCode" label="类型代码" :show-overflow-tooltip="true" align="center"></el-table-column>
        <el-table-column prop="paraTypeName" label="类型名称" :show-overflow-tooltip="true" align="center"></el-table-column>
        <el-table-column prop="posParaCode" label="参数代码" :show-overflow-tooltip="true" align="center"></el-table-column>
        <el-table-column prop="posParaValue" label="参数值" :show-overflow-tooltip="true" align="center"></el-table-column>
        <el-table-column prop="remark" label="备注" :show-overflow-tooltip="true" align="center"></el-table-column>
      </el-table>
      <div class="pagination">
        <el-pagination background @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="pagination.currentPage" :page-sizes="[10, 20, 50, 100]" :page-size="pagination.pageSize" layout="total, sizes, prev, pager, next" :total="pagination.total">
        </el-pagination>
      </div>
      <!-- 添加或修改菜单对话框 -->
      <el-dialog :title="title" :visible.sync="open" width="600px" :close-on-click-modal="false" append-to-body>
        <el-form ref="form" :model="form" :rules="rules" label-width="120px">
          <el-row>
            <el-col :span="12">
              <el-form-item label="工具/仪器类型" prop="deviceType">
  <!--              <el-select v-model="form.deviceType" value-key="value"  @change="getDeviceTypeName" :disabled="this.form.status == 'change'? true:false">-->
  <!--                <el-option v-for="item in deviceTypeList" :key="item.listCode" :label="item.listDesc" :value="item.listCode">-->
  <!--                </el-option>-->
  <!--              </el-select>-->
                <el-input v-model="form.deviceTypeName" v-show="false"></el-input>
                  <el-input v-model="form.deviceType" maxlength="200" :disabled="true" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="类型代码" prop="paraTypeCode">
  <!--              <el-input v-model="form.paraTypeCode" maxlength="60" :disabled="this.form.status == 'change'? true:false" placeholder="请输入类型代码"></el-input>-->
                <el-input v-model="form.paraTypeCode" maxlength="60" :disabled="disabledInput" suffix-icon="el-icon-search" @click.native="showInstrumentCheck" placeholder="请输入类型代码"></el-input>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item label="类型名称" prop="paraTypeName">
                <el-input v-model="form.paraTypeName" maxlength="200" :disabled="true"/>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="参数代码" prop="posParaCode">
                <el-input v-model="form.posParaCode" maxlength="60" placeholder="请输入参数代码" />
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item label="参数值" prop="posParaValue">
                <el-input v-model="form.posParaValue" maxlength="60" placeholder="请输入参数值" />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item label="备注" prop="remark">
                <el-input v-model="form.remark" maxlength="30" placeholder="请输入备注" />
              </el-form-item>
            </el-col>
          </el-row>
        </el-form>
  
        <div slot="footer" class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
        <instrument-define-select :showInstrumentDefineSelectDialog.sync="showInstrumentDefineSelectDialog" :enable="enable" @checkInstrument="checkInstrument"></instrument-define-select>
      </el-dialog>
    </div>
  </template>
  
  <script>
  import instrumentDefineSelect from "../../../components/company/instrumentDefine/instrumentDefineSelect"
  export default {
    name: "posPara",
    components: {instrumentDefineSelect},
    data () {
      return {
        // 遮罩层
        loading: true,
        // 显示搜索条件
        showSearch: true,
        // 供应商类型表格数据
        posParaList: [],
        deviceTypeList: [],
        showInstrumentDefineSelectDialog: false,
        disabledInput: false,
        enable: false,
        // 弹出层标题
        title: "",
        // 是否显示弹出层
        open: false,
        // 查询参数
        queryParams: {
          deviceType: "",
          paraTypeCode: "",
          paraTypeName: "",
          posParaCode: ""
        },
        // 表单参数
        form: {},
        pagination: {
          currentPage: 1,
          pageSize: 10,
          total: 0
        },
        // 表单校验
        rules: {
          deviceType: [
            { required: true, message: "工具/仪器类型不能为空", trigger: "blur" }
          ],
          paraTypeCode: [
            { required: true, message: "类型代码不能为空", trigger: "change" },
            {
              pattern: /^[A-Za-z0-9]{1,30}$/,
              message: "请输入数字或者字母",
              trigger: "blur"
            }
          ],
          paraTypeName: [
            { required: true, message: "类型名称不能为空", trigger: "blur" }
          ],
          posParaCode: [
            { required: true, message: "参数代码不能为空", trigger: "blur" },
            {
              pattern: /^[A-Za-z0-9]{1,30}$/,
              message: "请输入数字或者字母",
              trigger: "blur"
            }
          ],
          posParaValue: [
            { required: true, message: "参数值不能为空", trigger: "blur" }
          ]
        }
      }
    },
    created () {
      this.getPosParaList()
      this.getDeviceTypeList()
    },
    methods: {
      /** 查询参数列表 */
      getPosParaList () {
        this.loading = true
        let that = this
        let datas = {
          deviceType: that.queryParams.deviceType,
          paraTypeCode: that.queryParams.paraTypeCode,
          paraTypeName: that.queryParams.paraTypeName,
          posParaCode: that.queryParams.posParaCode,
          pageNum: this.pagination.currentPage,
          pageSize: this.pagination.pageSize
        }
        this.$request
          .fetchGetPosPara(datas)
          .then(res => {
            that.posParaList = res.data.data.list
            that.pagination.total = res.data.data.total
          })
          .catch(err => {
            console.log(err)
          })
  
        this.loading = false
      },
      /** 查询工具仪器类型 */
      getDeviceTypeList () {
        let that = this
        let data = { listTypeCode: "posDeviceType" }
        this.$request
          .fetchSystemResourceList(data)
          .then(res => {
            that.deviceTypeList = res.data.data.list
            // that.fixStatus.put({listCode:'',listDesc:'全部'});
          })
          .catch(err => {
            console.log(err)
          })
      },
      getDeviceTypeName (value) {
        let obj = this.deviceTypeList.find(item => {
          // 这里的oneData就是上面遍历的数据源
          return item.listCode === value // 筛选出匹配数据
        })
        this.form.deviceTypeName = obj.listDesc
        console.log(this.form.deviceTypeName)
      },
      // 取消按钮
      cancel () {
        this.open = false
      },
      // 表单重置
      reset () {
        this.form = {
          deviceType: "",
          paraTypeCode: "",
          paraTypeName: "",
          posParaCode: "",
          sorPosParaCode: "",
          posParaValue: "",
          remark: ""
        }
        this.resetForm("form")
      },
      /** 搜索按钮操作 */
      handleQuery () {
        this.getPosParaList()
      },
      /** 重置按钮操作 */
      resetQuery () {
        this.resetForm("queryForm")
        this.getPosParaList()
      },
      /** 新增按钮操作 */
      handleAdd (row) {
        this.reset()
        this.open = true
        this.title = "仪器参数新增"
        this.disabledInput = false
      },
      /** 修改按钮操作 */
      handleUpdate (row) {
        this.reset()
        this.form = JSON.parse(JSON.stringify(row))
        this.form.sorPosParaCode = this.form.posParaCode
        this.form.status = "change"
        this.open = true
        this.disabledInput = true
        this.title = "仪器参数修改"
      },
      /** 提交按钮 */
      submitForm: function () {
        let from = this.form
        this.$refs["form"].validate(valid => {
          if (valid) {
            if (this.form.status === "change") {
              let datas = {
                deviceType: from.deviceType,
                deviceTypeName: from.deviceTypeName,
                paraTypeCode: from.paraTypeCode,
                paraTypeName: from.paraTypeName,
                posParaCode: from.posParaCode,
                sorPosParaCode: from.sorPosParaCode,
                posParaValue: from.posParaValue,
                remark: from.remark
              }
              this.$request
                .fetchUpdatePosPara(datas)
                .then(res => {
                  if (res.data.code === 0) {
                    this.$message({
                      message: res.data.message,
                      type: "success"
                    })
                  } else {
                    Message({
                      message: res.data.message,
                      type: "warning"
                    })
                  }
                  this.open = false
                  this.getPosParaList()
                })
                .catch(err => {
                  console.log(err)
                })
            } else {
              let datas = {
                deviceType: from.deviceType,
                deviceTypeName: from.deviceTypeName,
                paraTypeCode: from.paraTypeCode,
                paraTypeName: from.paraTypeName,
                posParaCode: from.posParaCode,
                posParaValue: from.posParaValue,
                remark: from.remark
              }
              this.$request
                .fetchAddPosPara(datas)
                .then(res => {
                  if (res.data.code === 0) {
                    this.$message({
                      message: res.data.message,
                      type: "success"
                    })
                  } else {
                    Message({
                      message: res.data.message,
                      type: "warning"
                    })
                  }
                  this.open = false
                  this.getPosParaList()
                })
                .catch(err => {
                  console.log(err)
                })
            }
          }
        })
      },
      /** 删除按钮操作 */
      handleDelete (row) {
        let that = this
        this.$confirm("是否删除工具/仪器类型", "警告", {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        })
          .then(function () {
            return that.delType(row)
          })
          .catch(function () {})
      },
      // 删除工具/仪器类型
      delType (row) {
        // let datas = {
        //     deviceType: row.deviceType,
        //     paraTypeCode: row.paraTypeCode,
        //     posParaCode: row.posParaCode
        // };
        this.$request
          .fetchDelPosPara(row)
          .then(res => {
            this.getPosParaList()
            if (res.data.code === 0) {
              this.$message({
                message: res.data.message,
                type: "success"
              })
            } else {
              Message({
                message: res.data.message,
                type: "warning"
              })
            }
          })
          .catch(err => {
            console.log(err)
          })
      },
      // 界面改变
      handleSizeChange (val) {
        let that = this
        that.pagination.pageSize = val
        that.getPosParaList()
      },
      handleCurrentChange (val) {
        let that = this
        that.pagination.currentPage = val
        that.getPosParaList()
      },
      checkInstrument (row) {
        console.log(row)
        let editForms = JSON.parse(JSON.stringify(this.form))
        editForms.deviceType = row.deviceType
        editForms.deviceTypeName = row.deviceTypeName
        editForms.paraTypeCode = row.paraTypeCode
        editForms.paraTypeName = row.paraTypeName
        this.form = editForms
        this.showInstrumentDefineSelectDialog = false
      },
      showInstrumentCheck () {
        if (this.disabledInput === false) {
          this.showInstrumentDefineSelectDialog = true
        }
      }
    }
  }
  </script>
  <style >
  .pagination {
    padding-top: 20px;
    float: right;
  }
  </style>
  ```

  子页面：

  ​	

  ```vue
  <template>
    <div>
      <el-dialog title="仪器选择" :visible.sync="dialogTableVisible" append-to-body width="800px" @close="OnClose()">
        <el-form :inline="true" :model="queryParams" class="demo-form-inline">
          <el-form-item label="类型代码" prop="paraTypeCode"   size="mini">
            <el-input v-model="queryParams.paraTypeCode" placeholder="请输入类型代码"></el-input>
          </el-form-item>
          <el-form-item label="类型名称" prop="paraTypeName" size="mini">
            <el-input v-model="queryParams.paraTypeName" placeholder="请输入类型名称"></el-input>
          </el-form-item>
          <el-form-item size="mini">
            <el-button type="primary" size="mini"  @click="handleQuery">查询</el-button>
            <el-button type="primary" size="mini" @click="resetQuery">重置</el-button>
          </el-form-item>
        </el-form>
  
        <div class="supplier-content">
          <div style="flex:1">
            <el-table :data="instrumentList" highlight-current-row size="mini" border @row-dblclick="rowDbClick" @current-change="handleCurrentRow">
              <el-table-column prop="deviceType" label="仪器类型" align="center">
              </el-table-column>
              <el-table-column prop="deviceTypeName" label="仪器类型名称" align="center">
              </el-table-column>
              <el-table-column prop="paraTypeCode" label="类型代码" align="center">
              </el-table-column>
              <el-table-column prop="paraTypeName" label="类型名称" align="center">
              </el-table-column>
            </el-table>
  
            <div class="pagination">
              <el-pagination background @size-change="handleSizeChange" @current-change="handleCurrentChange"
                             :current-page="pagination.currentPage" :page-sizes="[10, 20, 50, 100]" :page-size="pagination.pageSize"
                             layout="total, sizes, prev, pager, next" :total="pagination.total">
              </el-pagination>
            </div>
          </div>
        </div>
  
        <div slot="footer" class="dialog-footer">
          <el-button type="primary" @click="submitForm">确 定</el-button>
          <el-button @click="cancel">取 消</el-button>
        </div>
      </el-dialog>
    </div>
  </template>
  
  <script>
  
  
  export default {
    props: {
      showInstrumentDefineSelectDialog: false
    },
    data () {
      return {
        dialogTableVisible: false,
        queryParams: {
          paraTypeCode: "",
          paraTypeName: ""
        },
        // 翻页
        pagination: {
          currentPage: 1,
          pageSize: 10,
          total: 0
        },
        instrumentList: [],
        currentRow: {}
      }
    },
    created () {
      this.getInstrumentList()
    },
    watch: {
      showInstrumentDefineSelectDialog (newValue, oldValue) {
        this.dialogTableVisible = newValue
      }
    },
    methods: {
      /* 查询采集点列表 */
      getInstrumentList () {
        this.loading = true
        let that = this
        let datas = {
          paraTypeCode: that.queryParams.paraTypeCode,
          paraTypeName: that.queryParams.paraTypeName,
          pageNum: that.pagination.currentPage,
          pageSize: that.pagination.pageSize
        }
        that.$request
          .fetchGetPosPara(datas)
          .then((res) => {
            that.instrumentList = res.data.data.list
            that.pagination.total = res.data.data.total
          })
          .catch((err) => {
            console.log(err)
          })
        this.loading = false
      },
      /** 搜索按钮操作 */
      handleQuery () {
        this.getInstrumentList()
      },
      /** 重置按钮操作 */
      resetQuery () {
        this.queryParams = {
          paraTypeCode: "",
          paraTypeName: ""
        }
        this.resetForm("queryForm")
        this.handleQuery()
      },
      // 界面改变
      handleSizeChange (val) {
        let that = this
        that.pagination.pageSize = val
        that.getInstrumentList()
      },
      handleCurrentChange (val) {
        let that = this
        that.pagination.currentPage = val
        that.getInstrumentList()
      },
      // 双击行
      rowDbClick (row) {
        this.dialogTableVisible = false // 子组件弹框隐藏
        this.$emit("checkInstrument", row)
      },
      // 关闭弹框
      OnClose () {
        // this.dialogTableVisible = false
        this.$emit("update:showInstrumentDefineSelectDialog", false)
      },
      // 选中某一行
      handleCurrentRow (val) {
        this.currentRow = val
      },
      // 确定按钮
      submitForm () {
        this.dialogTableVisible = false // 子组件弹框隐藏
        this.$emit("checkInstrument", this.currentRow)
      },
      // 取消按钮
      cancel () {
        // this.dialogTableVisible = false
        this.$emit("update:showInstrumentDefineSelectDialog", false)
      }
    }
  }
  </script>
  <style >
    @import '../../../assets/css/ztree.css';
  </style>
  ```

# 8.弹出框点空白区域消失设定

  ```vue
  <el-dialog :title="title" :visible.sync="open" width="600px" append-to-body :close-on-click-modal="false">
  ```

# 9.后面要用到前面异步执行的结果值(异步变同步)

用promise

```js
/** 查询点检明细 */
getFixCheckDetail (checkId) {
  return new Promise((resolve, reject) => {
    let that = this
    let data = { checkId: checkId }
    that.$request
      .fetchFixCheckDetail(data)
      .then((res) => {
        console.log(res)
        that.fixCheckDetail = res.data.data
        resolve()
        // that.fixStatus.put({listCode:'',listDesc:'全部'});
      })
      .catch((err) => {
        console.log(err)
        reject()
      })
  })
}
```

  

```js
/** 详情按钮操作 */
handleShowDetail (row) {
  if (row.executeStatus === "Y" && row.modeName === "点检") {
    let that = this
    let data = JSON.parse(JSON.stringify(row))
    Promise.all([
      that.getFixCheckDetail(data.checkId),
      that.getFixCheckAnomalousCount(data.checkId),
      that.getFixCheckImage(data.checkId)
    ]).then((values) => {
      that.checkEditeForm = data
      that.checkEditeForm.count = that.anomalousCount
      that.openMX = true
      that.titleMX = "点检单详情"
    }).catch((reason) => {
      console.log(reason)
    })
  }
},
```

**查百度好像还有一种方法：但是自己测试好像没生效**

```js
export function post_request(url, obj) {  
    //返回一个promise实例。  
    return new Promise((resolve, reject) => {  
        uni.request({  
            url: apiurl+url,  
            data: obj,
            method:'POST',
            success: (result) => {  
                resolve(result.data);
                
            },
            fail: (e) => {  
                reject(e);  
            }  
        })  
    })  
} 
```

post_request只需要返回一个Promise实例即可，success中resolve(result.data)

```js
//请求数据
uni.login({
    provider: 'weixin',
    success: async function (loginRes) {
        let code = loginRes.code        
        let result= await post_request("/oauth/oauth",{code:code})
        console.log(result)
    }
})
```

使用的时候，在 "上级函数"中加上async。然后再使用的时候加上 let result = await promiseFun()

# 10.this.$refs介绍

调用子组件的方法

![image-20210423175210171](https://gitee.com/gsshy/picgo/raw/master/img/image-20210423175210171.png)

![image-20210423175134985](https://gitee.com/gsshy/picgo/raw/master/img/image-20210423175134985.png)

# 11.element里面的trigger: 'blur'和trigger: 'change'有什么区别

触发方式，blur失去焦点，change数据改变

# 12.table里面控制按钮权限

班计划不能再排产：

![image-20210414141221971](https://gitee.com/gsshy/picgo/raw/master/img/image-20210414141221971.png)

```js
<el-table-column label="操作" align="center" fixed class-name="small-padding fixed-width" width="280">
  <template slot-scope="scope">
    <el-button size="mini" type="text" icon="el-icon-set-up" :disabled="scope.row.planTypeCode==='FlightPlan'"  @click="handleProduct(scope.row)">排产</el-button>
    <el-button size="mini" type="text" icon="el-icon-edit" @click="handleUpdate(scope.row)">修改</el-button>
     <el-button size="mini" type="text" icon="el-icon-delete" @click="handleDel(scope.row)">删除</el-button>
  </template>
</el-table-column>
```

# 13.上传文件图片及预览(员工证件档案功能)

## 1.form表单里上传文件

![image-20210421173527015](https://gitee.com/gsshy/picgo/raw/master/img/image-20210421173527015.png)

```js
<el-col :span="24">
    <el-form-item label="文件上传" prop="fileList">
      <el-upload class="upload-demo" ref="upload" :action="action" :on-preview="handlePreview"
                 :on-remove="handleRemove" accept=".png,.jpg" :on-exceed="handleExceed" :limit="3"
                 :on-success="handleSuccess" :on-error="handleError" :before-upload="beforeImgUpload" list-type="picture-card" :file-list="photoList">
        <el-button size="small" type="primary">点击上传</el-button>
      </el-upload>
      <!--图片预览的dialog-->
      <el-dialog :visible.sync="dialogVisible">
        <img width="100%" :src="previewSrc">
      </el-dialog>
    </el-form-item>
</el-col>
```

```js
return {
  // 搜索框
  maxHeight: null,
  previewSrc: "",
  previewSrcList: [],
  dialogVisible: false,
  remindList: [],
  photoList: [],
  action: window.location.protocol + "//" + window.location.hostname + ":8081/company/interCertificate/saveFile?token=" + store.getters.token,
  serverIp: localStorage.getItem("serverIp"),
  fileList: [],
 }
```

新增/修改按钮：图片的回显问题

```js
/** 新增按钮操作 */
handleAdd () {
  this.reset()
  this.form.status = "add"
  this.photoList = []
  this.disabledInput = false
  this.title = "人员证件档案新增"
  this.open = true
},
/** 修改按钮操作 */
handleUpdate (row) {
  this.reset()
  this.form = JSON.parse(JSON.stringify(row))
  //把查出来的图片地址列表赋给anlage
  let anlage = row.anlageList
  this.form.status = "change"
  if (anlage !== null) {
    for (let i = 0; i < anlage.length; i++) {
      let obj = new Object()
      obj.url = anlage[i]
        //photoList是绑定在el-upload里的fileList
      this.photoList.push(obj)
    }
  }
  this.disabledInput = true
  this.title = "人员证件档案修改"
  this.open = true
},
```

查询列表结果并修改路径值：

```js
this.$request
  .fetchSearchInterCertificateList(datas)
  .then(res => {
    let dataList = res.data.data.list
    dataList.forEach(obj => {
      console.log(obj.employeeName)
      let newAnlageList = []
      if (obj.anlageList !== null) {
        obj.anlageList.forEach(obj2 => {
            //直接用obj2=that.serverIp+obj2不行，所以用newAnlageList.push方式
            //如果不在查询结果这修改值的话也可以在v-for里动态改(列表里展示图片)
          obj2 = that.serverIp + obj2
          newAnlageList.push(obj2)
        })
        obj.anlageList = newAnlageList
      }
    })
    this.interCertificateList = dataList
    this.pagination.total = res.data.data.total
  })
  .catch(err => {
    console.log(err)
  })
```

## 2.table里展示图片

![image-20210421174505426](https://gitee.com/gsshy/picgo/raw/master/img/image-20210421174505426.png)

```js
<el-table-column prop="anlageList" label="附件" width="150px" align="center" :show-overflow-tooltip="true">
  <template slot-scope="scope">
    <span v-for="(item,index) in scope.row.anlageList" :key="index">
      <el-image
        style="width: 50px; height: 30px"
        :src="item"
        :preview-src-list="scope.row.anlageList">
      </el-image>
    </span>
  </template>
</el-table-column>
```

:src="item"动态加路径前缀也可以采用下面这种方法

![image-20210421174659694](https://gitee.com/gsshy/picgo/raw/master/img/image-20210421174659694.png)

# 14.vue Rules验证异常问题

![image-20210421175235407](https://gitee.com/gsshy/picgo/raw/master/img/image-20210421175235407.png)

人员已经输入了还提示员工不能为空

```js
<el-form-item label="人员" prop="employeeName">
  <el-input v-model="form.employeeName" placeholder="请输入内容" suffix-icon="el-icon-search" :disabled="disabledInput" size="small" @click.native="showEmployee">
  </el-input>
</el-form-item>
```

```js
// 表单校验
rules: {
  employeeName: [
    {
      required: true,
      message: "员工不能为空",
      trigger: "change"
    }
  ]
},
```

问题原因：

![image-20210421180941790](https://gitee.com/gsshy/picgo/raw/master/img/image-20210421180941790.png)

这里定义的时候起初没有把employeeName加进去,把form表单里属性都定义一下就好了

**小技巧，新建对象的时候直接let obj = new Object(),再用数组push进去，要不然会报对象属性错误的异常**

# 15.js里replace方法替换所有

问题：

``` js
let a = "aabbaacc"
let b = "aa"
a = a.replace(b,"dd")
//最后输入:ddbbaacc
```

正常replace并不会全文替换

要全文替换可以采用下面方式：

``` js
string.replace(new RegExp(key,'g'),"b");
```

![image-20210423171806701](https://gitee.com/gsshy/picgo/raw/master/img/image-20210423171806701.png)

可以参考：https://www.cnblogs.com/wenqiangit/p/10485245.html

# 16.强口领密码校验

``` js
password: [
          {required: true, validator: validatePass, trigger: "blur"},
          { pattern: /^(?=.*\d)(?=.*[,.?/"<>!@#$%^&*()\-_=+\\|\[\]{};:'~`\x22])(?=.*[a-z])(?=.*[A-Z]).{8,20}$/, message: '密码必须包含数字，小写字母，大写字母，特殊符号，长度为 8 - 20位' }
        ],
```

``` js
let pwdRule = /^(?=.*\d)(?=.*[,.?/"<>!@#$%^&*()\-_=+\\|\[\]{};:'~`\x22])(?=.*[a-z])(?=.*[A-Z]).{8,20}$/
                  if (!pwdRule.test(that.loginForm.password)) {
                    this.$alert("用户口令密码弱,请修改密码后重新登录", "提示", {
                      confirmButtonText: "确定",
                      callback: action => {
                        this.dialogPassVisible = true
                      }
                    });
                    return
                  }
```

