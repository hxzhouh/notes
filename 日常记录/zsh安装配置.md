

**首先设置 git wget 代理**

```bash
git config --global https.proxy http://127.0.0.1:1080
git config --global https.proxy https://127.0.0.1:1080
git config --global --unset http.proxy
git config --global --unset https.proxy
```

http 代理

```bash
export https_proxy=118.210.42.251:44367
export http_proxy=118.210.42.251:44367
unset http_proxy
unset http_proxy
```



一些命令 apt-get install zsh

切换默认shell解释器

``` shell
chsh -s /bin/zsh
```





在 命令后面显示时间

编辑 ~/.zshrc 添加一行

``` shell
RPROMPT="%{$fg[yellow]%}[%D{%f/%m/%y}|%@]"
```



**出现一个问题**

```bash
[oh-my-zsh] Insecure completion-dependent directories detected:
drwxrwxrwx 1 yixiao yixiao 512 Apr  8 19:08 /home/yixiao/.oh-my-zsh/custom/plugins/zsh-autosuggestions

[oh-my-zsh] For safety, we will not load completions from these directories until
[oh-my-zsh] you fix their permissions and ownership and restart zsh.
[oh-my-zsh] See the above list for directories with group or other writability.

[oh-my-zsh] To fix your permissions you can do so by disabling
[oh-my-zsh] the write permission of "group" and "others" and making sure that the
[oh-my-zsh] owner of these directories is either root or your current user.
[oh-my-zsh] The following command may help:
[oh-my-zsh]     compaudit | xargs chmod g-w,o-w

[oh-my-zsh] If the above didn't help or you want to skip the verification of
[oh-my-zsh] insecure directories you can set the variable ZSH_DISABLE_COMPFIX to
[oh-my-zsh] "true" before oh-my-zsh is sourced in your zshrc file.

```

因为 插件 `zsh-autosuggestions` 权限过大导致

执行命令以下命令解决

``` shell
chmod 755  /home/yixiao/.oh-my-zsh/custom/plugins/zsh-autosuggestions 
```

**默认使用root登录**

在 windows10 powershell 中使用以下命令解决

```
ubuntu config --default-user root
```