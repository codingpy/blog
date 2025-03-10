# 平方根倒数速算法

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

IEEE 浮点表示：

$$
\vec x = [s_{31}, \underbrace{e_{30}, \dots, e_{23}}_\text{exp}, \underbrace{f_{22}, \dots, f_0}_\text{frac}]
$$

$$ V = (-1)^s \times M \times 2^E $$

其中， $M = 1 + f$ ， $E = e - 127$ 。

推导：

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
  (f_y + \sigma) + (e_y - 127) & = -\frac{1}{2} [(f_x + \sigma) + (e_x - 127)] \\
  (f_y + e_y) \times 2^{23} & = \frac{3}{2} (127 - \sigma) \times 2^{23} - \frac{1}{2} (f_x + e_x) \times 2^{23} \\
  \text{B2T}_{32}(\vec y) & = R - \frac{1}{2} \text{B2T}_{32}(\vec x)
\end{align}
$$

其中, $R = \text{0x5f3759df}$ 。

牛顿法：

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
