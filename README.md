[Jekyll](https://jekyllrb.com/)根据Markdown文本生成页面。

# 主题
用[minimal-mistakes主题](https://mmistakes.github.io/minimal-mistakes/)，具体配置可以到官网查询。

`bundle exec jekyll serve --host ip`启动web服务器


## 自定义主题

 `bundle show minimal-mistakes-jekyll`定位主题文件位置。
 ```bash
 /_layouts
/_includes
/_sass
 ```
 将需要的文件拷贝到自己工程的根目录，然后修改对应的对方。

 页面是用模板语言[Liquid](https://jekyllrb.com/docs/liquid/)写的。比如`{{ variable }}`这段内容会被variable替换。

 ### 修改scss/sass
 修改全局的css格式。
比如修改页面header的背景色。