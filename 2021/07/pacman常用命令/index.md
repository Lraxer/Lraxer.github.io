# Pacman常用命令


<!--more-->

## pacman简介

pacman是Arch Linux下的软件包管理器。与Ubuntu等系统下常用的apt，apt-get不同，pacman有一套比较特别的命令，下面对常用的命令做一个总结。

## 命令格式

pacman的命令格式非常统一，都是

```bash
sudo pacman <operation> [options] [targets]
```

`<operation>` 代表一个主要动作，包括有 `DQRSTUFVh` 这几个。对于这些动作，还有一些 `[options]` 参数，可以应用于全部或者部分动作。后面列出一些常用的操作和一些可选参数，没有提及那些容易使用出错的。

## 常用命令

### 选取中国地区合适的源

```bash
sudo pacman-mirrors -i -c China -m rank
```

### 更新软件和系统

```bash
sudo pacman -Syu
sudo pacman -Syu <package> # 更新package list, 更新所有软件包, 然后安装软件包<package>
```

### 安装软件包

```bash
sudo pacman -S <pkg> # 从同步数据库安装

sudo pacman -U /home/user/ceofhack-0.6-1-x86_64.pkg.tar.gz # 从本地路径安装软件包
sudo pacman -U http://www.example.com/repo/example.pkg.tar.xz # 从URL安装软件包
```

### 删除软件包

```bash
sudo pacman -Rsc <package> # 删除<package>、所有依赖它的软件包、仅有它依赖的软件包
```

### 搜索软件包

```bash
pacman -Ss <package> # 在同步数据库中搜索符合正则表达式的软件包
pacman -Qs <package> # 搜索已经安装的软件包
```

### 清理软件包

```bash
sudo pacman -Sc # 清理未安装的包缓存
sudo pacman -Qtdq | sudo pacman -Rs - # 删除孤儿包
```

## Operations

| operations       | 描述                                                         |
| :--------------- | ------------------------------------------------------------ |
| `-D, --database` | 对pacman数据库进行修改。需要配合其他参数。                   |
| `-Q, --query`    | 查询**本地软件包数据库**。可以查单个包也可以查所有包（缺省包名时）的信息。 |
| `-R, --remove`   | 删除包（或者是group）。指定--nosave可以同时去除保存为.packsave的配置文件。（这里存疑，没明白数据库里这些文件的意义） |
| `-S, --sync`     | 从**同步数据库**安装包和所有依赖。可以这样指定版本：`pacman -S "bash>=3.2"`（必须有双引号）还有很多 `[options]` 可以与这个动作配合使用 |
| `-T, --deptest`  | 依赖检查。返回一个现在系统尚未满足的依赖列表。 `-T` 没有其他子选项。 |
| `-U, --upgrade`  | 通过URL或者本地路径安装包，而不是从pacman配置的源里。执行过程为"remove-then-add"。 |
| `-F, --files`    | 查询文件数据库。查看一个软件包是否包含特定文件，或者显示包含特定文件的软件包。 |
| `-V, --version`  | 显示版本。                                                   |
| `-h, --help`     | 查看帮助。例如：`pacman -Q --help` 查看 `-Q` 动作的子选项    |

## Options

### 通用

| Options                      | 描述                                                         |
| ---------------------------- | ------------------------------------------------------------ |
| `-v, --verbose`              | 输出pacman的配置信息。（文件目录等）                         |
| `--debug`                    | 输出debug messages。                                         |
| `--nocomfirm`                | 跳过所有"Are you sure?"。**只在使用脚本的时候建议这么做。需要手动开启关闭。** |
| --confirm                    | 取消--nocomfirm。                                            |
| `--disable-download-timeout` | 关闭下载的timeout。                                          |

### Transaction Options (-S, -R and -U)

| Options                   | 描述                                                         |
| ------------------------- | ------------------------------------------------------------ |
| `--noprogressbar`         | 不显示下载进度条。**适合编写脚本。**                         |
| `-p, --print`             | Only print the targets instead of performing the actual operation (sync, remove or upgrade). Use --print-format to specify how targets are displayed. The default format string is "%l", which displays URLs with -S, file names with -U, and pkgname-pkgver with -R. |
| `--print-format <format>` | Specify a printf-like format to control the output of the --print operation. The possible attributes are: "%n" for pkgname, "%v" for pkgver, "%l" for location, "%r" for repository, and "%s" for size. Implies --print. |

### Upgrade Options (-S, -U)

| Options                 | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `--asdeps`              | 非显式地安装软件包，将其看作一个依赖关系安装。               |
| `--asexplicit`          | 显式安装软件包。将一个软件包显式安装，就不会被删除时的 `--recursive` 删除。 |
| `--ignore <package>`    | 忽略更新个别软件包。多个包以逗号分隔。                       |
| `--ignoregroup <group>` | 忽略更新group。多个groups以逗号分隔。                        |
| `--needed`              | 不要重新安装已经更新的软件包。                               |

### Query Options (-Q)

| Options                 | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `-c, --changelog`       | 查看软件包的ChangeLog（如果有的话）                          |
| `-d, --deps`            | 限制查询范围，只过滤**作为依赖安装的软件包**。配合 `-t` 选项使用可以过滤不被任何已安装的软件包所依赖的包。 |
| `-e, --explicit`        | 限制查询范围，只过滤**显式安装的软件包**。配合 `-t` 选项使用可以过滤不被其他软件包要求安装的包。 |
| `-g, --groups`          | 列出一个group中的所有软件包。如果没有指明名称，就列出所有groups的软件包。 |
| `-i, --info`            | 列出给定软件包的信息。                                       |
| `-k, --check`           | 检查系统中是否有指定软件包的全部文件。                       |
| `-l, --list`            | 列出给定软件包的全部文件。可以给定多个软件包。               |
| `-m, --foreign`         | 只过滤**同步数据库找不到的软件包**。一般都是使用 `-U` 手动安装的软件包。 |
| `-n, --native`          | 只过滤**同步数据库能找到的软件包**。与 `-m` 相反。           |
| `-o, --owns <file>`     | 查询带有特定文件的软件包。文件可以指定多个。                 |
| `-p, --file`            | Signifies that the package supplied on the command line is a file and not an entry in the database. The file will be decompressed and queried. This is useful in combination with *--info* and *--list*. **不知道能做什么，应该是可以配合 `-i` 之类的参数查询package file的信息。** |
| `-q, --quiet`           | 显示更少的查询结果信息。**适合脚本编程使用。**               |
| `-s, --search <regexp>` | 搜索名称或者描述符合正则表达式的**本地已安装软件包**。有多个搜索条件时，返回全都匹配的软件包。 |
| `-t, --unrequired`      | 过滤**不被已安装的所有软件包所需要的包(directly or optionally)**。指定这个选项两次，可以过滤可选的包。(optionally) |
| `-u, --upgrades`        | 过滤**当前系统上过时的软件包**                               |

### Remove Options (-R)

| Options           | 描述                                                         |
| ----------------- | ------------------------------------------------------------ |
| `-c, --cascade`   | 删除目标软件包，并**递归删除所有依赖这些包的软件包**。       |
| `-n, --nosave`    | 忽略配置文件的备份名称。(Instructs pacman to ignore file backup designations.) |
| `-s, --recursive` | 删除目标软件包和它的依赖项。被删除的依赖项满足：（1）它们不被其他软件包依赖；（2）不是被用户显式安装的。想要移除第二个条件，传递两次这个参数。 |
| `-u, --unneeded`  | 移除目标软件包，以及不被其他软件包所需要的包。               |

### Sync Options (-S)

| Options                 | 描述                                                         |
| ----------------------- | ------------------------------------------------------------ |
| `-c, --clean`           | 从缓存中删除没有安装的软件包和未使用的同步数据库，以释放空间。 |
| `-g, --groups`          | 显示指定group的所有软件包成员。如果没有提供group name，列出所有groups；传递两次这个参数查看所有groups和它们的成员。 |
| `-i, --info`            | 显示一个给定的同步数据库软件包信息。 Passing two *--info* or *-i* flags will also display those packages in all repositories that depend on this package. |
| `-l, --list`            | 列出指定仓库的所有软件包。可以给定多个仓库。                 |
| `-q, --quiet`           | 显示较少的信息。**适合脚本编程。**                           |
| `-s, --search <regexp>` | 在同步数据库中搜索匹配的软件包。                             |
| `-u, --sysupgrade`      | 升级所有过期软件包。传递这个参数两次可以启用降级。也可以手动指定额外的升级/安装目标。 |
| `-y, --refresh`         | 从 `pacman.conf` 服务器设置的服务器下载数据库副本。传递这个参数两次可以强制更新。这个参数应当和 `-u` 一起使用，否则可能会有依赖问题。 |

### Database Options (-D)

| Options                  | 描述                                                         |
| ------------------------ | ------------------------------------------------------------ |
| `--asdeps <package>`     | 将一个软件包标记为非显式安装。（即作为依赖安装）             |
| `--asexplicit <package>` | 将一个软件包标记为显式安装。                                 |
| `-k, --check`            | 检查本地软件包数据库的内部一致性(internally consistent)。(This will check all required files are present and that installed packages have the required dependencies, do not conflict and that multiple packages do not own the same file. )传递这个参数两次将检查同步数据库。 |
| `-q, --quiet`            | 不显示成功完成的数据库操作。                                 |

### File Options (-F)

| Options             | 描述                                                         |
| ------------------- | ------------------------------------------------------------ |
| `-y, --refresh`     | 从服务器下载最新的软件包数据库。传递这个参数两次可以强制更新。 |
| `-l, --list`        | 列出被查询软件包拥有的文件。                                 |
| `-x, --regex`       | 将查询解释为正则表达式。                                     |
| `-q, --quiet`       | 显示更少的消息。                                             |
| `--machinereadable` | 显示更适合机器读取的消息。The format is *repository\0pkgname\0pkgver\0path\n* with *\0* being the NULL character and *\n* a linefeed. |


