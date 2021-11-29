---
title: vue开发-关于文件下载跨域的解决方法
comments: true
toc: true
description: vue开发-关于文件下载跨域的解决方法
top_img: https://gitee.com/gsshy/picgo/raw/master/img/top.jpg
categories:
  - vue
tags:
  - 前端
  - vue
  - 跨域
date: 2020-10-25 16:00:00
---
# vue开发----关于文件下载跨域的解决方法

## 一、情况说明：

项目将文件储存在第三方服务器（阿里云）上，在下载文件的时候，需要跨域，将处理过程记录如下。

回归到自己工作中：设备资料定义有pdf文档和word文档要下载，文档是保存在外网服务器上，如果没有跨域的话可以使用<a>标签下载

![image-20210902093209798](https://gitee.com/gsshy/picgo/raw/master/img/image-20210902093209798.png)

![image-20210902093310547](https://gitee.com/gsshy/picgo/raw/master/img/image-20210902093310547.png)

``` js
downloadImg (informationLink, versionType, name) {
      let loadingInstance = Loading.service({ fullscreen: true,text:'下载中，请稍后！' });
      let url = informationLink; // data:项目中获取的数据，包含文件url以及文件名等相关参数
      let rightStr = informationLink.substring(informationLink.length - 3)
      let fileName = versionType + "-" + name;
      let xhr = new XMLHttpRequest();
      xhr.open('GET', url, true);
      xhr.responseType = 'blob';
      xhr.onload = (e) => {
        let url2 = window.URL.createObjectURL(xhr.response)
        let a = document.createElement('a');
        a.href = url2
        if(rightStr === 'zip'){
          a.download = fileName + ".zip";
        }else {
          a.download = fileName + ".apk";
        }

        a.click()
        loadingInstance.close();
        // const res = e.target.response;
        // this.saveAs(res, fileName);
      };
      xhr.send();

    },
```

## 二、第一次尝试(还不是跨域问题)

刚开始的时候，因为能获取到文件的URL地址，所以尝试以<a>标签的形式下载，代码如下：

```javascript
// 下载按钮点击事件
fileDownload (url, fileName) {
    let ele = document.createElement('a');
    ele.download = fileName;
    ele.href = url;
    ele.style.display = 'none';
    document.body.appendChild(ele);
    ele.click();
    document.body.removeChild(ele);
};
```

**结果**：虽然配置了download属性，但是因为url指向第三方资源，download会失效，表现和不使用download时一致——浏览器能打开的文件，浏览器会直接打开（可以手动下载）；不能打开的文件，会直接下载。

## 三、第二次尝试：（开始了）

既然download属性无效，干脆通过xhr请求获取文件，然后下载到本地，代码如下：

```js
// 下载按钮点击事件
fileDownload() {
    let url = this.data.url; // data:项目中获取的数据，包含文件url以及文件名等相关参数
    let fileName = this.data.file_name;
    let xhr = new XMLHttpRequest();
    xhr.open('GET', url, true);
    xhr.responseType = 'blob';
    xhr.onload = (e) => {
        const res = e.target.response;
        this.saveAs(res, fileName);
    };
    xhr.send();
}
 
// 导出文件函数
saveAs (obj, fileName) {
    let ele = document.createElement('a');
    ele.download = fileName || '下载';
    ele.href = URL.createObjectURL(obj); // 绑定a标签
    ele.style.display = 'none';
    document.body.appendChild(ele); // 兼容火狐浏览器
    ele.click();
    setTimeout(function () { // 延时释放
        URL.revokeObjectURL(obj); // 用URL.revokeObjectURL()来释放这个object URL
        document.body.removeChild(ele);// 兼容火狐浏览器
    }, 100);
};
```

**结果**：文件可以直接下载，but为虾米有的文件可以下载、有的文件告诉我跨域问题？？？刷新页面之后，刚刚还可以下载的文件又不可以下载了！！！崩溃ing...联系运维的老哥尝试修改一下第三方服务器的参数，发现没有卵用，只能再尝试一下了。

## 四、第三次尝试:(jsonp)

既然是跨域的问题，网上的解决方案蛮多的，而且作为前端面试必考题，我背的也蛮熟的。首先使用jsonp来解决，步骤如下：

1.安装jsonp插件

```js
npm install jsonp --save
```

2.在代码中使用jsonp

```js
import jsonp from 'jsonp'; // 导入插件
 
// 下载按钮点击事件
fileDownload () {
    let url = this.data.url; // data:项目中获取的数据，包含文件url以及文件名等相关参数
    let fileName = this.data.file_name;
    // 先测试一下能不能跨域成功
    jsonp(url, null, (err, data) => {
        if (err) {
            console.error(err.message);
        } else {
            console.log(data);
        }
    })
}
```

**结果**：可能是哪里使用不对，反正是没有请求数据成功，还是显示跨域问题，既然不行，那就再换方法。

3.卸载jsonp插件

```js
npm uninstall jsonp
```

## 五、第四次尝试：（fetch跨域）

废话不多说，直接上代码：

```javascript
// 下载按钮点击事件
fileDownload () {
    let url = this.data.url; // data:项目中获取的数据，包含文件url以及文件名等相关参数
    let fileName = this.data.file_name;
    // 先测试一下能不能跨域成功
    let myHeaders = new Headers({
        'Access-Control-Allow-Origin': '*',
        'Content-Type': 'text/plain'
    });
    fetch(url, {
        method:'GET',
        headers:myHeaders,
        mode:'cors'
    }).then(res=>{
        console.log(res);
    });
}
```

**结果**：依然没有解决跨域的问题，革命尚未成功，老子还得努力啊。（ps: fetch的mode属性设置为’no-cors‘的时候能请求成功，但是返回值无法使用，木的办法）

## 六、**第五次尝试：（使用插件downloadjs下载文件）**

1.安装downloadjs插件

```js
npm install downloadjs --save
```

2.使用downloadjs插件

```js
import download from 'downloadjs'; // 引用插件
 
// 下载按钮点击事件
fileDownload () {
    let url = this.data.url; // data:项目中获取的数据，包含文件url以及文件名等相关参数
    download(url); // 没看错，就是这么简单
}
```

**结果**：我在自己的电脑上发现完全可以下载文件，沾沾自喜了大概十分钟，通过别人的电脑访问我的IP测试，发现有的完全没问题，有的还是出现跨域的问题。。。。。。。。。。。要疯了有木有。。。。。。。。接着改吧。。。。。。。

3.卸载downloadjs插件

```js
npm uninstall downloadjs
```

## 七、第六次尝试：（前端已经尽力了，让后端大佬帮忙吧）

在使用了好多方法之后，在历时一整天的尝试之后，我，决定放下前端的骄傲，找后端大佬（php）商量一下，决定后端先将第三方服务器上的文件转换成二进制文件，然后通过一个接口返回给前端处理，上代码：

```js
// 下载按钮点击事件
async fileDownload () {
    let url = this.data.url; // data:项目中获取的数据，包含文件url以及文件名等相关参数
    let fileName = this.data.file_name;
    const res = await getFile({ // 获取文件二进制数据的接口
        oss_url: url
    });
    this.saveAs(new Blob([res], { type: 'text/plain;charset=UTF-8' }), fileName);
}
 
// 导出文件函数
saveAs (obj, fileName) {
    let ele = document.createElement('a');
    ele.download = fileName || '下载';
    ele.href = URL.createObjectURL(obj); // 绑定a标签
    ele.style.display = 'none';
    document.body.appendChild(ele); // 兼容火狐浏览器
    ele.click();
    setTimeout(function () { // 延时释放
        URL.revokeObjectURL(obj); // 用URL.revokeObjectURL()来释放这个object URL
        document.body.removeChild(ele);// 兼容火狐浏览器
    }, 100);
};
```

PS:  (1).将二进制流转为Blob类型的时候，属性：{type: 'text/plain;charset=UTF-8'}；

        (2).获取二进制文件的接口，我是使用项目里封装的axios方法，需注意设置属性：responseType: 'blob'。

结果：究极妥协之后终于看到了黎明的曙光，可以正常下载所需类型的文件了，大功告成！！！

## 八、vue+java代码：

vue：

```js
	  let loadingInstance = Loading.service({ fullscreen: true,text:'下载中，请稍后！' });
      let datas = {
        informationName: informationName,
        informationLink: informationLink,
      };
      this.$request
        .fetchGetFileOutputStream(datas)
        .then((res) => {
          loadingInstance.close();
          const link = document.createElement("a");
          let blob = new Blob([res.data]);
          link.style.display = "none";
          link.href = URL.createObjectURL(blob);
          link.download = informationName; //下载的文件名
          // document.body.appendChild(link);
          link.click();
          // document.body.removeChild(link);

          // self.dataToBase(informationName,data)
        })
        .catch((error) => {
            loadingInstance.close();
            self.$message({
                message: '网络连接错误',
                type: 'warning'
            });
            self.$Notice.error({
              title: "错误",
              desc: "网络连接错误",
            });
            console.log(error);
        });
```

java:

```java
@GetMapping("/getFileOutputStream")
    public void getFileOutputStream(FixCardInformationEntity entity, HttpServletResponse response) {
        try {
            fixCardInformationService.getFileOutputStream(entity, response);
        } catch (Exception e) {
            e.printStackTrace();
            logger.error(e.getMessage());
        }
    }
```

```java
@Override
    public void getFileOutputStream(FixCardInformationEntity entity, HttpServletResponse response) throws Exception {
        response.setContentType("application/vnd.ms-excel");
        response.setCharacterEncoding("UTF-8");
        String fileName = URLEncoder.encode("未命名文件", "UTF-8").replaceAll("\\+", "%20");
        if (!StringUtils.isEmpty(entity.getInformationName())) {
            fileName = URLEncoder.encode(entity.getInformationName(), "UTF-8").replaceAll("\\+", "%20");
        }
        response.setHeader("Content-disposition", "attachment;filename=" + fileName);
        URL url = null;
        InputStream is = null;
        HttpURLConnection httpUrl = null;
        try {
            url = new URL(entity.getInformationLink());
            httpUrl = (HttpURLConnection) url.openConnection();
            httpUrl.connect();
            is = httpUrl.getInputStream();
            //创建一个Buffer字符串
            byte[] buffer = new byte[1024];
            //每次读取的字符串长度，如果为-1，代表全部读取完毕
            int len = 0;
            //使用一个输入流从buffer里把数据读取出来
            while ((len = is.read(buffer)) != -1) {
                //用输出流往buffer里写入数据，中间参数代表从哪个位置开始读，len代表读取的长度
                response.getOutputStream().write(buffer, 0, len);
            }


        } catch (Exception e) {
            e.printStackTrace();
            logger.info(e.getMessage());
        } finally {
            if (is != null) {
                try {
                    is.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
            if (httpUrl != null) {
                httpUrl.disconnect();
            }
        }

    }
```

## 九、Java中HttpURLConnection使用详解、总结

https://www.cnblogs.com/tfxz/p/12621611.html#GET_22

