# Coscidata 开发手册

> 用来记录前后端代码风格、开发环境配置、服务器脚本等

<!-- TOC -->

- [技术栈](#技术栈)
- [加速开发](#加速开发)
    - [brew 换源](#brew-换源)
    - [pip 换源](#pip-换源)
    - [npm 换源](#npm-换源)
- [开发环境](#开发环境)
    - [后端环境](#后端环境)
        - [Python3](#python3)
        - [virtualenvwrapper](#virtualenvwrapper)
        - [格式化代码](#格式化代码)
- [最佳实践](#最佳实践)
    - [React 实践](#react-实践)
        - [Container and Presentational Components](#container-and-presentational-components)
    - [Flask 实践](#flask-实践)
        - [Model 定义](#model-定义)
        - [API 定义](#api-定义)
    - [MySQL 实践](#mysql-实践)
        - [使用 utf8mb4](#使用-utf8mb4)

<!-- /TOC -->

## 技术栈

后端：[Python3](https://www.python.org/download/releases/3.0/) + [flask](http://flask.pocoo.org/) + [mysql](https://www.mysql.com/)

前端：[React](https://reactjs.org/) + [Ant Design](https://ant.design/index-cn)

服务器：[Centos 7](https://www.centos.org/)

## 加速开发

因为不可抗力，前后端系统安装软件及依赖时，有时会非常缓慢，强烈安利以下方式，节约时间。

### brew 换源

使用中科大源（或者清华源，不过最近经常抽风，并且修复速度慢，先不推荐）

- [替换 homebrew 默认源](https://lug.ustc.edu.cn/wiki/mirrors/help/brew.git)
- [替换 homebrew bottles 默认源](https://lug.ustc.edu.cn/wiki/mirrors/help/homebrew-bottles)

### pip 换源

[使用阿里源](http://mirrors.aliyun.com/help/pypi)

### npm 换源

~~使用淘宝源~~

1. ~~`npm config set registry https://registry.npm.taobao.org`~~
1. ~~[cnpm](https://npm.taobao.org/)（使用 create-react-app 时不会走 cnpm，所以更推荐第一个方法）~~

淘宝源，属于玄学，还是用官方源保平安吧。宁可慢点。可以在命令行中使用 `export https_proxy` 通过代理来加速。

```shell
npm config set registry "http://registry.npmjs.org/"
```

## 开发环境

### 后端环境

#### Python3

##### 安装

```bash
brew install python3
```

#### virtualenvwrapper

##### 安装

用来管理 python 虚拟环境

```bash
pip3 install virtualenvwrapper
```

添加下列代码到 ~/.zshrc

```zsh
# virtualenvwrapper
if [ `id -u` != '0' ]; then
  export VIRTUALENV_USE_DISTRIBUTE=1        # <-- Always use pip/distribute
  export WORKON_HOME=$HOME/.virtualenvs       # <-- Where all virtualenvs will be stored
  source /usr/local/bin/virtualenvwrapper.sh
  export PIP_VIRTUALENV_BASE=$WORKON_HOME
  export PIP_RESPECT_VIRTUALENV=true
fi
```

##### 使用

在项目根目录，创建虚拟环境

```bash
mkvirtualenv seattle -p python3
```

激活虚拟环境

```bash
workon seattle
```

其它命令

```bash
deactivate # 退出虚拟环境
cdvirtualenv # 进入虚拟环境文件夹
```

#### 格式化代码

使用 [yapf](https://github.com/google/yapf)、[isort](https://github.com/timothycrosley/isort) 和 [flake8](http://flake8.pycqa.org/en/latest/) 来检查、格式化代码

##### 安装

```bash
pip3 install yapf isort flake8

curl https://raw.githubusercontent.com/coscidata/about_code/master/python/.isort.cfg > ~/.isrot.cfg
```

##### 使用

```bash
yapf -ir . -e migrations
isort -rc --atomic .
```

##### 编辑器配置

###### vim

```config
Plug 'fisadev/vim-isort'
autocmd FileType python nnoremap <leader>a :0,$!yapf<Cr>
```

###### vscode

参考[文章](https://github.com/DonJayamanne/pythonVSCode/wiki)配置并使用 flake8 和 yapf

## 最佳实践

### React 实践

#### Container and Presentational Components

在 React 开发中，我们推荐将一个功能拆分为一个 Container 组件和作为其子组件的多个 Presentational 组件。

Container 主要负责获取数据、以及和服务端的交互，然后通过 props 将特定的数据传递给对应的 Presentational 子组件。

这样的好处是能够保证 Presentational 组件拥有良好的复用性，以及它的数据结构是清晰地，大部分的 Presentational 组件都可以写成纯函数的形式：

```javascript
// 没错，每个介绍的文章都要以 CommentList 举例
const CommentList = props => {
  const { comments, handleDelete } = props;
  return (
    <ul>
      {comments.map(<Comment onDelete={handleDelete} />)}
    </ul>
  );
};
```

每个 Presentational 组件都会通过 props 中的数据和事件回调与其父级的 Container 组件通信。

使用纯函数编写的组件是无法使用 State 的，但并不等于所有 Presentational 组件都不允许使用 State。我们只是不推荐在 Presentational 组件中直接获取数据，但是依旧可以使用 State 去维持一些样式上的状态，例如 Loading 状态或是切换不同的样式。

我们并不强制要求一个功能只能有一个 Container 组件和多个 Presentational 组件，试想一下如果一个功能所使用的组件数量很多、层级很深，所有事件回调和数据获取都放在最外层的 Container 组件下的话，反而会使代码变得更加混乱，不过我们暂时也没遇到这种复杂的场景。

参考资料：

- [Container Components](https://medium.com/@learnreact/container-components-c0e67432e005)
- [Presentational and Container Components](https://medium.com/@dan_abramov/smart-and-dumb-components-7ca2f9a7c7d0)

### Flask 实践

#### Model 定义

我们使用 Flask-SQLAlchemy，并在其基础的 ORM 上添加一些更友善的约定，使开发变得更加安全和高效。

在 Model 层，我们会在 `db.Model` 之上抽象出一个 `BaseModelMixin`，定义一些基础的字段和方法。

基础字段有：

- `status`：主要用于逻辑删除，逻辑删除相对于物理删除的好处不必多说
- `created_at`：每行记录的创建时间
- `updated_at`：每行记录的更新时间

创建时间和更新时间的默认值都放在了数据库层面，原因是我们不希望一些应用外的脚本、手动操作对数据库的更改没有触发这些字段的设置，导致排查时的误判。

基础方法主要就是一个 Model 最基础的增删改查功能。

对于创建和修改，我们更倾向于使用声明式的字段权限控制以替代在 API 里写重复度非常高的校验代码，例如：

```python
class User(BaseModelMixin):
    __tablename__ = 'user'

    user_id = db.Column(db.Integer, primary_key=True, autoincrement=True)
    username = db.Column(
        db.String(255),
        nullable=False,
        info=dict(creatable=True))
    password = db.Column(
        db.String(255),
        nullable=False,
        info=dict(creatable=True, editable=True))
    nickname = db.Column(
        db.String(255),
        nullable=False,
        info=dict(creatable=True, editable=True))
```

在 User 中，其 `user_id` 是通过自增生成的，不允许手动设置，`username` 可以在创建时设置，但不允许修改，而 `password` 和 `nickname` 则是既可在创建时设置，也可以在修改时设置。

这样对于最基础的 Create 和 Update 方法中，我们可以放心的将整个 Request 映射到 Model 中，不用一个个字段进行赋值，也不需要担心接口被传入一些重要字段。

而对于查询和删除，我们将删除方法实现是修改 `status` 为 `DELETED`，又在 `query` 中默认添加了 `filter(self.status != Status.DELETED.value)`，对使用者隐藏了逻辑删除的实现。

另外，所有查询的方法都需要将实现写在 Model 里，不希望 API 去自行拼接查询方法。并且如果有复用的复杂查询，推荐通过 `@hybrid_property` 或是 `@hybrid_method` 封装起来。

#### API 定义

我们的 API 尽量贴近 RESTful 定义，但是不强求。一个基础的例子：

- 创建用户：`POST /users/`
- 修改用户信息：`PUT /users/1`
- 用户列表：`GET /users/`
- 删除用户：`DELETE /users/1`
- 点赞：`PUT /users/1/like`
- 取消点赞：`DELETE /users/1/like`

但是这样很多 API 的语义其实并不明确，举几个反例：

- 对用户资源的创建在很多场景下等同于注册，这时比起 `POST /users/` 用 `POST /signup` 或是 `POST /register` 会更好一些。
- 如果修改、删除的资源是确定的，那就不需要加入主键了，例如一个用户只能修改它自己的资料，那么 `PUT /users/1` 这种 API 定义就是累赘。
- 查询接口尽量起一些更有意义的别名，比如 `/following` 和 `/followers` 要比 `/users?type=following/followers` 要好很多。

对于基础的创建、更新、删除接口，我们固定会返回 Model 当前状态的完整数据。这样在大部分场景下都可以减少客户端、前端的一次额外请求刷新或是自己编写替换逻辑。

### MySQL 实践

#### 使用 utf8mb4

编辑 `/etc/my.cnf` 添加配置

```cnf
[client]
default-character-set = utf8mb4

[mysql]
default-character-set = utf8mb4

[mysqld]
character-set-client-handshake = FALSE
character-set-server = utf8mb4
collation-server = utf8mb4_unicode_ci
```

使用下面命令检查编码是否正确

```mysql
SHOW VARIABLES WHERE Variable_name LIKE 'character\_set\_%' OR Variable_name LIKE 'collation%';
```