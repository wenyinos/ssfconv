# 简介
fcitx输入法框架能够自定义皮肤，然后有个很nb的作者开发了个搜狗皮肤转换成fcitx皮肤的，这是原项目地址 https://github.com/VOID001/ssf2fcitx

然后我亲自试了几个我喜欢的皮肤，居然真的可以转换，跟搜狗差不多了，不过一段时间后，发现一些bug：设置了皮肤之后，输入法菜单隔空而且透明，字都看不清。部分皮肤文字位置很奇怪。于是，我看了他的源码，发现逻辑还挺简单，然后看了下fcitx的自定义皮肤的各种格式，打算亲自研究研究这是怎么回事。

最终打算参考这个项目，自己用python写个。

由于 fcitx5 也支持主题，最终也实现了转换成 fcitx5 主题！

## 成果

在原作者的基础上进行了下面几方面改进：

1. 部分皮肤文字位置重新计算，摆放更合理
2. 将菜单的背景也设置成皮肤的主题色，文字大小和颜色均计算到合理
3. 字体单位改成像素，和搜狗一致，完美还原
4. 调整了翻页指示器的位置，自动生成指示器的图像
5. 额外支持 fcitx5


## 开始使用

下面直接举例吧。



对于其它发行版下，请按照下面方法逐步安装。

### 下载此仓库

```shell
git clone https://github.com/fkxxyz/ssfconv.git
cd ssfconv
```

### 安装python依赖

该项目使用 python3 开发，依赖于 pycryptodome、pillow、numpy 库，最好使用相应的发行版的包管理器安装它们，或者使用 pip：

```shell
pip install -r requirements.txt
```

### 下载皮肤

先从[搜狗输入法的皮肤官网](https://pinyin.sogou.com/skins/)下载自己喜欢的皮肤，得到ssf格式的文件，放入 `sougou/` 目录。

### 快速使用（一键转换+安装）

将 `sougou/` 目录下的所有 .ssf 文件转换为 fcitx5 主题 zip 包，并自动安装：

```shell
./ssfconv -i
```

输出文件位于 `output/` 目录，同时自动解压安装到 `~/.local/share/fcitx5/themes/`。

### 命令行参数

```
ssfconv [src] [dest] [-t type] [-i] [--no-zip]
```

| 参数 | 说明 |
|------|------|
| `src` | 源文件路径，不指定则自动从 `sougou/` 目录读取所有 .ssf 文件 |
| `dest` | 输出路径，不指定则自动输出到 `output/<皮肤名>_<格式>.zip` |
| `-t` / `--type` | 输出格式：`fcitx5`（默认）、`fcitx`、`dir` |
| `-i` / `--install` | 自动安装到 `~/.local/share/fcitx5/themes/`（仅 fcitx5） |
| `--no-zip` | 输出为目录而非 zip 压缩包（默认打包为 zip） |

### 使用示例

批量转换并安装（最常用）：

```shell
./ssfconv -i
```

转换单个文件并安装：

```shell
./ssfconv -i 花未央.ssf
```

仅转换为 fcitx5 zip 包（不安装）：

```shell
./ssfconv 花未央.ssf
```

输出为目录而非 zip：

```shell
./ssfconv --no-zip 花未央.ssf
```

转换为 fcitx 格式：

```shell
./ssfconv -t fcitx 花未央.ssf
```

### 启用 fcitx5 主题

打开 fcitx5 的配置，附加组件标签，经典用户界面，点配置，在主题的下拉列表里，选择这款皮肤。

或者直接修改配置文件 `~/.config/fcitx5/conf/classicui.conf`，将 `Theme` 的值改成皮肤的名称即可。

查看已安装主题的名称：

```shell
grep Name ~/.local/share/fcitx5/themes/<主题目录>/theme.conf
```

## 详细介绍

使用方法被封装得非常简单，像个转换器，可以在下面格式之间任意转换：

1. ssf格式（加密）
2. ssf格式（未加密，本质是zip）
3. 文件夹（解密或解压ssf格式得到）
4. fcitx格式（在文件夹的基础上多了fcitx_skin.conf，可用于fcitx）
5. fcitx5格式（在文件夹的基础上多了theme.conf，可用于fcitx5）

注意，源文件的格式可以是以上任意格式之一，不需要指定，程序已经可以智能识别格式。

默认输出为 fcitx5 格式的 zip 压缩包（`*_fcitx5.zip`）。

支持的输出类型（`-t` 参数）：

```
fcitx5          可直接用于fcitx5的文件夹（默认）
fcitx           可直接用于fcitx的文件夹
dir             解包后的文件夹
encrypted       加密的ssf皮肤（暂未实现）
zip             未加密的ssf皮肤（zip）
```

## 已知缺陷

### fcitx

因为 fcitx 的限制，输入框里只能对文字的外边距进行设置，无法像搜狗拼音输入法一样任意调整坐标，导致部分皮肤只能在图片拉升和文件位置靠右来二选一的取舍。不过大多数皮肤都能挺不错的转换，只有少数皮肤实在是没办法了，只好用图片拉升代替（原作者是将文字调整到靠右，留了很多空白）。

### fcitx5

- fcitx5 能够完美地像搜狗输入法一样调整，但是主题中所设置的字体是无效的，需要手动设置字体，经过我反复的实验，将字体设置为 "Sans 10" 似乎是大多数皮肤的最佳体验。
- 菜单字体颜色无法通过主题调整，只能为黑色高亮白色，所以在背景比较黑或者比较白的皮肤下，菜单可能体验不理想。
- 部分皮肤可能转换效果不太好，需要寻找原因，欢迎提出 issues 帮助我改进，最好说明皮肤的下载链接便于排查。

## 致谢

该项目的思路，以及解密的过程和密钥，完全参考了 [VOID001/ssf2fcitx](VOID001/ssf2fcitx) 在此表示感谢！

