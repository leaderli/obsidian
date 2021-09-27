# html


### 合并单元格

```html
<table border="1" width="200" height="100" cellspacing="0" cellpadding="4">
    <tr>
        <td colspan="2">11</td>
        <td>13</td>
    </tr>
    <tr>
        <td rowspan="2">21</td>
        <td>22</td>
        <td>23</td>
    </tr>
    <tr>
        <td>32</td>
        <td>33</td>
    </tr>
</table>
```
效果如下：

<table border="1" width="200" height="100" cellspacing="0" cellpadding="4">
    <tr>
        <td colspan="2">11</td>
        <td>13</td>
    </tr>
    <tr>
        <td rowspan="2">21</td>
        <td>22</td>
        <td>23</td>
    </tr>
    <tr>
        <td>32</td>
        <td>33</td>
    </tr>
</table>

- `colspan`合并纵向的单元格，需要删除右边多余的单元格
- `rowspan`合并横向的单元格，需要删除下边多余的单元格


### 自定义列表

```html
<dl>
    <dt>Coffee</dt>
    <dd>Black hot drink</dd>
    <dd>Black hot drink</dd>
    <dt>Milk</dt>
    <dd>White cold drink</dd>
    <dd>White cold drink</dd>
</dl>
```

<dl>
    <dt>Coffee</dt>
    <dd>Black hot drink</dd>
    <dd>Black hot drink</dd>
    <dt>Milk</dt>
    <dd>White cold drink</dd>
    <dd>White cold drink</dd>
</dl>

## Emmet

Emmet是一个快速编写html或css的插件工具，常用语法如下，使用自动tab补全

1. `div#test `
    ```html
    <div id="test"></div>
    ```
2. `div.test`
    ```html
    <div class="test"></div>
    ```
3. `div>ul>li>p `
    ```html
    <div>
    <ul>
        <li>
        <p></p>
        </li>
    </ul>
    </div>
    ```
4. `div+ul+p`
    ```html
    <div></div>
    <ul></ul>
    <p></p>

    ```
5.  `div*5`（*号后面添加数字表示重复的元素个数）
6. `ul>li.test$*3 `
    ```html
    <ul>
        <li class="test1"></li>
        <li class="test2"></li>
        <li class="test3"></li>
    </ul>

    ```
7. ``

    

