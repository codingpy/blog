# 平方根倒数速算法

[[toc]]

《雷神之锤III竞技场》的 [源代码](https://github.com/id-Software/Quake-III-Arena/blob/dbe4ddb10315479fc00086f08e25d968b4b43c49/code/game/q_math.c#L549-L572) ：

```c{10}
float Q_rsqrt( float number )
{
    long i;
    float x2, y;
    const float threehalfs = 1.5F;

    x2 = number * 0.5F;
    y  = number;
    i  = * ( long * ) &y;                       // evil floating point bit level hacking
    i  = 0x5f3759df - ( i >> 1 );               // what the fuck?
    y  = * ( float * ) &i;
    y  = y * ( threehalfs - ( x2 * y * y ) );   // 1st iteration
//  y  = y * ( threehalfs - ( x2 * y * y ) );   // 2nd iteration, this can be removed

    return y;
}
```

## IEEE 浮点表示

$$
\vec x = [s, \underbrace{e_{k-1}, \dots, e_1, e_0}_\text{exp}, \underbrace{f_{n-1}, \dots, f_1, f_0}_\text{frac}]
$$

在单精度浮点格式中， $k = 8$ 、 $n = 23$ 。
在双精度浮点格式中， $k = 11$ 、 $n = 52$ 。

$$ V = (-1)^s \times M \times 2^E $$

其中，

$$
\begin{align}
  M &= 1 . f_{n-1} \dots f_1 f_0 \\
    &= 1 + f
\end{align}
$$

$$
\begin{align}
  E &= e_{k-1} \dots e_1 e_0 - (2^{k-1} - 1) \\
    &= e - \text{Bias}
\end{align}
$$

## 魔数

$$
\begin{align}
  y & = \frac{1}{\sqrt x} \\
  \log_2{y} & = -\frac{1}{2} \log_2{x} \\
  \log_2(M_y \times 2^{E_y}) & = -\frac{1}{2} \log_2(M_x \times 2^{E_x}) \\
  \log_2{M_y} + E_y & = -\frac{1}{2} (\log_2{M_x} + E_x)
\end{align}
$$

$$
\begin{align}
  \log_2{M}
    &= \log_2(1 + f) \\
    & \approx f + \sigma
\end{align}
$$

$$
\begin{align}
  (f_y + \sigma) + (e_y - \text{Bias}) & = -\frac{1}{2} [(f_x + \sigma) + (e_x - \text{Bias})] \\
  (f_y + e_y) \times 2^n & = \frac{3}{2} (\text{Bias} - \sigma) \times 2^n - \frac{1}{2} (f_x + e_x) \times 2^n \\
  \text{B2T}_w(\vec y) & = R - \frac{1}{2} \text{B2T}_w(\vec x)
\end{align}
$$

其中, $R = \text{0x5f3759df}$ 。
[McEniry](https://0x5f37642f.com/documents/McEniryMathematicsBehind.pdf) 认为，这一常数最初或许便是以“在可容忍误差范围内使用二分法”的方式求得。

::: info
对向量 $\vec x = [x_{w-1}, x_{w-2}, \dots, x_0]$ ：

$$ \text{B2T}_w(\vec x) \doteq -x_{w-1} 2^{w-1} + \sum_{i=0}^{w-2} x_i 2^i $$

最高有效位 $x_{w-1}$ 也称为 *符号位* ，它的“权重”为 $-2^{w-1}$ ，是无符号表示中权重的负数。
:::

## 牛顿法

迭代公式：

$$ x_{n+1} = x_n - \frac{f(x_n)}{f'(x_n)} $$

$$
\begin{align}
  y & = \frac{1}{\sqrt x} \\
  \frac{1}{y^2} - x & = 0
\end{align}
$$

令 $f(y) = \frac{1}{y^2} - x$ ，得

$$
\begin{align}
  y_{n+1}
    &= y_n - \frac{\frac{1}{{y_n}^2} - x}{-2 \frac{1}{{y_n}^3}} \\
    &= y_n (1.5 - 0.5 x {y_n}^2)
\end{align}
$$

由于雷神之锤III引擎的图形计算中并不需要太高的精度，所以代码中只进行了一次迭代，二次迭代的代码则被注释。
