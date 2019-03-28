# zsh

## [zsh-autosuggestions](https://github.com/zsh-users/zsh-autosuggestions)（命令自动补全）

``` sh
# Oh My Zsh-安装方式
git clone https://github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
# 修改 ~/.zshrc，添加插件
plugins=(git zsh-autosuggestions)
```

## [powerline-status](https://powerline.readthedocs.io/en/latest/installation/osx.html) （不用安装）

```sh
# 先安装python，安装的是python3（mac系统安装的是python2，Xcode command line tool）
brew install python
# 使用pip3安装（Python3自带pip3，mac系统的python2没有pip，只有easy_install，可以使用锁sudo easy_install pip安装pip）
pip3 install --user powerline-status
# 查看
pip3 show powerline-status
# 根据上面查看的powerline-status所在目录，在~/.zshrc添加配置，{repository_root}改为所在目录
. {repository_root}/powerline/bindings/zsh/powerline.zsh
```

> 我觉得没必要安装这个，安装下面的字体就可以了。其他的主题都是需要字体，感觉和这个没关系吧。
>
> 上面配置powerline.zsh后还有错误，找不到powerline-config，算了不安装这个了，就当了解下pip3和pip吧。

## [Powerline Fonts](https://github.com/powerline/fonts) (字体)

``` sh
# clone
git clone https://github.com/powerline/fonts.git --depth=1
# install
cd fonts
./install.sh
# clean-up a bit
cd ..
rm -rf fonts
```

> 最后在 Hyper 终端的配置文件 `.hyper.js` 中修改字体，将 `fontFamily = '...'` 一项引号内添加"Meslo LG M for Powerline"或者其他 Powerline 字体，重启终端便可生效。

## [Spaceship Prompt](https://github.com/denysdovhan/spaceship-prompt)（主题）

``` sh
# Oh My Zsh 安装方式
git clone https://github.com/denysdovhan/spaceship-prompt.git "$ZSH_CUSTOM/themes/spaceship-prompt"
# 添加链接
ln -s "$ZSH_CUSTOM/themes/spaceship-prompt/spaceship.zsh-theme" "$ZSH_CUSTOM/themes/spaceship.zsh-theme"
# 修改 ~/.zshrc，修改主题
ZSH_THEME="spaceship"
```

## [Fira Code](https://github.com/tonsky/FiraCode)（为写程序而生的字体）

> 下载字体文件，解压，打开ttf文件夹，全选所有文件，右键选择使用字体册打开，然后点击安装



# 快捷复制文件目录

1. 显示里开启路径栏，右键路径栏上路径，选择拷贝路径。(推荐，省事)

![image-20190228105505458](../images/image-20190228105505458.png)

![image-20190228105657894](../images/image-20190228105657894.png)

2. 使用自动操作，创建拷贝路径服务，再设置个快捷键

![image-20190228111852247](../images/image-20190228111852247.png)

![image-20190228111430672](../images/image-20190228111430672.png)

![image-20190228112148593](../images/image-20190228112148593.png)

![image-20190228112224491](../images/image-20190228112224491.png)