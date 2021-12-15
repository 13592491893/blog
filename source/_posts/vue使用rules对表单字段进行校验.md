---
title: vue使用rules对表单字段进行校验
comments: true
toc: true
description: vue使用rules对表单字段进行校验
top_img: 'https://gitee.com/gsshy/picgo/raw/master/img/top.jpg'
categories:
  - vue
tags:
  - 校验
  - vue
  - rules
abbrlink: b0401a8b
date: 2020-10-25 16:00:00
---
# vue使用rules对表单字段进行校验

在实际开发过程中，发现以下的写法比较累赘，因为在后面的项目中，继续优化表单验证的方法，让代码更简洁。主要的修改是验证方法的修改和调用

以前验证表单字段的最大长度，description: [{ max: 200, message: '最大长度为200个字符', trigger: 'blur' }]

现在写法：description: [validateLen(200)]

　　其中，validateLen是我封装的一个方法。

使用步骤：

　　1、在uitls文件夹中新建一个validate.js文件。

　　2、定义需要验证的方法

　　3、在需要使用的页面中引入import { validateLen, validateChart_ } from '@/utils/validate'

　　4、在用于校验的rules中使用。例如：description: [validateLen(200)]

具体写法：参考如下。只是把旧的写法统一封装成了return 进行输出。少写很多代码，省事整洁

``` js
// 是否必填
export function validateNecessary(tipInfo = '请输入', trigger = 'blur', isNeces = true,) {
  return {
    required: isNeces,
    message: tipInfo,
    trigger
  }
}

// 验证最大长度
export function validateLen(len = 20) {
  return {
    max: len,
    message: '最大长度为' + len + '个字符',
    trigger: 'blur'
  }
}
```

// -----------------------------第一版----------------------

基于element-ui

1、在代码中，添加属性：:rule

```vue
<el-form :model="form" :rules="rules" ref="form" label-width="150px"></el-form>
并且，在<el-form-item>中添加prop属性，对应rules中的规则
```

![image-20210309110347192](https://gitee.com/gsshy/picgo/raw/master/img/image-20210309110347192.png)

2、新开一个文件夹（validate.js）定义验证规则

![image-20210309110437037](https://gitee.com/gsshy/picgo/raw/master/img/image-20210309110437037.png)

3、在页面(index.vue)中引入验证规则定义的文件，并在export default中定义rule规则，使用语法：{validator:验证方法,trigger:验证触发}

![image-20210309110519467](https://gitee.com/gsshy/picgo/raw/master/img/image-20210309110519467.png)

4、以下是validator.js文件的部分验证方法

``` js
/*是否合法IP地址*/
export function validateIP(rule, value,callback) {
  if(value==''||value==undefined||value==null){
    callback();
  }else {
    const reg = /^(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])\.(\d{1,2}|1\d\d|2[0-4]\d|25[0-5])$/;
    if ((!reg.test(value)) && value != '') {
      callback(new Error('请输入正确的IP地址'));
    } else {
      callback();
    }
  }
}

/* 是否手机号码或者固话*/
export function validatePhoneTwo(rule, value, callback) {
  const reg = /^((0\d{2,3}-\d{7,8})|(1[34578]\d{9}))$/;;
  if (value == '' || value == undefined || value == null) {
    callback();
  } else {
    if ((!reg.test(value)) && value != '') {
      callback(new Error('请输入正确的电话号码或者固话号码'));
    } else {
      callback();
    }
  }
}
/* 是否固话*/
export function validateTelphone(rule, value,callback) {
  const reg =/0\d{2}-\d{7,8}/;
  if(value==''||value==undefined||value==null){
    callback();
  }else {
    if ((!reg.test(value)) && value != '') {
      callback(new Error('请输入正确的固话（格式：区号+号码,如010-1234567）'));
    } else {
      callback();
    }
  }
}
/* 是否手机号码*/
export function validatePhone(rule, value,callback) {
  const reg =/^[1][3,4,5,7,8][0-9]{9}$/;
  if(value==''||value==undefined||value==null){
    callback();
  }else {
    if ((!reg.test(value)) && value != '') {
      callback(new Error('请输入正确的电话号码'));
    } else {
      callback();
    }
  }
}
/* 是否身份证号码*/
export function validateIdNo(rule, value,callback) {
  const reg = /(^\d{15}$)|(^\d{18}$)|(^\d{17}(\d|X|x)$)/;
  if(value==''||value==undefined||value==null){
    callback();
  }else {
    if ((!reg.test(value)) && value != '') {
      callback(new Error('请输入正确的身份证号码'));
    } else {
      callback();
    }
  }
}
/* 是否邮箱*/
export function validateEMail(rule, value,callback) {
  const reg =/^([a-zA-Z0-9]+[-_\.]?)+@[a-zA-Z0-9]+\.[a-z]+$/;
  if(value==''||value==undefined||value==null){
    callback();
  }else{
    if (!reg.test(value)){
      callback(new Error('请输入正确的邮箱地址'));
    } else {
      callback();
    }
  }
}
/* 合法uri*/
export function validateURL(textval) {
  const urlregex = /^(https?|ftp):\/\/([a-zA-Z0-9.-]+(:[a-zA-Z0-9.&%$-]+)*@)*((25[0-5]|2[0-4][0-9]|1[0-9]{2}|[1-9][0-9]?)(\.(25[0-5]|2[0-4][0-9]|1[0-9]{2}|[1-9]?[0-9])){3}|([a-zA-Z0-9-]+\.)*[a-zA-Z0-9-]+\.(com|edu|gov|int|mil|net|org|biz|arpa|info|name|pro|aero|coop|museum|[a-zA-Z]{2}))(:[0-9]+)*(\/($|[a-zA-Z0-9.,?'\\+&%$#=~_-]+))*$/;
  return urlregex.test(textval);
}

/*验证内容是否英文数字以及下划线*/
export function isPassword(rule, value, callback) {
  const reg =/^[_a-zA-Z0-9]+$/;
  if(value==''||value==undefined||value==null){
    callback();
  } else {
    if (!reg.test(value)){
      callback(new Error('密码仅由英文字母，数字以及下划线组成'));
    } else {
      callback();
    }
  }
}

/*自动检验数值的范围*/
export function checkMax20000(rule, value, callback) {
  if (value == '' || value == undefined || value == null) {
    callback();
  } else if (!Number(value)) {
    callback(new Error('请输入[1,20000]之间的数字'));
  } else if (value < 1 || value > 20000) {
    callback(new Error('请输入[1,20000]之间的数字'));
  } else {
    callback();
  }
}

//验证数字输入框最大数值,32767
export function checkMaxVal(rule, value,callback) {
  if (value < 0 || value > 32767) {
    callback(new Error('请输入[0,32767]之间的数字'));
  } else {
    callback();
  }
}
//验证是否1-99之间
export function isOneToNinetyNine(rule, value, callback) {
  if (!value) {
    return callback(new Error('输入不可以为空'));
  }
  setTimeout(() => {
    if (!Number(value)) {
      callback(new Error('请输入正整数'));
    } else {
      const re = /^[1-9][0-9]{0,1}$/;
      const rsCheck = re.test(value);
      if (!rsCheck) {
        callback(new Error('请输入正整数，值为【1,99】'));
      } else {
        callback();
      }
    }
  }, 0);
}

// 验证是否整数
export function isInteger(rule, value, callback) {
  if (!value) {
    return callback(new Error('输入不可以为空'));
  }
  setTimeout(() => {
    if (!Number(value)) {
      callback(new Error('请输入正整数'));
    } else {
      const re = /^[0-9]*[1-9][0-9]*$/;
      const rsCheck = re.test(value);
      if (!rsCheck) {
        callback(new Error('请输入正整数'));
      } else {
        callback();
      }
    }
  }, 0);
}
// 验证是否整数,非必填
export function isIntegerNotMust(rule, value, callback) {
  if (!value) {
    callback();
  }
  setTimeout(() => {
    if (!Number(value)) {
      callback(new Error('请输入正整数'));
    } else {
      const re = /^[0-9]*[1-9][0-9]*$/;
      const rsCheck = re.test(value);
      if (!rsCheck) {
        callback(new Error('请输入正整数'));
      } else {
        callback();
      }
    }
  }, 1000);
}

// 验证是否是[0-1]的小数
export function isDecimal(rule, value, callback) {
  if (!value) {
    return callback(new Error('输入不可以为空'));
  }
  setTimeout(() => {
    if (!Number(value)) {
      callback(new Error('请输入[0,1]之间的数字'));
    } else {
      if (value < 0 || value > 1) {
        callback(new Error('请输入[0,1]之间的数字'));
      } else {
        callback();
      }
    }
  }, 100);
}

// 验证是否是[1-10]的小数,即不可以等于0
export function isBtnOneToTen(rule, value, callback) {
  if (typeof value == 'undefined') {
    return callback(new Error('输入不可以为空'));
  }
  setTimeout(() => {
    if (!Number(value)) {
      callback(new Error('请输入正整数，值为[1,10]'));
    } else {
      if (!(value == '1' || value == '2' || value == '3' || value == '4' || value == '5' || value == '6' || value == '7' || value == '8' || value == '9' || value == '10')) {
        callback(new Error('请输入正整数，值为[1,10]'));
      } else {
        callback();
      }
    }
  }, 100);
}
// 验证是否是[1-100]的小数,即不可以等于0
export function isBtnOneToHundred(rule, value, callback) {
  if (!value) {
    return callback(new Error('输入不可以为空'));
  }
  setTimeout(() => {
    if (!Number(value)) {
      callback(new Error('请输入整数，值为[1,100]'));
    } else {
      if (value < 1 || value > 100) {
        callback(new Error('请输入整数，值为[1,100]'));
      } else {
        callback();
      }
    }
  }, 100);
}
// 验证是否是[0-100]的小数
export function isBtnZeroToHundred(rule, value, callback) {
  if (!value) {
    return callback(new Error('输入不可以为空'));
  }
  setTimeout(() => {
    if (!Number(value)) {
      callback(new Error('请输入[1,100]之间的数字'));
    } else {
      if (value < 0 || value > 100) {
        callback(new Error('请输入[1,100]之间的数字'));
      } else {
        callback();
      }
    }
  }, 100);
}

// 验证端口是否在[0,65535]之间
export function isPort(rule, value, callback) {
  if (!value) {
    return callback(new Error('输入不可以为空'));
  }
  setTimeout(() => {
    if (value == '' || typeof(value) == undefined) {
      callback(new Error('请输入端口值'));
    } else {
      const re = /^([0-9]|[1-9]\d|[1-9]\d{2}|[1-9]\d{3}|[1-5]\d{4}|6[0-4]\d{3}|65[0-4]\d{2}|655[0-2]\d|6553[0-5])$/;
      const rsCheck = re.test(value);
      if (!rsCheck) {
        callback(new Error('请输入在[0-65535]之间的端口值'));
      } else {
        callback();
      }
    }
  }, 100);
}
// 验证端口是否在[0,65535]之间，非必填,isMust表示是否必填
export function isCheckPort(rule, value, callback) {
  if (!value) {
    callback();
  }
  setTimeout(() => {
    if (value == '' || typeof(value) == undefined) {
      //callback(new Error('请输入端口值'));
    } else {
      const re = /^([0-9]|[1-9]\d|[1-9]\d{2}|[1-9]\d{3}|[1-5]\d{4}|6[0-4]\d{3}|65[0-4]\d{2}|655[0-2]\d|6553[0-5])$/;
      const rsCheck = re.test(value);
      if (!rsCheck) {
        callback(new Error('请输入在[0-65535]之间的端口值'));
      } else {
        callback();
      }
    }
  }, 100);
}

/* 小写字母*/
export function validateLowerCase(str) {
  const reg = /^[a-z]+$/;
  return reg.test(str);
}
/*保留2为小数*/
export function validatetoFixedNew(str) {
  return str ;
}
/* 验证key*/
// export function validateKey(str) {
//     var reg = /^[a-z_\-:]+$/;
//     return reg.test(str);
// }

/* 大写字母*/
export function validateUpperCase(str) {
  const reg = /^[A-Z]+$/;
  return reg.test(str);
}

/* 大小写字母*/
export function validatAlphabets(str) {
  const reg = /^[A-Za-z]+$/;
  return reg.test(str);
}
```

