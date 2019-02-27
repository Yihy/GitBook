---
title: Github项目使用多CI构建指南
---
# Github项目使用多CI构建指南

## 起因
  我最近在捣鼓电子书，用python写了一个脚本转换工具，写的脚本为了使用方便，用pyinstaller打包，生成多平台（macos、linux、windows）的执行文件。打包出windows可执行文件需要在windows上编译，不同平台的可执行文件需要在不同的平台上编译，三个平台打包下来。收集文件后再一个个上传到github的releases，真的是非常麻烦，本来我只是想写个脚本省事，结果却因为打包上传耗费大量的时间。

  怎么解决这个问题呢？项目放在github上面，在github上面有一些免费的CI服务，
比如travis-ci、appveyor等构建服务。
写的项目放在github上面，github上面有一些开源的免费的CI服务

[convert-ebook](https://github.com/Yihy/convert-ebook)这个是工具项目的地址，大家可以参照我的配置，配置自己的多平台构件

## 构建服务介绍
我使用travis-ci、appveyor这两个解决了构建问题。CI服务使用配置文件执行构建项目。
下面我就以我写的工具为例介绍，如何使用两个服务自动构建项目。
使用travis-ci构建linux、macos版本的可执行文件，使用appveyor构建windows可执行文件
### travis-ci
travis-ci网址 [https://travis-ci.org](https://travis-ci.org) ，打开后直接使用github登录就行，按提示添加项目就行。

 使用`.travis.yml`文件组织构建，文件需要存放在项目根目录，如图
![-w309](http://md.yihy.cc/2019-02-24-15510153267733.jpg)
travis-ci支持linux、macos镜像，多种语言、构建工具打包项目。


贴配置
```yaml
# 管理员权限
sudo: required
# 使用python环境
language: python
# python版本
python:
  - "3.5.1"
matrix:
  # 使用的系统镜像（osx、linux）
  include:
    - os: osx
      # travis-ci osx镜像使用python3有问题，会导致构建失败。这里使用generic覆盖上面的python
      language: generic
    - os: linux

# 执行安装前，用shell做的一些额外的操作。这里用来处理osx python版本的问题
before_install: |
  if [ "$TRAVIS_OS_NAME" == "osx" ]; then
    # 我测试了ci的osx镜像，发现存在python3，我需要python指向的版本是2的
    sudo ln -f $(which python3) $(which python)
    sudo ln -f $(which pip3) $(which pip)
    python --version
  fi
# 准备需要的构建环境
install:
  # 安装需要使用依赖包
  - pip install threadpool
  # 安装构建工具
  - pip install pyinstaller
# 构建项目
script:
  - pyinstaller ./convert-ebook.py -F --add-data ./lib/kindlegen:lib/kindlegen
  - mv ./dist/convert-ebook ./dist/convert-ebook-$TRAVIS_OS_NAME
# 发布项目
deploy:
  # 标志发布到github的releases
  provider: releases
  # 发布使用的文件或目录
  file: "dist/convert-ebook-$TRAVIS_OS_NAME"
  # 发布到github需要使用的token
  api_key: $api_key
  # 跳过清理文件 发布文件时，一定要配置为true
  skip_cleanup: true
  # 如果发布时，存在相同的tag版本，继续上传
  # 这一步很关键，多个构件发布releases，存在相同tag会写入到该releases中
  overwrite: true
  # 触发条件，
  on:
    branch: master
    # 必须存在tag时才能发布
    tags: true
```



### appveyor
appveyor网址 [https://ci.appveyor.com](https://ci.appveyor.com) ，打开后直接使用github登录就行，按提示添加项目就行。
appveyor支持用windows、ubuntu等镜像

需要使用`appveyor.yml`文件组织构件，文件需要存放在项目根目录，如上一张图

```yaml
environment:
  matrix:
    - PYTHON: "C:\\Python36"
      PYTHON_VERSION: "3.6.x" # currently 3.6.5
      PYTHON_ARCH: "32"
# 安装python
install:
  - ps: if ($env:APPVEYOR_PULL_REQUEST_NUMBER -and $env:APPVEYOR_BUILD_NUMBER -ne ((Invoke-RestMethod `
      https://ci.appveyor.com/api/projects/$env:APPVEYOR_ACCOUNT_NAME/$env:APPVEYOR_PROJECT_SLUG/history?recordsNumber=50).builds | `
      Where-Object pullRequestId -eq $env:APPVEYOR_PULL_REQUEST_NUMBER)[0].buildNumber) { `
      throw "There are newer queued builds for this pull request, failing early." }
  - ECHO "Filesystem root:"
  - ps: "ls \"C:/\""

  - ECHO "Installed SDKs:"
  - ps: "ls \"C:/Program Files/Microsoft SDKs/Windows\""

  # Install Python (from the official .msi of https://python.org) and pip when
  # not already installed.
  - ps: if (-not(Test-Path($env:PYTHON))) { & appveyor\install.ps1 }

  # Prepend newly installed Python to the PATH of this build (this cannot be
  # done from inside the powershell script as it would require to restart
  # the parent CMD process).
  - "SET PATH=%PYTHON%;%PYTHON%\\Scripts;%PATH%"
  - "python -m pip install --upgrade pip"
  - "%CMD_IN_ENV% pip install pyinstaller"
  - "%CMD_IN_ENV% pip install threadpool"
# 构件脚本
build_script:
  # Build the compiled extension
  - ps: "ls"
  - "%CMD_IN_ENV% pyinstaller ./convert-ebook.py -F --add-data lib/kindlegen;lib/kindlegen"
test_script:
  - ps: "ls dist"
# 如果要发布文件，这里一定配置
artifacts:
  - path: dist/*.exe
deploy:
  - provider: GitHub
    auth_token:
      secure: Iw3T58CTmwWRa79KqeIZ/ZE9qXBQD2mTaIa4mK3Hp/4id+igJqEC4GePD5MhCAxS
    prerelease: false
    # 多构件环境下需要为true
    force_update: true
    # 发布条件
    on:
      # 存在tag时，才能上传。这个和master是冲突的，配置时二选一
      APPVEYOR_REPO_TAG: true

```

### 多构件组合
我是坑过来的。。。。。。最开始的时候没有配置发布时的限制条件`deploy.on`，push了一次，两个构件构件项目后，会发布到releases。又会重新触发构件事件，然后死循环了。来来回回推了50多个release，一个一个删的我要吐血。

现在说一些配置的关键点
- 使用git的tag功能，当需要发布的时候，打tag，推送到仓库。构件上传时会根据该次提交的id去匹配对应的tag进行release发布。
- 打tag要写描述，描述会作为release的描述。`git tag release-v1.0.0 -m "tag描述"`
- CI服务要配置在tag事件才能发布
    - travis-ci  `deploy.on.tags: true`
    - appveyor  `deploy.on.APPVEYOR_REPO_TAG: true`
- 发布时要打开强制上传，不能存在releases就失败了。
    - travis-ci  `deploy.overwrite: true`
    - appveyor  `deploy.force_update: true`
- 不同环境下打包上传的文件名不要相同，否则会覆盖掉


使用appveyor配置上传的文件如下，官方文档描述发布github有坑
```yaml
# 这里必须配置
artifacts:
  - path: dist/*.exe
    name: myartifacts
deploy:
  - provider: GitHub
    # 这里是可选
    artifact: myartifacts
```
这是我发布出来的效果图
![-w334](http://md.yihy.cc/2019-02-24-15510192871334.jpg)

## 结语
我在这里简单介绍了两种ci的基本配置，主要将怎么组合多种构件共同发布release。
如果你想了解配置你需要的构建环境，请查看它们的官网文档，里面有不同环境的模板。