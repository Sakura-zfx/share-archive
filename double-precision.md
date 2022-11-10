# 双精度浮点数
## 先试一试，思考以下几个例子的值

```js
0.1 + 0.2 === 0.3 // ?
8.7 * 100 === 870 // ?
(1005 / 1000).toFixed(2) // ?
9007199254740991 > 9007199254740990.6 // ?
9007199254740991 > 9007199254740990.5 // ?
```

## 为什么会出现这些情况？

JavaScript中的数字（BigInt除外）不管是整数还是浮点数，都是按照 [IEEE 754双精度浮点数格式](https://en.wikipedia.org/wiki/Double-precision_floating-point_format) 存在计算机中的。  
数字会转成二进制的科学计数法（$1 * 2^{E-2013} + M$ ），存储格式如下：

![image.png](https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/27d68ce0a1f740c1b4e95950c8cb06e8~tplv-k3u1fbpfcp-watermark.image?)

- **Sign** 存储 符号位（1位。0代表整数，1代表负数）
- **Exponent** 存储 指数位（11位），因为科学计数法中存在负数，所以会使用[偏码规则](https://en.wikipedia.org/wiki/Exponent_bias)进行处理。即11位二进制最大值2047，正负数各使用一半。以1023为中间位，则计算规则为 Exponent - 1023 = 科学计数法的指数值
- **Mantissa** 存储 尾数位 (显示存储52位，实际可以表示53位)

### 特殊值
当 E = 2047 （即11位指数全为1）时
- 如果 M 不全为 0，则表示 NaN
- 如果 S 为 0，M 全为 0，表示 +Infinity
- 如果 S 为 1，M 全为 1，表示 -Infinity

当 E = 0 （即11位指数全为0）时
- 如果 S 为 0，表示 +0
- 如果 S 为 1，表示 -0

所以 `Math.pow(2, 1024) === Infinity`

### 舍入规则

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/be54f2a3bc234635a02cfc5a6110e08f~tplv-k3u1fbpfcp-watermark.image?)

按照描述，对超出精度位置的处理我的理解是：  
以保留精度 10 位为例子🌰
- 如果 11 位为 1 且 11 位后面的都为 0 ，此时向上舍入和向下舍弃的精度都一样，那么就会去判断 10 位是 0 还是 1。如果 10 位是 0 代表偶数，则舍弃；如果 10 位是 1，则舍入
- 如果 11 位为 1 且 11 位后面的不都为 0 ，那么此时向上舍入的精度会更高，所以会进行舍入
- 如果 11 位为 0 ，此时距离 10 位向下舍弃的精度比向上舍入的精度高，所以会直接舍弃 11 位以及后面的位

## 知识点
1. 为什么最大安全整数是 正负 `Math.pow(2, 53) - 1` ？
>- $2^{53}$ 转换成二进制为：$1\underbrace{0000...000}_{53个0}$
>- 这个值转科学计数法后存入计算机中会舍弃最后一个 0
>- $2^{53} + 1$ 转换成二进制为：$1\underbrace{0000...000}_{52个0}1$  
>- 这个值转科学计数法后存入计算机中会舍弃最后一个 1
>M 位最大存储 52 位，所以 52 位后面的会按照规则进行舍入  
或者可以这样理解  
>因为尾数位置最大能存下 52 个1，加上科学计数法的整数位1，则一共有 53 个1，即 `Math.pow(2, 53) - 1`。  
2. 为什么 0.1 + 0.2 === 0.30000000000000004 会存在精度丢失，而 0.2 + 0.2 === 0.4 没有精度丢失？
维基上的一个解释
![image.png](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8b7850cb6be94c84aaddb6cf6f9dda6c~tplv-k3u1fbpfcp-watermark.image?)

```js
(0.1 + 0.2).toPrecision(17) // 0.30000000000000004
// 0.010011001100110011001100110011001100110011001100110101 (0.30000000000000004)
// 0.0100110011001100110011001100110011001100110011001101 (0.3)

(0.2 + 0.2).toPrecision(17) // 0.40000000000000002
// 0.01100110011001100110011001100110011001100110011001101 (0.40000000000000002)
// 0.01100110011001100110011001100110011001100110011001101 (0.4)
```
3. 如何处理精度丢失？
提出一个解决方案：
- 如果只涉及整数的运算，则可以转成 BigInt 类型进行计算
- 如果涉及小数的运算，则可以使用 toPrecision 或者转换成整数再计算
- 善用工具 [number-precision](https://github.com/nefe/number-precision)    / [lodash round](https://www.lodashjs.com/docs/lodash.round)

## 参考
> https://en.wikipedia.org/wiki/IEEE_754  维基上关于 IEEE-754 的介绍  
> https://en.wikipedia.org/wiki/Double-precision_floating-point_format  维基上关于 双精度浮点数的介绍  
> https://github.com/camsong/blog/issues/9  github 的讨论，挺详细的  
> https://zhuanlan.zhihu.com/p/66949640  

## 转换工具
> https://tool.oschina.net/hexconvert/ 在线进制转换  
> http://www.binaryconvert.com/result_double.html  可视化双精度存储格式
