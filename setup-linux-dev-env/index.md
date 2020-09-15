# Linux环境配置备忘录


一些配置备忘录

<!--more-->



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
echo "d=docker" >> ~/.zshrc
echo "k=kubectl" >> ~/.zshrc
source ~/.zshrc
```


