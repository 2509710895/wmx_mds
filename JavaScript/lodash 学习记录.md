# lodash 学习记录

[lodash 常用功能总结 ](https://juejin.cn/post/7143579596217122853#heading-17)

[lodash 常用](https://juejin.cn/post/7018399380516388877)

## 数组

## 字符串

## 算术与数字

### 判断是否为 NAN

```
_.isNaN(num)
```

## 对象

## 函数

### 防抖 debounce

### 节流 throttle

### 只执行一次 once

### 只执行 n 次 before

### 在 n 次后执行 after

### 延迟 xx ms 执行

```TypeScript
_.delay(func,wait,...args)
```

### 在本次调用堆栈被清空后执行 defer

### 缓存函数结果 memorize

```TypeScript
// resolver用于计算缓存key，当key相同时，使用缓存。默认使用func的第一个参数为key
_.memorize(func [,resolver])
```

### 柯里化 curry

## 通用工具

### 流水线 flow

```TypeScript
_.flow([func])
// 大驼峰
const pascalCase=_.flow(_.upperFirst,_.camelCase)
```

### 生成唯一 ID uniqueId

```TypeScript
_.uniqueId([prefix=""])
```

## 语言

### cloneDeep 深拷贝

### clone 浅拷贝
