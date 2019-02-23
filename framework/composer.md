# Composer
Composer 是 PHP 的一个依赖管理工具。声明所依赖的代码库，就会在项目中安装他们。

Composer 代码库包括"packages" 和 "libraries"，默认情况下它不会在全局安装任何东西，仅仅是一个依赖管理。

Composer 是从从包的来源直接安装，而不是简单的下载 zip 文件。

## 发布
登录`https://packagist.org/`，使用GitHub账号登录，授权token等，发布一个包含正常规则的`composer.json`的类包。通过tag打版本。

## 私有包
在项目中一些私有包可以使用`repositories`指定仓库地址
```json
{
    "name" : "server-demo",
    "description": "server demo",
    "repositories": [
        {
            "type": "vcs",
            "url": "git@code.aliyun.com:ruesin/server.git"
        },
        {
            "type": "git",
            "url": "git@code.aliyun.com:ruesin/core.git"
        }
    ],
    "require": {
        "ruesin/server": "dev-master",
        "ruesin/core": "1.1.0",
        "aliyuncs/oss-sdk-php": "2.3.0"
    },
    "require-dev" :{
    },
    "autoload": {
        "psr-4": {
            "App\\": "app/"
        },
        "files": [
            "app/helpers.php"
        ]
    },
    "minimum-stability": "stable",
    "scripts" : {
    }
}

```

## composer.json

```json
{
    "#name" : "包的名称：`供应商名称/项目名称`，如果发布此包（库），必填",
    "name" : "ruesin/demo",

    "#description" : "包的描述，如果发布此包（库），必填",
    
    "#keywords" : "该包相关的关键词的数组，可用于搜索和过滤。",
    "keywords": ["ruesin","server"],

    "#homepage" : "主页地址",
    "#version" : "因为Composer可以从git推断版本号，不是必须的，建议忽略",
    "#time" : "版本发布时间，必须符合 YYYY-MM-DD 或 YYYY-MM-DD HH:MM:SS 格式",
    
    "#type" : "包的安装类型，用来定义安装逻辑，默认为 library。
    如果这个包需要一个特殊的逻辑，你以设定一个自定义的类型。比如：symfony-bundle、wordpress-plugin、typo3-module。这些类型都将是具体到某一个项目，而对应的项目将要提供一种能够安装该类型包的安装程序。
    library: 简单的将文件复制到 vendor 目录
    project: 表示当前包是一个项目，而不是一个库。使用 IDE 创建一个新的工作区时，这可以为其提供项目列表的初始化。
    metapackage: 当一个空的包，包含依赖并且需要触发依赖的安装，这将不会对系统写入额外的文件。因此这种安装类型并不需要一个 dist 或 source。
    composer-plugin: 一个安装类型为 composer-plugin 的包，它有一个自定义安装类型，可以为其它包提供一个 installler。详细请查看 自定义安装类型。
    ",
    "type": "project",
    
    "#license" : "开源协议",
    "license": "MIT",
    "#authors" : "作者",
    "authors": [
        {
            "name": "Ruesin Liu",
            "email": "ruesin@gmail.com",
            "homepage": "https://www.xinkoukaihe.com",
            "role": "Developer"
        }
    ],

    "#require" : "依赖，包名:版本号",
    "require": {
        "php": ">=7.0",
        "ext-json": "*",
        "ruesin/swover": "^1.1",
        "predis/predis": "1.1.1"
    },
    "#require-dev" : "为开发或测试等目的额外列出的依赖。“root 包”的 require-dev 默认是会被安装的。install 或 update 使用 --no-dev 跳过 require-dev",
    "#conflict" : "列表中的包与当前包的这个版本冲突。它们将不允许同时被安装",
    "#replace" : "列表中的包将被当前包取代",
    "suggest" : "建议安装的包，它们增强或能够与当前包良好的工作。这些只是信息，并显示在依赖包安装完成之后，给你的用户一个建议，他们可以添加更多的包。",
    
    "#autoload" : "PHP autoloader 的自动加载映射。支持PSR-0、PSR-4、classmap、files。",
    "autoload": {
        "psr-4": {
            "App\\": "app/"
        },
        "classmap": ["src/", "lib/", "Something.php"],
        "files": [
            "common/helpers.php"
        ]
    },
    "autoload-dev": {
        "psr-4": { "MyLibrary\\Tests\\": "tests/" }
    },

    "#minimum-stability" : "定义了通过稳定性过滤包的默认行为。默认为 stable（稳定）。如果依赖于一个 dev（开发）包，应该明确的进行定义。
对每个包的所有版本都会进行稳定性检查，而低于 minimum-stability 所设定的最低稳定性的版本，将在解决依赖关系时被忽略。对于个别包的特殊稳定性要求，可以在 require 或 require-dev 中设定。
可用的稳定性标识（按字母排序）：dev、alpha、beta、RC、stable。",

    "#prefer-stable" : "Composer 将优先使用更稳定的包版本",
    "prefer-stable": true,

    "#repositories" : "默认情况下 composer 只使用 packagist 作为包的资源库。通过指定资源库，使用自定义的包资源库。
Repositories 并不是递归调用的，只能在“Root包”的 composer.json 中定义。附属包中的 composer.json 将被忽略。
composer: 一个 composer 类型的资源库，是一个简单的网络服务器（HTTP、FTP、SSH）上的 packages.json 文件，它包含一个 composer.json 对象的列表，有额外的 dist 和/或 source 信息。这个 packages.json 文件是用一个 PHP 流加载的。你可以使用 options 参数来设定额外的流信息。
vcs: 从 git、svn 和 hg 取得资源。
pear: 从 pear 获取资源。
package: 如果你依赖于一个项目，它不提供任何对 composer 的支持，你就可以使用这种类型。你基本上就只需要内联一个 composer.json 对象。
顺序是非常重要的，当 Composer 查找资源包时，它会按照顺序进行。默认情况下 Packagist 是最后加入的，因此自定义设置将可以覆盖 Packagist 上的包。
",
"repositories": [
        {
            "type": "composer",
            "url": "http://packages.example.com"
        },
        {
            "type": "composer",
            "url": "https://packages.example.com",
            "options": {
                "ssl": {
                    "verify_peer": "true"
                }
            }
        },
        {
            "type": "vcs",
            "url": "https://github.com/Seldaek/monolog"
        },
        {
            "type": "pear",
            "url": "http://pear2.php.net"
        },
        {
            "type": "package",
            "package": {
                "name": "smarty/smarty",
                "version": "3.1.7",
                "dist": {
                    "url": "http://www.smarty.net/files/Smarty-3.1.7.zip",
                    "type": "zip"
                },
                "source": {
                    "url": "http://smarty-php.googlecode.com/svn/",
                    "type": "svn",
                    "reference": "tags/Smarty_3_1_7/distribution/"
                }
            }
        }
    ],

    "#config" : "应用于本项目的配置项，如vendor、cache目录等",

    "#scripts" : "在安装过程中的各个阶段挂接脚本，如在 create-project 时执行的 post-root-package-install",
    "scripts": {
        "post-root-package-install": [
            "@php -r \"copy('./config/samples/queue.php', './config/queue.php');\"",
        ]
    },
    "#extra" : "任意的，供 scripts 使用的额外数据",
    "#bin" : "用于标注一组应被视为二进制脚本的文件，他们会被软链接到（config 对象中的）bin-dir 属性所标注的目录，以供其他依赖包调用",
    "#archive" : "在创建包存档时使用。exclude: 允许设置一个需要被排除的路径的列表。与 .gitignore 语法相同。一个前导的（!）将会使其变成白名单而无视之前相同目录的排除设定。前导斜杠只会在项目的相对路径的开头匹配。星号为通配符。",
    "archive": {
        "exclude": ["/foo/bar", "baz", "/*.test", "!/foo/bar/baz"]
    }
}
```


## 命令

### 初始化 init 
可以手动创建composer.json，也可以通过`composer init`初始化项目，配置composer.json。

### 安装 install
从当前目录读取 composer.json 文件，处理依赖关系，并把其安装到 vendor 目录下。
如果当前目录下存在 composer.lock 文件，会从此文件读取依赖版本，而不是根据 composer.json 文件去获取依赖。确保该库的每个使用者都能得到相同的依赖版本。

如果没有 composer.lock 文件，composer 将在处理完依赖关系后创建它。

- --prefer-source: 下载包的方式有两种： source 和 dist。对于稳定版本 composer 将默认使用 dist 方式。而 source 表示版本控制源 。如果 --prefer-source 是被启用的，composer 将从 source 安装（如果有的话）。如果想要使用一个 bugfix 到你的项目，这是非常有用的。并且可以直接从本地的版本库直接获取依赖关系。
- --prefer-dist: 与 --prefer-source 相反，composer 将尽可能的从 dist 获取，这将大幅度的加快在 build servers 上的安装。这也是一个回避 git 问题的途径，如果你不清楚如何正确的设置。
- --dry-run: 如果你只是想演示而并非实际安装一个包，你可以运行 --dry-run 命令，它将模拟安装并显示将会发生什么。
- --dev: 安装 require-dev 字段中列出的包（这是一个默认值）。
- --no-dev: 跳过 require-dev 字段中列出的包。
- --no-scripts: 跳过 composer.json 文件中定义的脚本。
- --no-plugins: 关闭 plugins。
- --no-progress: 移除进度信息，这可以避免一些不处理换行的终端或脚本出现混乱的显示。
- --optimize-autoloader (-o): 转换 PSR-0/4 autoloading 到 classmap 可以获得更快的加载支持。特别是在生产环境下建议这么做，但由于运行需要一些时间，因此并没有作为默认值。

### 更新 update

获取依赖的最新版本，升级 composer.lock 文件，解决项目的所有依赖，并将确切的版本号写入 composer.lock。

```
// 全部更新
composer update
// 指定更新
composer update vendor/package vendor/package2
// 通配符更新
composer update vendor/*
```
除了支持`install`的所有参数外，还有两个参数：
- --lock: 仅更新 lock 文件的 hash，取消有关 lock 文件过时的警告。
- --with-dependencies 同时更新白名单内包的依赖关系，这将进行递归更新。

### 声明依赖 require
增加新的依赖包到当前目录的 composer.json 文件中，在添加或改变依赖时， 修改后的依赖关系将被安装或者更新。

```
// 命令行交互声明
composer require
// 指定声明
composer require swover/swover ruesin/utils:dev-master
```
- --prefer-source: 当有可用的包时，从 source 安装。
- --prefer-dist: 当有可用的包时，从 dist 安装。
- --dev: 安装 require-dev 字段中列出的包。
- --no-update: 禁用依赖关系的自动更新。
- --no-progress: 移除进度信息，这可以避免一些不处理换行的终端或脚本出现混乱的显示。
- --update-with-dependencies 一并更新新装包的依赖。

### 全局执行 global
允许在 `COMPOSER_HOME` 目录下执行其它命令，比如 install、require 或 update。

如果将 `$COMPOSER_HOME/vendor/bin` 加入到了 $PATH 环境变量中，就可以用它在命令行中安装全局应用。

### 搜索 search
搜索依赖包，通常只搜索 packagist.org 上的包，`composer search monolog`也可以通过传递多个参数来进行多条件搜索，参数`--only-name (-N)`指定的名称搜索（完全匹配）。

### 展示 show
```
// 列出所有可用的软件包
composer show
// 查看一个包的详细信息
composer show monolog/monolog
// 查看一个包指定版本的详细信息
composer show monolog/monolog 1.0.2
```
- --installed (-i): 列出已安装的依赖包。
- --platform (-p): 仅列出平台软件包（PHP 与它的扩展）。
- --self (-s): 仅列出当前项目信息。

### 依赖性检测 depends
查出已安装在项目中的某个包，是否正在被其它的包所依赖，并列出他们。
```
composer depends --link-type=require monolog/monolog
```
--link-type: 检测的类型，默认为 require 也可以是 require-dev。

### 有效性检测 validate
在提交`composer.json`文件，和创建`tag`前，应该检测`composer.json`文件是否有效。参数`--no-check-all`是否进行完整的校验。

### 依赖包状态检测 status
如果依赖包是从source（自定义源）安装的，修改依赖包的代码后，可以使用此命令检查本地做了哪些更改。

### 自我更新 self-update
```
//更新 Composer到最新版本。
composer self-update
//更新到指定版本
composer self-update 1.0.0-alpha7
```
- --rollback (-r): 回滚到你已经安装的最后一个版本。
- --clean-backups: 在更新过程中删除旧的备份，这使得更新过后的当前版本是唯一可用的备份。

### 更改配置 config
可以使用`config`命令编辑`Composer`的一些基本设置，无论是本地的`composer.json`或者全局的`config.json`文件。


```
// 查看配置列表
composer config --list
// 命令使用
composer config [options] [setting-key] [setting-value1] ... [setting-valueN]

//修改包来源
composer config repositories.foo vcs http://github.com/foo/bar
```

- --global (-g): 操作位于 $COMPOSER_HOME/config.json 的全局配置文件。如果不指定该参数，此命令将影响当前项目的 composer.json 文件，或 --file 参数所指向的文件。
- --editor (-e): 使用文本编辑器打开 composer.json 文件。默认情况下始终是打开当前项目的文件。当存在 --global 参数时，将会打开全局 composer.json 文件。
- --unset: 移除由 setting-key 指定名称的配置选项。
- --list (-l): 显示当前配置选项的列表。当存在 --global 参数时，将会显示全局配置选项的列表。
- --file="..." (-f): 在一个指定的文件上操作，而不是 composer.json。注意：不能与 --global 参数一起使用。

### 创建项目 create-project
从现有的包中创建一个新的项目，相当于执行了`git clone`后，再将这个包的依赖安装。

场景：
- 快速的部署应用
- 检出资源包，并开发它的补丁
- 多人开发项目，可以用它来加快应用的初始化
- 创建基于`Composer`的新项目，使用 "create-project" 命令。传递一个包名，它会为你创建项目的目录。第三个参数为版本号，否则将获取最新的版本

```
//创建基于doctrine/orm:2.2.* 的项目到 path 目录
composer create-project doctrine/orm path 2.2.*
```

- --repository-url: 提供一个自定义的储存库来搜索包，这将被用来代替 packagist.org。可以是一个指向 composer 资源库的 HTTP URL，或者是指向某个 packages.json 文件的本地路径。
- --stability (-s): 资源包的最低稳定版本，默认为 stable。
- --prefer-source: 当有可用的包时，从 source 安装。
- --prefer-dist: 当有可用的包时，从 dist 安装。
- --dev: 安装 require-dev 字段中列出的包。
- --no-install: 禁止安装包的依赖。
- --no-plugins: 禁用 plugins。
- --no-scripts: 禁止在根资源包中定义的脚本执行。
- --no-progress: 移除进度信息，这可以避免一些不处理换行的终端或脚本出现混乱的显示。
- --keep-vcs: 创建时跳过缺失的 VCS 。如果你在非交互模式下运行创建命令，这将是非常有用的。

### 打印自动加载索引 dump-autoload
更新 autoloader。可以打印一个优化过的、符合`PSR-0/4`规范的类的索引，这也是出于对性能的可考虑。在大型的应用中会有许多类文件，而`autoloader`会占用每个请求的很大一部分时间，使用`classmaps`或许在开发时不太方便，但它在保证性能的前提下，仍然可以获得`PSR-0/4`规范带来的便利。

- --optimize (-o): 转换 PSR-0/4 autoloading 到 classmap 获得更快的载入速度。这特别适用于生产环境，但可能需要一些时间来运行，因此它目前不是默认设置。
- --no-dev: 禁用 autoload-dev 规则。

### 查看许可协议 licenses
列出已安装的每个包的名称、版本、许可协议。`--format=json` 获取 JSON 格式的输出。

### 执行脚本 run-script
手动执行脚本，只需要指定脚本的名称，`--no-dev`禁用开发者模式。

### 诊断 diagnose

### 归档 archive
对指定包的指定版本进行`zip/tar`归档。也可以用来归档整个项目，不包括 excluded/ignored（排除/忽略）的文件。
```
composer archive vendor/package 2.0.21 --format=zip
```
- --format (-f): 指定归档格式：tar 或 zip（默认为 tar）。
- --dir: 指定归档存放的目录（默认为当前目录）。

### 获取帮助信息 help
使用 help 可以获取指定命令的帮助信息，比如`composer help install`

### 环境变量
//TODO