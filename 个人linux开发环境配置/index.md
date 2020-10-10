# 个人Linux开发环境配置


一些配置备忘录

<!--more-->



## 安装

```bash
apk update
apk install git vim zsh
```



## zsh

```bash
# 使用gitee的zsh镜像安装
sh -c "$(wget -O- https://gitee.com/mirrors/oh-my-zsh/raw/master/tools/install.sh)"
sed -i 's/ZSH_THEME=.*/ZSH_THEME="fishy"/g' ~/.zshrc

# zsh plugins
# 1. zsh-autosuggestions
git clone --depth=1 https://gitclone.com/github.com/zsh-users/zsh-autosuggestions ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions
# 2. zsh-syntax-highlighting
git clone --depth=1 https://gitclone.com/github.com/zsh-users/zsh-syntax-highlighting.git ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
sed -i 's/plugins=.*/plugins=(git zsh-autosuggestions zsh-syntax-highlighting)/g' ~/.zshrc
source ~/.zshrc
```



## vim

```bash
git clone --depth=1 https://gitclone.com/github.com/amix/vimrc.git ~/.vim_runtime
sh ~/.vim_runtime/install_awesome_vimrc.sh
echo "set number" >> ~/.vim_runtime/my_configs.vim
```



## alias

```bash
echo "alias d=docker" >> ~/.zshrc
echo "alias k=kubectl" >> ~/.zshrc
source ~/.zshrc
```



## dokcer

```bash
echo "http://mirrors.ustc.edu.cn/alpine/v3.12/community" >> /etc/apk/repositories
apk update
apk add docker
```



## rancher

```bash
mount --make-rshared /
```


