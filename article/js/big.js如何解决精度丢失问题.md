# big.js 如何解决精度丢失问题
**Big 构造函数**  
``` 
function _Big_() {
    function Big(n) {
        var x = this;
        // 可以通过函数调用的形式来创建 Big 对象
        if (!(x instanceof Big)) return n === UNDEFINED ? _Big_() : new Big(n);
        // 区分是否为 Big 示例.
        if (n instanceof Big) {
            x.s = n.s;
            x.e = n.e;
            x.c = n.c.slice();
        } else {
            // 边界处理
            if (typeof n !== 'string') {
                if (Big.strict === true && typeof n !== 'bigint') {
                    throw TypeError(INVALID + 'value');
                }
                // Minus zero?
                n = n === 0 && 1 / n < 0 ? '-0' : String(n);
            }
            parse(x, n);
        }
        x.constructor = Big;
    }
    ...
    return Big;
}
```
构造函数进行了 边界处理 以及 入参检查 ，随后通过 parse 函数处理，最后修正构造函数的指向  
先将 this 对象添加进 watches 里，接着 parse 函数后的位置打个断点然后刷新。  
看看经过 parse 函数生成的数据,
c 是去除首尾的 0 之后的所有数字组成的数组，e 表示用科学计数法表示 parse 函数的入参 x 时 幂的值 ，s 表示正负（1 表示正数，-1 表示负数）  
分析一下 new Big(120) ：去除首尾 0 后，c 属性的值为 [1, 2]；入参 x = 120，用科学计数法表示就是 1.2 * 10²；e 的值就是对应的 幂 ，也就是 2；120 为正，即 s = 1。同理，new Big(1.2) 对应的值就是 c = [1, 2], e = 0（1.2 * 10<sup>2</sup>）, s = 1。  

**plus/add方法**  
``` 
P.plus = P.add = function (y) {
    // 1. 用 x 和 Big 两个变量分别保存 this(调用者) 和 Big 构造函数
    var e, k, t,
        x = this,
        Big = x.constructor;
    // 2. 将入参转化为 Big 对象
    y = new Big(y);

    // 3. 判断是否符号不同，如果不同则直接调用 minus 做减法（1 + （-1）=== 1 - 1）
    if (x.s != y.s) {
        y.s = -y.s;
        return x.minus(y);
    }
    
    // 4. 分别存储 x 和 y 各自的小数点位置以及 number 数组
    var xe = x.e,
        xc = x.c,
        ye = y.e,
        yc = y.c;

    // 5. Either zero?
    if (!xc[0] || !yc[0]) {
        if (!yc[0]) {
            if (xc[0]) {
                y = new Big(x);
            } else {
                y.s = x.s;
            }
        }
        return y;
    }
    
    // 6. copy xc 数组
    xc = xc.slice();
    
    // 7. 补 0（对齐 x 和 y 的长度）
    // Prepend zeros to equalise exponents.
    // Note: reverse faster than unshifts.
    if (e = xe - ye) {
        if (e > 0) {
            ye = xe;
            t = yc;
        } else {
            e = -e;
            t = xc;
        }

        t.reverse();
        for (; e--;) t.push(0);
        t.reverse();
    }
    
    // 8. 如果 xc 长度大于 yc，则交换它们
    // Point xc to the longer array.
    if (xc.length - yc.length < 0) {
        t = yc;
        yc = xc;
        xc = t;
    }
    e = yc.length;
    
    // 9. 相加
    // Only start adding at yc.length - 1 as the further digits of xc can be left as they are.
    for (k = 0; e; xc[e] %= 10) k = (xc[--e] = xc[e] + yc[e] + k) / 10 | 0;
    // No need to check for zero, as +x + +y != 0 && -x + -y != 0
    if (k) {
        xc.unshift(k);
        ++ye;
    }

    // Remove trailing zeros.
    for (e = xc.length; xc[--e] === 0;) xc.pop();

    y.c = xc;
    y.e = ye;

    return y;
};
```
步骤:
1. 使用 x 和 Big 两个变量分别保存 this(调用者) 和 Big 构造函数
2. 将入参 y 转为 Big 实例
3. 判断 x 和 y 的符号是否不同，如果不同的话，会先将 y 取反，调用 minus 方法处理，因为 一个数加上一个负数相当于减去这个负数取反
4. 将 x 和 y 的数字数组和符号位置都保存起来
5. if 的判断条件是 !xc[0] || !yc[0]，诶，这就用到了步骤 4 的变量了。xc 就是 x 的数字数组（就是 big 实例的 c 属性），yc 同理。经过构造函数调用 parse 解析之后，实际上是已经移除首尾的 0 了，那么 !xc[0] 和 !yc[0] 怎么可能为 true ？那肯定是有段逻辑让它变成了 0  
    ``` 
    function parse(x, n) {
        ...
        // Determine leading zeros.
        for (i = 0; i < nl && n.charAt(i) == '0';) ++i;

        if (i == nl) {
        // Zero.
        x.c = [x.e = 0];
        } else {
        // Determine trailing zeros.
        for (; nl > 0 && n.charAt(--nl) == '0';);
        x.e = e - i - 1;
        x.c = [];

        // Convert string to array of digits without leading/trailing zeros.
        for (e = 0; i <= nl;) x.c[e++] = +n.charAt(i++);
        }
        ...
    }
   ```
   很明显的，它都给出注释了，当实例化时入参 x 判定为 0 的时候，x.c 和 x.e 都会被置为 0 。那步骤 5 很显然就是个 提前返回操作 ，直接返回和 0 相加的结果
6. 拷贝 xc 防止数据污染
7. 补 0
8. 比较 xc 和 yc 的长短,将 xc 指向长度较长的数组，yc 指向较短的数组，且将较短的 yc.length 用 e 存储起来。
9. plus 操作
   较长的数组前面几位都是要和较短的数组进行计算的，而后面的几位和较短的数组根本不沾边（可以把 [0, 9, 1, 2] 与 [1, 2] 的计算想象成 [0, 9, 1, 2] 与 [1, 2, 0, 0] 的计算，[1, 2, 0, 0] 后面两个 0 没必要参与计算的，直接 从较短数组最后一位开始计算 就好了）。

原文:  
[10 分钟从源码搞懂 big.js 如何解决精度丢失问题](https://mp.weixin.qq.com/s/85I2e2ZoyqwO1cahobT73A)
