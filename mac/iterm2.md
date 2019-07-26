### iterm2使用

---

#### iterm2 隐藏主机名称 显示用户名 

`vim ~/.oh-my-zsh/themes/agnoster.zsh-theme`

```powershell
prompt_context() {
  if [[ "$USER" != "$DEFAULT_USER" || -n "$SSH_CLIENT" ]]; then
    prompt_segment black default "%(!.%{%F{yellow}%}.)$USER"
  fi
}
```

wq 保存退出后  `source ~/.zshrc`

#### iterm2 远程连接

获取 sshpass `brew install sshpass`

在iterm2中的设置中的profiles中配置相应的远程连接

1. 新建连接
2. Command中新添命令 `/usr/local/bin/sshpass -p swallow@ying92? ssh root@39.97.109.173`
3. 第一次需要先 ssh 手动连接一下 这个比较坑