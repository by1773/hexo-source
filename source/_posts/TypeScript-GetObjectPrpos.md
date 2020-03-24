---
title: 舒适访问对象深层次属性~
date: 2020-03-24 08:26:52
tags: ['ts','js']
categories: 前端
keywords: ['ts','js']
top_img: https://goss1.cfp.cn/creative/vcg/veer/400/new/VCG41N680280554.jpg?x-oss-process=image/format,webp
cover : https://goss3.cfp.cn/creative/vcg/veer/nowarter800/new/VCG41N514576868.jpg?x-oss-process=image/format,webp
---
## 舒适访问对象深层次属性~

### 一、黑暗时代

​		在前后端分离的系统中，前端页面一般通过调用 REST API 来获取服务端提供的与页面相关的数据。这里我们以获取用户基本信息的接口为例，假设该接口会返回以下数据：

```
const apiResult = {
  code: 200,
  data: {
    name: "Semlinker",
    age: 30,
    address: {
      province: '福建',
      city: '厦门'
    }
  }
}
```

如果页面中需要显示当前用户的地址信息，比如省、市信息。这时就可以通过以下方式来获取：

```
const province = apiResult.data.address.province; // 福建
const city = apiResult.data.address.city; // 厦门
```

上面的数据访问方式很直观，就是一层层的进行数据访问。然而这种方式会存在很大的隐患，比如有的用户可能未设置地址信息，那么这时候的返回的数据结构就可能是这样的：

```
const apiResult = {
  code: 200,
  data: {
    name: "Semlinker",
    age: 30
  }
}
```

这时如果我们还是使用 `apiResult.data.address.province` 的方式来访问 `province` 省份信息，页面就会抛出以下异常信息：

```
Uncaught TypeError: Cannot read property 'province' of undefined
```

针对这个问题，我们有以下几种解决方案：

**方案一：**

```
const province =
  apiResult &&
  apiResult.data &&
  apiResult.data.address &&
  apiResult.data.address.province;
```

**方案二：**

```
const province = !apiResult
  ? undefined
  : !apiResult.data
    ? undefined
    : !apiResult.data.address
      ? undefined
      : apiResult.data.address.province;
```

**方案三：**

```
let province: string | undefined = undefined;
try {
  province = apiResult.data.address.province;
} catch (error) {
  // 执行异常处理程序
}
```

**方案四：**

```
import * as _ from 'lodash';
const province = _.get(apiResult, 'data.address.province', undefined);
```

通过观察以上几种方案，我们发现处理多级嵌套对象的属性访问时，需要增加很多的判断逻辑，这对于我们开发者来说，是很令人抓狂的事情。值得庆幸的是，在 TypeScript 3.7 以后版本，我们就可以使用可选链（Optional Chaining）来优雅的解决上述问题。

### 二、什么是可选链

TypeScript 3.7 实现了呼声最高的 ECMAScript 功能之一：可选链（Optional Chaining）。有了可选链后，我们编写代码时如果遇到 `null` 或 `undefined` 就可以立即停止某些表达式的运行。可选链的核心是新的 `?.` 运算符，它支持以下语法：

> ```
> obj?.prop
> obj?.[expr]
> arr?.[index]
> func?.(args)
> ```

这里我们来举一个可选的属性访问的例子：

```
const val = a?.b;
```

为了更好的理解可选链，我们来看一下该 `const val = a?.b` 语句编译生成的 ES5 代码：

```
var val = a === null || a === void 0 ? void 0 : a.b;
```

上述的代码会自动检查对象 a 是否为 `null` 或 `undefined`，如果是的话就立即返回 `undefined`，这样就可以立即停止某些表达式的运行。介绍完可选链，前面获取省份的例子，就可以改写成以下方式：

```
const province = apiResult?.data?.address?.province;
```

同样，我们再来看一下该语句生成的 ES5 代码：

```
var province = (_b = (_a = apiResult === null || apiResult === void 0 ? void 0 :
  apiResult.data) === null || _a === void 0 ? void 0 : _a.address) === null ||
    _b === void 0 ? void 0 : _b.province;
```

对比编译前的 TypeScript 代码和编译后的 JavaScript 代码，你是不是感受到了可选链是多么的给力。

### 三、?. 与 && 的区别

你可能已经发现你可以使用 `?.` 来替代很多使用 `&&` 执行空检查的代码：

```
if(a && a.b) { }

if(a?.b){ }
/**
* if(a?.b){ } 编译后的ES5代码
*
* if(
*  a === null || a === void 0
*  ? void 0 : a.b) {
* }
*/
```

但需要注意的是，`?.` 与 `&&` 运算符行为略有不同，`&&` 专门用于检测 `falsy`值，比如空字符串、0、NaN、null 和 false 等。而 `?.` 只会验证对象是否为`null` 或 `undefined`，对于 0 或空字符串来说，并不会出现 "短路"。

### 四、可选元素访问

可选链除了支持可选属性的访问之外，它还支持可选元素的访问，它的行为类似于可选属性的访问，只是可选元素的访问允许我们访问非标识符的属性，比如任意字符串、数字索引和 Symbol：

```
function tryGetArrayElement<T>(arr?: T[], index: number = 0) {
    return arr?.[index];
}
```

以上代码经过编译后会生成以下 ES5 代码：

```
function tryGetArrayElement(arr, index) {
    if (index === void 0) { index = 0; }
    return arr === null || arr === void 0 ? void 0 : arr[index];
}
```

通过观察生成的 ES5 代码，很明显在 `tryGetArrayElement` 方法中会自动检测输入参数 arr 的值是否为 `null` 和 `undefined`，从而保证了我们代码的健壮性。最后我们来介绍一下可选链与函数调用。

### 五、可选链与函数调用

当尝试调用一个可能不存在的方法时也可以使用可选链。在实际开发过程中，这是很有用的。系统中某个方法不可用，有可能是由于版本不一致或者用户设备兼容性问题导致的。函数调用时如果被调用的方法不存在，使用可选链可以使表达式自动返回 `undefined` 而不是抛出一个异常。

可选调用使用起来也很简单，比如：

```
let result = obj.customMethod?.();
```

该 TypeScript 代码编译生成的 ES5 代码如下：

```
var result = (_a = obj.customMethod) === null
  || _a === void 0 ? void 0 : _a.call(obj);
```

另外在使用可选调用的时候，我们要注意以下两个注意事项：

1. 如果存在一个属性名且该属性名对应的值不是函数类型，使用 `?.` 仍然会产生一个`TypeError` 异常。
2. 可选链的运算行为被局限在属性的访问、调用以及元素的访问 —— 它不会沿伸到后续的表达式中，也就是说可选调用不会阻止 `a?.b / someMethod()` 表达式中的除法运算或 `someMethod` 的方法调用。

### 六、参考资源

- Using Optional Chaining in TypeScript and JavaScript
- optional-chaining-in-typescript
- 深入理解 TypeScript
- 重磅！TypeScript 3.7 RC 发布，备受瞩目的 Optional Chaining 来了
- MDN - 可选链