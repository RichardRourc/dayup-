Node Version Switcher(NVS)
windows上也可以使用的node版本管理工具

用这个工具的原因，公司电脑没管理员权限等原因，无法安装成功著名node版本管理工具的nvm，但是接手的几个项目是老项目，需要方便切换node版本，经过在网上的一番查询，发现了NVS这个可以在公司电脑使用来做node版本管理的工具

##### 安装
[安装地址，git](https://github.com/jasongin/nvs#setup)

#### 使用
```js

# 查看当前使用的nvs版本
nvs --version
# 或者
nvs -v

# 列出所有可用的nvs版本
# 可供下载
nvs ls-remote
# 本地已下载
nvs ls

# 下载指定版本，如 nvs add 16、nvs add 16.14、nvs add lts
nvs add <version>

# 删除指定版本，如 nvs rm 16、nvs rm 16.14、nvs rm lts
nvs rm <version>

# 切换到指定版本，如 nvs use 16、nvs use 16.14、nvs use lts
# 仅在此次终端会话有效，下次终端会话重新使用默认版本
nvs use <version>

# 指定默认的版本，如 nvs link 16
nvs link <version>
# 这样就可以使用 nvs use 默认版本
nvs use default
# 或直接运行
nvs use

# 查看当前使用的node版本
nvs which
```

我个人常用的命令行是nvs add 1x(10,14,16的node版本), nvs use 1x

#### 自动切换node版本
在windows以及wsl上测试，只会在当前命令行生效，重新打开命令行后失效
想要使用需要再次运行 nvs auto on 
比较鸡肋，这也是这个工具的缺点，每次新打开都需要手动切换node环境

#### 全局模块安装(没咋用过，不太清楚)
在使用nvs的情况下，安装的全局模块并不会随着node版本的切换而在新的版本中可用
可以使用以下命令将某一个版本中的全局模块安装到另一个版本中

# from 为先前已经安装过全局模块的版本 to 为即将安装全局模块的版本，  
# 例如：nvs migrate 14 16，表示将14版本中的全局模块也安装到16版本中
nvs migrate <from> <to>


参考
https://github.com/jasongin/nvs