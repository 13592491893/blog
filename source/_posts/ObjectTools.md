---
title: ObjectTools
comments: true
toc: true
description: ObjectTools
top_img: https://gitee.com/gsshy/picgo/raw/master/img/top.jpg
categories:
  - java
tags:
  - utils
  - object
  - tool
date: 2021-12-15 16:00:00
---


ObjectTools

```java
package com.shys.industrialinternet.common.utils;

import com.esotericsoftware.reflectasm.FieldAccess;
import com.esotericsoftware.reflectasm.MethodAccess;
import org.apache.commons.lang3.ArrayUtils;
import org.apache.commons.lang3.StringUtils;

import java.lang.reflect.Field;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.ConcurrentMap;

public class ObjectTools {

    private static final ConcurrentMap<Class, MethodAccess> methodLocalCache = new ConcurrentHashMap();
    private static final ConcurrentMap<Class, Field[]> fieldLocalCache = new ConcurrentHashMap();

    private static MethodAccess getMethods(Class clazz) {
        if (methodLocalCache.containsKey(clazz)) {
            return methodLocalCache.get(clazz);
        }

        MethodAccess methodAccess = MethodAccess.get(clazz);
        methodLocalCache.putIfAbsent(clazz, methodAccess);
        return methodAccess;
    }

    private static Field[] getFields(Class clazz) {
        if (fieldLocalCache.containsKey(clazz)) {
            return fieldLocalCache.get(clazz);
        }

        Field[] result = new Field[0];
        for (Class nextClass = clazz; nextClass != Object.class; nextClass = nextClass.getSuperclass()) {
            Field[] fields = nextClass.getDeclaredFields();
            if (fields.length != 0) {
                result = ArrayUtils.addAll(result, fields);
            }
        }

        fieldLocalCache.putIfAbsent(clazz, result);

        return result;
    }


    public static <F, T> T copyProperties(F from, Class<T> to) {
        T result = null;
        try {
            result = to.newInstance();
        } catch (Exception e) {
            e.printStackTrace();
        }
        MethodAccess fromMethodAccess = getMethods(from.getClass());
        MethodAccess toMethodAccess = getMethods(to);
        Field[] fromDeclaredFields = getFields(from.getClass());
        for (Field field : fromDeclaredFields) {
            String name = field.getName();
            try {
                Object value = fromMethodAccess.invoke(from, "get" + StringUtils.capitalize(name), null);
                toMethodAccess.invoke(result, "set" + StringUtils.capitalize(name), value);
            } catch (Exception e) {
                // 设置异常，可能会没有对应字段，忽略
            }
        }

        return result;
    }

}

```

