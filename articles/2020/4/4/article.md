# 背景环境
早些时候，Github的项目里多了一个Actions标签页，因为没有需求，一直以来也没有去了解过它，大约是能进行一些自动化操作的功能。  
近期由于我的博客进行了改版，更新流程变得繁琐了，每次更新了新内容，需要在本地几个项目间pull来push去折腾几次才能发布，非常麻烦。  
昨天突然好奇心旺盛，找了几篇相关的博客，发现功能非常强大，可以满足我一次push后自动发布的需求，折腾了一番。  
~~这篇没写完的笔记也正好用来测试实现效果。~~  
继续写。 

# 需求
我提交的文章会在Github上存档，同时，存档库会作为Github Pages仓库的submodule包含在内。往常我需要提交文章存档，并且更新本地的Github Pages仓库及其submodule，然后上传到Github后由其编译发布，文章资源也借此实现的CDN功能被我的个人站点引用。  
而现在我需要利用Github Actions来实现提交文档后自动更新Github Pages仓库及submodule并且上传的功能。

# 踩坑流程
Github Actions类似于在服务器提供一个虚拟机，有一些默认变量，也可以自行注入一些隐私变量，然后执行设定的YML脚本，功能还是比较强大的。  
同时，每次更新脚本都会自动执行，并且在任务中还可以看到执行的输出内容。因此，可以通过在脚本中插入类似echo、cat、ls等指令来查探脚本运行环境进行debug。  

## 设计脚本功能
参考我的实际操作流程，脚本应该完成如下功能：  
在代码提交时触发 =》 clone一份Github Pages仓库 =》 cd进入目标路径 =》 更新子模块 =》 提交Github Pages仓库

## Clone的坑
直接clone目标仓库的时候报错，提示：
```bash
Cloning into 'xxxxx'...
Warning: Permanently added the RSA host key for IP address 'xxxxxx' to the list of known hosts.
git@github.com: Permission denied (publickey).
fatal: Could not read from remote repository.
```
这个应该是运行的虚拟机里没有设置对应的秘钥，但是查找相关资料的时候，找到了一个名为actions/checkout的脚本。  
没错，Github Actions可以运行别人写好的脚本！  
这个脚本可以通过配置参数来将指定代码库clone到指定目录
代码段如下：
```yml
- name: Checkout
    uses: actions/checkout@v2.1.0
    with:
        repository: xxxxxxxx
        ssh-key: ${{ secrets.deploy_key }}
        submodules: recursive
        path: ./xxxxxxx
```
这里的ssh-key要说明一下，${{secrets.deploy_key}}就是前面提到的自行注入的隐私变量了，secrets指向的是当前脚本运行仓库Setting项里的Secrets子项里添加的隐私变量，其中deploy_key里保存的是目标库Deploy keys里秘钥对的私钥。  
因此，我们也需要在目标库的Setting项里的Deploy keys子项里添加上述私钥对应的公钥，以使运行仓库有权限访问目标仓库。  
这样有一个好处，就是这个脚本似乎将ssh-key给放入到了运行环境里，后面再运行git操作就不再需要设置秘钥了。  
注意一下path，这个比较坑，如果不指定一个其他路径，它会将目标仓库的内容放到以运行仓库命名（默认）的目录里，然后提交时的目标就自动变成了运行仓库，带来一系列报错。  

## 更新子模块的坑
虽然在拉取代码的时候指定了submodules为recursive，但是并没有更新submodule的版本，因此需要特别地指定更新子模块。 
因此在代码里增加一句：
```yml
git submodule update --recursive --remote
```

## 提交的坑
走完上面的流程，理论上事情已经做得差不多了，但是到了这一步可能会有意外发生，因为git commit需要你配置你的个人信息。  
添加两行代码，其中的变量可以硬编码，也可以存到secrets隐私变量里：
```yml
git config --global user.email "xxxxxxx"
git config --global user.name "xxxxxx"
```

## 成品代码
```yml
# This is a basic workflow to help you get started with Actions

name: CI

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the master branch
on:
  push:
    branches: [ master ]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      - name: Checkout
        uses: actions/checkout@v2.1.0
        with:
          repository: xxxxxxxxxxxxxxxxxxxxxxxx
          ssh-key: ${{ secrets.deploy_key }}
          submodules: recursive
          path: ./xxxxxxxxxxxxxxx
      - name: push
        run: |
          cd ./xxxxxxxxxxxxxxxx
          git config --global user.email "xxxxxxxxxxxxxx"
          git config --global user.name "xxxxxxxxxxxx"
          git submodule update --recursive --remote
          echo Update Done
          git add *
          git commit -m 'auto publish'
          git push
          echo Push Done

```

# 参考文献
1. [GitHub Actions 入门教程](http://www.ruanyifeng.com/blog/2019/09/getting-started-with-github-actions.html) 
2. [GitHub Actions Documentation](https://help.github.com/en/actions)