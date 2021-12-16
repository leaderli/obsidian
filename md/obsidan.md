Releases · denolehov/obsidian-git · GitHubov/obsidian-git · GitHubov/obsidian-git · GitHub户可以使用独立的配置，例如使用`.obsidian_win`
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

### obsidian-copy-block-link
[📒 obsidian-copy-block-link](https://github.com/mgmeyers/obsidian-copy-block-link/releases)
### kanban
[📒 obsidian-kanban ](https://github.com/mgmeyers/obsidian-kanban/releases)
### obsidian-switcher-plus
[📒 obsidian-switcher-plus](https://github.com/darlal/obsidian-switcher-plus/releases)
### obsidian_supercharged_links
[📒 obsidian_supercharged_links](https://github.com/mdelobelle/obsidian_supercharged_links/releases)
### obsidian-git
[📒 obsidian-git](https://github.com/denolehov/obsidian-git/releases)
### recent-files-obsidian 
[ 📒 recent-files-obsidian](https://github.com/tgrosinger/recent-files-obsidian)
### obsidian-admonition
[📒 obsidian-admonition](https://github.com/valentine195/obsidian-admonition)

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
```ad-error
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


### 自定义插件

参考官方插件和其他插件编写的
 [obsidian-sample-plugin](https://github.com/obsidianmd/obsidian-sample-plugin) 
 [obsidian-admonition](https://github.com/valentine195/obsidian-admonition)
 
```ts
import { App, Modal, Notice, Plugin, PluginSettingTab, Setting } from 'obsidian';

function startsWithAny(str: string, needles: string[]) {
	for (let i = 0; i < needles.length; i++) {
		if (str.startsWith(needles[i])) {
			return i;
		}
	}
	return false;
}

function getParametersFromSource(src: string) {

	const keywordTokens = ["color:", "size:"];
	const keywords = ["color", "size"];
	let lines = src.split("\n");
	let skipLines = 0;
	let params: { [k: string]: string } = {};

	for (let i = 0; i < lines.length; i++) {
		let keywordIndex = startsWithAny(lines[i], keywordTokens);

		if (keywordIndex === false) {
			break;
		}

		let foundKeyword = keywords[keywordIndex];

		if (params[foundKeyword] !== undefined) {
			break;
		}

		params[foundKeyword] = lines[i]
			.substr(keywordTokens[keywordIndex].length)
			.trim();
		++skipLines;
	}

	let { color, size } = params;
	let content = lines.slice(skipLines).join("\n");

	if (color === undefined || color.trim() === "") {
		color = 'coral'
	}
	if (size === undefined || size.trim() === '') {
		size = 'medium'
	}

	return { color, size, content };
}
export default class MyPlugin extends Plugin {

	async onload() {

		this.registerMarkdownCodeBlockProcessor('li-color', (source, el, ctx) => {
			const params = getParametersFromSource(source);
			el.innerText = params.content
			el.removeAttribute('class')
			el.setAttribute('style', 'color:' + params.color + ';font-size:' + params.size)
		})
	}

}

```


## 其他


vim下光标位置不对

关闭`minimal theme setting - fancy cursor`


自定义的插件和css文件可能会差生奇奇怪怪的问题