---
layout: post
title: Jekyll相关1
---

来自: [Jekyll 教程三：目录结构](http://www.zhanxin.info/jekyll/2013-08-07-jekyll-directory-structure.html)

## 缘起
转载他人的成果，供自己查阅使用。

## 目录结构
----

使用Jekyll简单生成一个博客后，如同Rails一样，文件夹中自动生成了很多的文件。而运行 build 之后，又会把所有的文件都拉进去 _site 目录中，为毛？存在即具有合理性，这是Jekyll干的好事。
Jekyll的核心是一个文本转换引擎，将零散的文件、文本组合起来，生成一个个网页，其生成的博客拥有如下的目录结构:

    .
    ├── _config.yml
    ├── _drafts
    |   ├── begin-with-the-crazy-ideas.textile
    |   └── on-simplicity-in-technology.markdown
    ├── _includes
    |   ├── footer.html
    |   └── header.html
    ├── _layouts
    |   ├── default.html
    |   └── post.html
    ├── _posts
    │   └── 2013-08-07-welcome-to-jekyll.markdown
    ├── _site
    └── index.html

这些目录的介绍如下:

  文件/目录  |  描述
------------ | -----------------------------------------
  config.yml | 存储配置数据。很多全局的配置或者指令写在这里
 _drafts   	 | 存放为发表的文章。这些是没有日期的文件
 _includes   | 存放一些组件。可以通过`include file.ext`来引用
 _layouts    |	布局
 _posts 	   | 存放写文章，格式化为：YEAR-MONTH-DAY-title.md
 _site 	     | 最终生成的博客文件就在这里
 index.html  |	博客的主页
 other 	     | 例如静态文件 CSS，Images 和其他

根据自己的需要，将相关的文件放到博客目录下，通过`jekyll build`，就会自动将相关的文件组合生成所需的HTML文件，并将其放置到`_site`目录中。然后在通过git向github提交时，可以通过`.gitignore`忽略`_site`目录，Github默认会自动为gh-pages分支生成网页。

## 配置文件
----

Jekyll的配置非常的强大灵活，所有选项都可通过命令行或者`_config.yml`指定。

### 全局的配置项

 *设置*                                |   *选项或者参数*
-------------------------------------- | --------------------------------------
Site Source网站源文件的根目录          |  source: DIR
Site Destination生成网站的目录地址     |  destination: DIR
Safe是否使用自定义插件                 |  safe: BOOL
Exclude排除目录或者文件                |  exclude: [DIR, FILE, ...]
Include包含的目录或者文件              |  include: [DIR, FILE, ...]
Time Zone设置时区                      |  timezone: TIMEZONE

### 编译的命令选项

 设置                                  |  选项或者参数
-------------------------------------- | --------------------------------------
Regeneration                           | 
文件被修改时，启动自动生成             | -w, --watch
Configuration                          |
指定一个配置文件，可以覆盖_config.yml  |--config FILE
Drafts 处理和渲染草稿文章              |--drafts
Future 发布未来的文章                  | future: BOOL   --future
LSI 发布相关的文章                     | lsi: BOOL
发布相关的文章                         | lsi: BOOL --lsi
Limit Posts 限制文章发布的数量         | limit_posts: NUM --limit_posts NUM


### 服务器的命令选项

设置                  |            选项或者参数
--------------------- | -------------------------------------
Local Server Port     |
本地服务器启动的端口号|  port: PORT   --port PORT
Local Server Hostname |
服务器名称            |  host: HOSTNAME --host HOSTNAME
Base URL 基础链接     |  baseurl: URL  --baseurl URL

配置文件(YML)中，不允许有Tab，否则配置文件不能生效。默认的配置文件如下：

     source:      .
     destination: ./_site
     plugins:     ./_plugins
     layouts:     ./_layouts
     include:     ['.htaccess']
     exclude:     []
     keep_files:  ['.git','.svn']
     timezone:    nil
     
     future:      true
     show_drafts: nil
     limit_posts: 0
     pygments:    true
     
     relative_permalinks: true
     
     permalink:     date
     paginate_path: 'page:num'
     
     markdown:      maruku
     markdown_ext:  markdown,mkd,mkdn,md
     textile_ext:   textile
     
     excerpt_separator: "\n\n"
     
     safe:        false
     watch:       false    # deprecated
     server:      false    # deprecated
     host:        0.0.0.0
     port:        4000
     baseurl:     /
     url:         http://localhost:4000
     lsi:         false
     
     maruku:
       use_tex:    false
       use_divs:   false
       png_engine: blahtex
       png_dir:    images/latex
       png_url:    /images/latex
     
     rdiscount:
       extensions: []
     
     redcarpet:
       extensions: []
     
     kramdown:
       auto_ids: true
       footnote_nr: 1
       entity_output: as_char
       toc_levels: 1..6
       smart_quotes: lsquo,rsquo,ldquo,rdquo
       use_coderay: false
     
       coderay:
         coderay_wrap: div
         coderay_line_numbers: inline
         coderay_line_numbers_start: 1
         coderay_tab_width: 4
         coderay_bold_every: 10
         coderay_css: style
     
     redcloth:
       hard_breaks: true

## 后记
----
偶然遇到了一个问题，`Page build failed: File is a symlink`。这是由于在文件中通过include包含了不存在的文件导致的。没能想到特别好的处理方法。后来，想到了好的处理方法，那就是利用Liquid模板中的的逻辑判断。


