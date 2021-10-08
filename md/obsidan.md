
## 常用设置
不同平台或者不同用户可以使用独立的配置，例如使用`.obsidian_win`
![[Pasted image 20210927231208.png]]

### 配置默认笔记目录

设置默认新建笔记的目录，例如`md`
![[Pasted image 20210927231336.png]]
## 快捷键

![[cheatsheet_快捷键#obsidian]]
## 搜索

`foo bar` 搜索同时存在`foo`和`bar`的笔记
`foo OR bar` 搜索存在`foo`或`bar`的笔记
`foo -bar`搜索存在`foo`但不存在`bar`的笔记
`/[a-z]{3}/` 使用正则表达式去搜索，可以和上述规则组合，也可以和自己组合

## 插入绝对路径的文件

```
[file](file:///D:\download\obsidian-kanban-1.0.21.zip)
```

## todo
可点击完成
```
- [ ] 01
- [ ] 02
- [x] 03
```

- [ ] 01
	- [ ] 001
	- [x] 002
- [ ] 02
- [x] 03

##  插件

### [obsidian-admonition](https://github.com/valentine195/obsidian-admonition)

```ad-note
title: Nested Admonitions
collapse: open

Hello!

!!! ad-note
    title: This admonition is nested.
    This is a nested admonition!
    !!! ad-warning
        title: This admonition is closed.
        collapse: close


This is in the original admonition.
```

支持的类型有

```ad-abstract
```
```ad-info
```
```ad-tip
```
```ad-success
```
```ad-question
```
```ad-warning
```
```ad-failure
```
```ad-bug
```
```ad-example
```
```ad-quote
```

可以使用别名

| Type     | Aliases                     |
| -------- | --------------------------- |
| note     | note, seealso               |
| abstract | abstract, summary, tldr     |
| info     | info, todo                  |
| tip      | tip, hint, important        |
| success  | success, check, done        |
| question | question, help, faq         |
| warning  | warning, caution, attention |
| failure  | failure, fail, missing      |
| danger   | danger, error               |
| bug      | bug                         |
| example  | example                     |
| quote    | quote, cite                 |

也可以自己定义类型

```ad-li
```

可以使标题为空
```ad-info
title:
11
```
### obsidian-timeline

使用3个`+`作为一个时间段

```timeline
[line-5, body-3, active-color-interactive-accent-hover]
+ 1991
+ 出生
+ 嘎嘎嘎

+ 2015
+ 工作
+ 上海
```


### obsidian-kanban
- [ ] 待补充

### obsidian-icons-plugin

图标插件

`ris:Notification` `ris:Bank` `ris:AncientGate`

### find-unlinked-files

查找未被引用的笔记，文件，可用来删除无用的图片等
