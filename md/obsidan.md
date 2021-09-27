
## 配置
#todo 
## 快捷键

![[快捷键#obsidian]]
## 搜索

`foo bar` 搜索同时存在`foo`和`bar`的笔记
`foo OR bar` 搜索存在`foo`或`bar`的笔记
`foo -bar`搜索存在`foo`但不存在`bar`的笔记
`/[a-z]{3}/` 使用正则表达式去搜索，可以和上述规则组合，也可以和自己组合

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

### obsidian-admonition

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