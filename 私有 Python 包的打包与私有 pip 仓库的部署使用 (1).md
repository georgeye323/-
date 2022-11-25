在团队协作的 Python 项目开发中，我们可能会有这样的需求：

1. 需要抽象出一些通用的模块/包/工具类，供不同项目使用，而不是在每个项目中 copy 一份代码——这样既费时间，也不利于统一维护升级
2. 这些通用包应能直接通过 pip 命令安装使用，但出于安全要求，不能直接公开发布到公有的 pypi 源

本文章旨在提出并实践针对以上两个问题的解决方案：

1. 使用 setuptools 打包自己的 Python 包
2. 使用 git 源或搭建私有 pip 仓库的形式，对这些自有 Python 包进行管理


# Python 包的开发与打包

这里以建立一个最简单的 Python 包 `example_pkg` 为例，该包只有打印输出“Hello World”这一个最简单的功能，方便后面验证该包可以被正常导入与使用。

### 创建包目录结构

在本地创建以下目录结构：
```
/packaging_tutorial
  /example_pkg
    __init__.py
```

> 这里只是一个最简单的示例，实际的项目中，可能包含多个子包或模块，可以阅读 [Python documentation for packages and modules](https://docs.python.org/3/tutorial/modules.html#packages) 进一步了解。


创建此结构后，之后所有的修改操作都将在 `/packaging_tutorial` 目录下进行，执行 `cd packaging_tutorial` 进入该目录。

### 给包添加实际功能

下面我们为这个包添加一点实际功能——添加一个方法打印“Hello World”。

编辑 `/example_pkg/__init__.py` 在其中增加一个 `hello_world()` 方法：
```python
def hello_world():
    print('Hello World!')
```

### 创建 setup.py

setup.py 是 setuptools 的构建脚本，它告诉 setuptools 该如何对代码进行打包，并设置一些包的基本信息。

创建 setup.py 文件并输入以下内容：
```python
import setuptools

with open("README.md", "r") as fh:
    long_description = fh.read()

setuptools.setup(
    name="example-pkg",
    version="0.0.1",
    author="Author",
    author_email="author@example.com",
    description="A small example package",
    long_description=long_description,
    long_description_content_type="text/markdown",
    url="https://github.com/pypa/sampleproject",
    packages=setuptools.find_packages(),
    install_requires=['pytest>=4.0.0,<4.6.0'],
    classifiers=[
        "Programming Language :: Python :: 3",
        "License :: OSI Approved :: MIT License",
        "Operating System :: OS Independent",
    ],
)
```

`setup()` 的几个参数说明：

- `name` - 包的_分发名称_。只能包含字母、数字、`_` 和 `-`，它不能与其他包的名字重复。
- `version` - 包版本，可参考 **[PEP 440](https://www.python.org/dev/peps/pep-0440) **获取有关版本的更多详细信息。
- `author` 和 `author_email` - 标识别包的作者和作者邮箱。
- `description` - 一句话形式的包的简短说明。
- `long_description` - 包的详细说明。它会显示在 Python Package Index 包详细信息中。上面的例子中，详细说明直接加载 `README.md` 的内容，这是一种常见的方式。
- `long_description_content_type` - 设置用于包详细描述信息内容的语法类型。上面的例子中，使用的是 Markdown。
- `url` - 项目主页的URL。对于许多项目，这只是一个指向 GitHub、GitLab、Bitbucket 或类似代码托管服务的链接。
- `packages` - 应包含在分发包（[Distribution Package](https://packaging.python.org/glossary/#term-distribution-package)）中的所有 Python 导入包（[Import Packages](https://packaging.python.org/glossary/#term-import-package)）的列表。我们可以使用 `find_packages()` 自动发现所有包和子包，而不是手动列出每个包。上面的例子中，包列表将是 _example_pkg_，因为它是唯一存在的包。
- `install_requires` - 该包的依赖包，当该包通过 pip 安装时，会自动检索并安装依赖的包。上面的例子中，指明该包依赖于 pytest 这个包，并且需要 pytest 包的版本在 4.0.0 ~ 4.6.0 之间。
- `classifiers` - 告诉 Python Package Index 该包的一些额外元数据。上面的例子中，该包仅与 Python 3 兼容，根据 MIT 许可证进行许可，并且与操作系统无关。您应始终至少包含您的软件包所使用的 Python 版本、软件包可用的许可证以及您的软件包可使用的操作系统。有关 classifiers 的完整列表，请参阅：[https://pypi.org/classifiers/](https://pypi.org/classifiers/)。

除了上面提到的还有很多其他参数，更多详细信息，请参阅  [Packaging and distributing projects](https://packaging.python.org/guides/distributing-packages-using-setuptools/)。

### 创建 README.md

`README.md` 通常用作包的详细说明文档，告诉使用者该包各模块的功能和使用方法。

创建 `README.md` 文件并输入一些内容，可以按照你自己的想法自定义：
```markdown
# Example Package

This is a simple example package. You can use
[Github-flavored Markdown](https://guides.github.com/features/mastering-markdown/)
to write your content.
```

### 创建 LICENSE 许可协议

上传到 Python Package Index 的每个包都应包含许可证，告诉用户安装您的软件包可以使用您的软件包的条款。有关选择许可证的帮助，请访问 [https://choosealicense.com/]()。选择许可证后，打开 LICENSE并输入许可证文本。例如，如果选择了 MIT 许可证：
```
Copyright (c) 2018 The Python Packaging Authority

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.
```

如果这个包仅用于内部使用，可以选择不添加许可证。

### 生成可分发的包

经过上述步骤，该包的本地目录结构应如下所示：
```
/packaging_tutorial
  /example_pkg
    __init__.py
  setup.py
  LICENSE
  README.md
```

下一步，使用 setuptools 将你的包进行打包，生成分发包（[Distribution Packages](https://packaging.python.org/glossary/#term-distribution-package)）。分发包是可以上传到 Python Package Index 的文件，同样，它们也可已上传到私有 pip 仓库中，通过 [pip](https://packaging.python.org/key_projects/#pip) 命令进行安装。

确保已经安装了最新版本的 `setuptools` 和 `wheel`：
```bash
python3 -m pip install --user --upgrade setuptools wheel
```

> 如果安装时遇到问题，可参考：[Installing Packages](https://packaging.python.org/tutorials/installing-packages/)


在 `setup.py` 文件所在的目录运行下面命令进行打包：
```bash
python3 setup.py sdist bdist_wheel
```

如果一切顺利，会在 dist 目录中生成两个文件：
```
dist/
  example_pkg-0.0.1-py3-none-any.whl
  example_pkg-0.0.1.tar.gz
```

`tar.gz` 文件是源文件存档（[Source Archive](https://packaging.python.org/glossary/#term-source-archive)），`.whl` 是构建的分发（[Built Distribution](https://packaging.python.org/glossary/#term-built-distribution)），新版 pip 优先使用构建的分发进行安装，但如果需要，也可以回退到使用源文件存档安装。您应当始终上传源文件存档并为项目兼容的平台提供构建的分发。上面的示例中，示例包兼容任何平台，所以只需要一个构建的分发。

经过以上步骤，我们实现了一个最简单模块的开发和打包，下一步，就是让我们的这个包可以通过 pip 来安装使用了！


# Python 包的私有化分发

私有化分发，即不上传到公共的 Python Package Index（例如 pypi）中，这样可以保证你的包只在内部使用，避免所有人都可以通过 pip 公有源进行安装。

包的分发方式有多种，下面介绍几种最常见的方式。

## 通过文件分发安装

这是最简单的分发方式，以上一节中打包好的文件为例，可以直接执行以下命令安装：
```bash
pip install dist/example_pkg-0.0.1-py3-none-any.whl
```

如果需要在其他主机上安装该包，可以直接把文件拷贝到相应的主机，并执行 `pip install [打包好的 .whl 或 tar.gz 文件路径]` 安装。

> 直接通过文件进行分发安装的方式最简单，但在某些时候并不方便，因为这样无法对代码和包进行集中管理。下面介绍两种可以集中对包进行版本管理的方式。


## 通过 git 源的方式分发安装

一般情况下，我们都会把自己开发的包通过 git 进行版本控制，并上传到私有 git 仓库。pip 本身也提供了通过 git 协议直接拉取远程 git 仓库代码进行包安装的方式。

### 将包的源代码上传到 git 仓库

以上传到腾讯开发者平台的代码托管服务为例，在包的根目录 `/packaging_tutorial` 执行：
```bash
git init
git add .
git commit -am "My example-pkg"
git remote add origin git@git.dev.tencent.com:littlesky/example-pkg.git
git push -u origin master
```

成功上传后，我们就可以在任何有权限访问该 git 的主机上通过 pip 命令来安装这个包了。

### 安装包

如何获取安装的 git url 路径？我们只需要对 git 仓库的 ssh 路径做一点小修改。

例如，刚刚上传到 git 仓库的 ssh 地址为：
 `git@git.dev.tencent.com:littlesky/example-pkg.git`
将其中的 `:` 修改为 `/`，并在最前面添加 `git+ssh://` ，得到如下地址：
 `**git+ssh://**``git@git.dev.tencent.com``**/**``littlesky/example-pkg.git`
这就是我们用 pip 安装这个包时需要使用的地址了！

执行以下语句即可安装：
```bash
pip install git+ssh://git@git.dev.tencent.com/littlesky/example-pkg.git
```

### 通过 requirements.txt 管理包

我们通常会使用 `requirements.txt` 文件来管理项目中所有的包和版本，类似下面所示：
```
...
example-pkg==0.0.1
...
```

但因为我们自己开发的 `example-pkg-your-username` 包并不存在于任何 pip 仓库中，所以我们需要在 `requirements.txt` 文件中替换为完整的 git url 路径使其可以正确下载并安装。修改如下：
```
...
git+ssh://git@git.dev.tencent.com/littlesky/example-pkg.git
...
```

至此，我们就可以在任何有此 git 访问权限的主机上，通过 `pip install -r requirements.txt` 自动安装所有 Python 包了。

> 这种方式实现包的 pip 远程安装较为简单，但是安装时实际上是通过源码再次编译进行安装的，速度较慢，同时需要输入完整的代码仓库 git url 地址，使用时不方便。


## 通过搭建私有 pip 索引服务器的方式分发安装

更进一步，我们可能希望像 pypi 公有索引那样去上传和管理自己的私有包，而使用者也不用去为修改包的 git url 而担心，像正常使用 pip 一样安装使用这些私有包。这就需要我们自己来搭建 pip 索引服务器来实现了。常见的开源 pip 索引服务器主要有 [pypiserver](https://github.com/pypiserver/pypiserver)、[devpi](https://www.devpi.net/) 等。

[pypiserver](https://github.com/pypiserver/pypiserver) 功能相对简单，仅实现了 pip install、pip search 等简单命令。而 [devpi](https://www.devpi.net/) 功能更为强大：支持本地包的镜像和缓存、在 web 端查看包、索引继承、集群化，并提供一些更好用的客户端上传、安装、管理包的工具。

这里我们以 [devpi](https://www.devpi.net/) 为例来搭建 pip 索引服务器。

### 服务器安装 devpi

devpi 包含 devpi-client、devpi-server、devpi-web 等多个包：

- devpi-server 提供 pip 索引服务器
- devpi-client 可以管理和操作 devpi-server
- devpi-web 可以在网页访问查看 devpi-server 上包

我们在本地安装完整的 devpi 来进行测试：
```bash
pip install -U devpi
```

这将安装 devpi-client、devpi-server、devpi-web 等所有 devpi 包。

### 容器安装devpi
####  1、Dockerfile
```
FROM python:3.7-alpine

MAINTAINER yangyuhang@qq.com

RUN echo "http://mirrors.aliyun.com/alpine/v3.11/main" > /etc/apk/repositories && \
    echo "http://mirrors.aliyun.com/alpine/v3.11/community" >> /etc/apk/repositories

RUN apk update && \
    apk add --no-cache gcc g++ python python-dev py-pip mysql-dev linux-headers libffi-dev openssl-dev

RUN pip install devpi-server devpi-web devpi-client -i https://mirrors.aliyun.com/pypi/simple/ && \
    mkdir /devpi

EXPOSE 3141
ADD run.sh /
CMD ["/bin/sh","run.sh"]
```
#### 2、启动脚本
```
cat run.sh
#!/bin/sh
set -e
set -x
export DEVPISERVER_SERVERDIR=/devpi
[[ -f $DEVPISERVER_SERVERDIR/.serverversion ]] || initialize=yes

if [[ $initialize == yes ]]; then
    devpi-server --port 3141 --serverdir $DEVPISERVER_SERVERDIR --init
fi

devpi-server --host 0.0.0.0 --port 3141 --serverdir $DEVPISERVER_SERVERDIR
```
####  3、容器启动命令
```
docker run --name devpi -p 3141:3141 -v /data/devpi:/devpi --restart=always --cpus 0.5 -d registry-vpc.cn-hangzhou.aliyuncs.com/senguo-test/pypi:devpi
```

### 配置启动 devpi

`devpi quickstart` 命令会执行一些基础的 devpi 初始化步骤：

- 启动一个后台的 devpi-server，可通过 `http://localhost:3141` 访问
- 配置客户端工具 `devpi` 来连接最新启动的 devpi 服务
- 创建和登录一个用户，使用你当前默认的用户名和一个空的密码
- 创建一个 `dev` 索引然后直接使用它

让我们执行 quickstart 命令来触发一系列 devpi 命令快速设置一个 devpi 服务器：
```bash
$ devpi quickstart
--> $ devpi-server --start
2014-09-04 15:12:19,311 INFO  NOCTX DB: Creating schema
2014-09-04 15:12:19,353 INFO  [Wtx-1] setting password for user u'root'
2014-09-04 15:12:19,353 INFO  [Wtx-1] created user u'root' with email None
2014-09-04 15:12:19,353 INFO  [Wtx-1] created root user
2014-09-04 15:12:19,353 INFO  [Wtx-1] created root/pypi index
2014-09-04 15:12:19,367 INFO  [Wtx-1] fswriter0: committed: keys: u'.config',u'root/.config'
starting background devpi-server at http://localhost:3141
/tmp/home/.devpi/server/.xproc/devpi-server$ /home/hpk/venv/0/bin/devpi-server
process u'devpi-server' started pid=841
devpi-server process startup detected
logfile is at /tmp/home/.devpi/server/.xproc/devpi-server/xprocess.log
--> $ devpi use http://localhost:3141
using server: http://localhost:3141/ (not logged in)
no current index: type 'devpi use -l' to discover indices
~/.pydistutils.cfg     : no config file exists
~/.pip/pip.conf        : no config file exists
~/.buildout/default.cfg: no config file exists
always-set-cfg: no

--> $ devpi user -c test password=
user created: testuser

--> $ devpi login test --password=
logged in 'test', credentials valid for 10.00 hours

--> $ devpi index -c dev
http://localhost:3141/test/dev:
  type=stage
  bases=root/pypi
  volatile=True
  uploadtrigger_jenkins=None
  acl_upload=testuser
  pypi_whitelist=

--> $ devpi use dev
current devpi index: http://localhost:3141/test/dev (logged in as testuser)
~/.pydistutils.cfg     : no config file exists
~/.pip/pip.conf        : no config file exists
~/.buildout/default.cfg: no config file exists
always-set-cfg: no
COMPLETED!  you can now work with your 'dev' index
  devpi install PKG   # install a pkg from pypi
  devpi upload        # upload a setup.py based project
  devpi test PKG      # download and test a tox-based project
  devpi PUSH ...      # to copy releases between indexes
  devpi index ...     # to manipulate/create indexes
  devpi use ...       # to change current index
  devpi user ...      # to manipulate/create users
  devpi CMD -h        # help for a specific command
  devpi -h            # general help
docs at http://doc.devpi.net
```

经过上面的一系列命令，系统自动以当前登录用户的名称创建了一个空密码的账号，并且自动登录了进去，同时在这个用户下创建了名为 dev 的 pip 索引地址：http://localhost:3141/test/dev。

如需添加一个 test 用户，密码也为 test，可以手动执行 `devpi user` 命令：
```bash
devpi user -c test password=test
```

如需修改密码，可使用 `-m` 参数：
```bash
devpi user -m test password=test
```

索引地址、用户名和密码会在上传包时用到。

### 上传包到 devpi

下面我们把之前本地已经打包好的包上传到 devpi 服务器相应的索引上，这里使用官方推荐的工具 [twine](https://packaging.python.org/key_projects/#twine)，确保已安装最新版本的 twine：
```bash
pip install -U twine
```

#### 配置 .pypirc 文件

`.pypirc` 文件是多个工具使用的配置文件，例如 easy_install、twine。它包含有关在发布包时使用特定 pip 索引服务器的配置信息。

我们可以将我们 devpi 搭建的 pip 索引服务器的相关信息配置到 `~/.pypirc` 文件中，这样就不用在每次上传时都输入 pip 索引地址以及账号密码了。

新建或更新 `~/.pypirc` 文件，根据自己的需要命名并增加 pip 索引配置：
```diff
[distutils]
index-servers =
  pypi
  local
+  dev

[pypi]
username: <your_pypi_username>
password: <your_pypi_passwd>

[local]
repository: http://localhost:8080
username: senguodev
password: senguodev

+ [dev]
+ repository: http://localhost:3141/test/dev/
+ username: test
+ password: test
```

带 `+` 号的为添加的行，上面的示例中，将之前 devpi 搭建的源索引命名为了 `dev`：

- `[distutils]` 配置块可以配置多个 pip 索引服务器名称：
   - `index-servers` - 索引名称，可以配置多个，同时需要在下面建立对应名称的详细配置块
- `[dev]` 配置块就是我们用 devpi 搭建的 pip 索引服务器的相关信息，包含：
   - `repository` - 索引地址
   - `username` - 用户名
   - `password` - 密码

这样配置之后，上传包时就可以直接用 `dev` 这个索引名称来指定并上传到我们用 devpi 搭建的 pip 索引服务器了。

#### 注册包并上传

如果是要上传新包，而不是进行版本更新，那么需要先在 pip 源上进行包名注册：
```bash
twine register -r dev dist/example_pkg-0.0.1-py3-none-any.whl
```

然后进行上传：
```bash
twine upload -r dev dist/*
```

上面的命令会把我们之前生成的可分发包（存储在项目 `dist/` 目录）上传到名为 `dev` 的索引中（也就是我们用 devpi 搭建的 pip 索引服务器），此时用浏览器访问该索引地址 http://localhost:3141/test/dev/ 应当可以看到对应的包信息。

下面我们就可以使用 pip 命令来安装这个包了。

### 安装包

#### 配置 pip.conf 文件

`pip.conf` 文件是 pip 工具使用的配置文件，用于设置搜索、下载包时所使用的 pip 索引。

为了安装方便，不需要手动指定包的安装索引地址，我们先在 `pip.conf` 配置一下安装源信息：

新建更新 `~/.pip/pip.conf`，添加如下内容：
```diff
[global]
extra-index-url = http://localhost:3141/test/dev/
```

这会告诉 pip，在使用 pip install 安装包时，除了 pypi 公有源，同时从 `http://localhost:3141/test/dev/` 这个索引地址进行搜索安装。

#### 安装并测试

经过以上配置，我们现在可以像平时一样，在本地使用 pip 命令安装我们自己开发的 Python 包了！

安装 example-pkg-your-username 这个包：
```bash
pip install example-pkg
```

测试一下是否可以正常导入使用：
```bash
>>> import example_pkg
>>> example_pkg.hello_world()
Hello World!
```
 
It Works!
