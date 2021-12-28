[参考文档](https://katex.org/docs/supported.html)

基本 KaTex 首尾需要`$`包含,例如`$X_y$`表示$X_y$

### 常用

| 表达式                         | 示例                                    |
|:------------------------------ |:--------------------------------------- |
| `\{\}`                         | $\{\}$                                  |
| `\ge`                          | $\ge$                                   |
| `\le`                          | $\le$                                   |
| `\ne`                          | $\ne$                                   |
| `\cdots`                       | $\cdots$                                |
| `X_y`                          | $X_y$                                   |
| `X^{y}` `X^{y+1}`              | $X^{y}$                       $X^{y+1}$ | 
| `\hat{\delta}`                 | $\hat{\delta}$                          |
| `\theta`                       | $\theta$                                |
| `\varepsilon`                  | $\varepsilon$                           |
| `\Sigma`                       | $\Sigma$                                |
| `\omega`                       | $\omega$                                |
| `\lbrace \rbrace`              | $\lbrace \rbrace$                       |
| `\langle\rangle`               | $\langle\rangle$                        |
| `\vert`                        | $\vert$                                 |
| `\varrho`                      | $\varrho$                               |
| `\varrho`                      | $\vartheta$                             |
| `\delta`                       | $\delta$                                |
| `\cap`                         | $\cap$                                  |
| `\cup`                         | $\cup$                                  |
| `\displaystyle\bigcup_{i=1}^k` | $\displaystyle\bigcup_{i=1}^k$          |
| `\subset`                      | $\subset$                               |
| `\subseteq`                    | $\subseteq$                             |
| `\supset`                      | $\supset$                               |
| `\supseteq`                    | $\supseteq$                             |
| `\varepsilon\text{-}NFA`       | $\varepsilon\text{-}NFA$                |
| `\rightarrow`                  | $\rightarrow$                           |
| `\leftarrow`                   | $\leftarrow$                            |


### 逻辑与集合
|                                |                                |
| ------------------------------ | ------------------------------ |
| `\forall`                      | $\forall$                      |
| `\to`                          | $\to$                          |
| `\gets`                        | $\gets$                        |
| `\in`                          | $\in$                          |
| `\notin`                       | $\notin$                       |
| `\emptyset`                    | $\emptyset$                    |
| `\alpha \Rightarrow^n\alpha_n` | $\alpha \Rightarrow^n\alpha_n$ |
| `\displaystyle\sum_{i=1}^n`    | $\displaystyle\sum_{i=1}^n$    |
| `\Rightarrow`                  | $\Rightarrow$                               |


### 颜色

```latex
$$
\colorbox{black}{\color{white}123}
$$
```

$$
\colorbox{black}{\color{white}123}
\colorbox{#DDD}{\color{#777}123}
$$

### 其他
```ad-warning
title:
块状使用`$$`包含
```

```latex
\begin{cases}
   a &\text{if } b \\
   c &\text{if } d
\end{cases}
```

$$
 \begin{cases}
   a &\text{if } b \\
   c &\text{if } d
\end{cases}
$$


换行

```latex
\begin{align}
1 == 1 \\
2 == 2
\end{align}
```

$$
\begin{align}
1 == 1 \\
2 == 2
\end{align}
$$

```ad-fail
不支持中文
```
