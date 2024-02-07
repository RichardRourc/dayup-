

##### 方法

1、直接将一些项目修改远程仓库为个人git

2、个别私人新建项目设置为个人git账号



第一种方法：

我在试的时候，因为没将那个仓库git账号修改为个人的或者是git抽风了吧，一直无法赋权，也就无法登录



第二种方法：

新建个人仓库，单独设置git为个人git仓库，git提示赋权方式，我去到了浏览器赋予权限，然后成功设置并上传了 （如果是新建的git，有提示命令行如何操作一些git远程仓库设置）

// 初始git
git init

```js
// 写文件
something write
```

// 添加
git add .

// 提交说明
git commit -m "fix: test"

// 关联个人远程仓库
git remote add origin yourregister.git

// 项目设置git用户名
git config user.name "yourname"

// 项目设置git用户邮箱
git config user.email "youreamil@example.com"

// 推送到远程仓库的远程分支
git push --set-upstream origin master