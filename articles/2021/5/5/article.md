# blog-tool是啥
blog-tool是一个基于node的博文本地管理工具。  
[代码仓库地址](https://github.com/cyx7788414/blog-tool)  

# 为什么要开发blog-tool
我用github仓库来保存我的.md博文，计划让站点从github pages引用博文内容，但博文的分类、标签、时间、作者等信息无法跟随博文正文保存，而我的静态文本也不想使用数据库来保存。  
因此我开发了这个blog-tool工具，它可以在新建文章时根据年月来进行归档，通过一个json索引文件来保存博文信息，满足了我的使用需求。

# blog-tool里的概念
## base path
base path是blog-tool工作的基础路径，该路径下有索引文件index.json，以及保存博文的目录articles，articles目录内分别按年、月、id三层保存博文及附件，目录结构大致如下：
```shell
.
├── articles
│   └── 2019
│       └── 12
│           ├── 1
│           │   └── article.md  
│           └── 2
│               └── article.md
└── index.json
```

## type
type是博文的分类，用于描述博文的类别，每篇博文只能有一个分类。  

## tag
tag是博文的标签，用于描述博文的关键词，每篇博文可以有多个标签。  

## status
博文有如下三个状态：
```shell
    editing： 编辑中
    published： 已发布
    deleted： 已删除
```

# blog-tool里的菜单项
## init
用于初始化base path，新建index.json与articles目录默认当前执行路径，可以通过--path (-p) 参数指定目标路径，无法重复初始化，路径必须已存在。

## add
用于在当前月份的路境内新增一个最新id的文章目录，建好.md文件，必须用--name (-n)参数输入文章标题，可以通过--path (-p)指定操作的基础路径，--date (-d)参数指定发表时间（用于加入历史文章或未来文章）并且参数必须是可以被Date对象解析的合法时间字符串，--auther (-a)参数来标记作者。

## update
用于在修改了.md文件后更新对应博文在index.json内的信息，或重新修改文章的分类/标签/状态，可以通过--path (-p)指定操作的路径（含有article.md文件的路径）（默认当前路径），--auther (-a)参数来修改作者，--name (-n)参数来修改文章标题。

## delete
用于删除一篇文章，可以通过--path (-p)指定操作的路径（含有article.md文件的路径）（默认当前路径），--force (-f)参数将默认的“delete属性置为true”行为变为“彻底从index.json与本地删除”。

## attr
用于管理分类与标签，可以通过--list (-l), --rename (-r), --delete (-d)参数来在选择操作分类或标签后执行有交互的展示/修改/删除操作，可以通过--path (-p) 参数指定目标路径。

## search
可以对文章进行模糊搜索，--path (-p) 参数指定目标路径，其余参数如下：
```shell
  --tag, --ta             标签名子串         [string]
  --type, --ty            类名子串        [string]
  --name, -n              标题子串        [string]
  --auther, -a            作者子串      [string]
  --earliestcreate, --ec  最早新建时间  [string]
  --latestcreate, --lc    最晚新建时间   [string]
  --earliestupdate, --eu  最早更新时间  [string]
  --latestupdate, --lu    最晚更新时间    [string]
  --status, -s            文章状态子串      [string]
```

# blog-tool的实践案例
我的个人站点[卡比窝](www.kabiwo.com)中的文章部分采用了这个工具进行写作，本地写完后同步到github的远程仓库，github上配置的actions会自动将仓库内容合并至github pages，作为个人站点的内容cdn。  
随后，我的个人站点页面会从github pages获取index.json作为索引，并且在进入文章页后拉取对应的.md文件，随后将其编译为html嵌入页面。  
.md的css样式是开源的，网上很多。  
至于为什么要这么麻烦，那自然是因为我没有钱了，只有一个共享php空间。