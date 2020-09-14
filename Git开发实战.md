### Git实战

#### 第一章	快速入门

##### 1.1 什么是Git？

```
是一款分布式、用于做版本控制的软件。
```

##### 1.2 为什么要做版本控制？

```
保留之前的所有版本，以便会滚和修改。
```

##### 1.3 安装git

#### 第二章	“东北热”创业史

##### 2.1 第一阶段：单枪匹马开始干

```python
想要让git对一个目录进行版本控制需要以下步骤：    
1.进入要管理的文件夹
2.执行初始化命令
	git init
3.管理目录下的文件状态
	git status
4.管理指定文件
	git add 文件名
	git add .
5.个人信息配置：用户名、邮箱【一次即可】
	git config --global user.email "你的邮箱"
	git config --global user.name "你的名字"
6.生成版本
	git commit -m '描述信息'
```

##### 2.2 扩展新功能

```
git add
git commit -m '短视频'
```

##### 2.3 第三阶段：“约饭事件”

```
1.回滚至之前版本
	git log
	git reset --hard 版本号
	
2.回滚至之后的版本
	git reflog
	git reset --hard 版本号
```

##### 2.4 总结

![](C:\Users\mutang\Desktop\python\git流程2.png)



```
git branch 查看分支
git branch dev 创建dev分支
git checkout dev 切换到dev分支

合并分支
git branch bug 创建bug分支，修复bug
git checkout master 切换到master
git merge bug 把bug分支合并到master上（可能会产生冲突）
git branch -d bug 删除bug分支
```

#### 第三章	远程（GitHub）仓库

##### 3.1 从家推送代码到远端

```
1.创建GitHub账号
2.创建仓库
3.给远程仓库起别名，推送
	git remote add origin 仓库地址
	git push -u origin master  (有可能要输入git用户信息)	
```

##### 3.2 在公司第一次从远端拉代码

```
1.创建文件
2.进入文件
3.克隆代码
	git clone 仓库地址 （内部已实现git remote add origin 仓库地址）
```

##### 3.3 第一天白天公司开发，推送

```
1.切换分支，在dev分支做开发
	git checkout dev	
2.下班推送dev分支代码
	git add .
	git commit -m '描述信息'
	git push origin dev
```

##### 3.4 第一天晚上在家开发，推送

```
1.进入项目文件
2.切换到dev分支
	git checkout dev
3.从远端拉取代码
	git pull origin dev
4.开发，生成版本
	git add .
	git commit -m '描述信息'
5.推送dev分支
	git push origin dev
```

##### 3.5 第二天白天公司开发，推送

```
1.进入项目文件
2.切换到dev分支
	git checkout dev
3.从远端拉取代码
	git pull origin dev
4.开发，生成版本
	git add .
	git commit -m '描述信息'
5.推送dev分支
	git push origin dev
```

##### 3.6 知识点

```
git pull origin dev   从远端到工作区
	等价于
git fetch origin dev  从远端到版本库
git merge origin/dev  从版本库到工作区
```

#### 第四章	rebase(变基)

```
使git记录变得简洁。
```

##### 4.1 第一种应用场景

```
git记录：
	C1 <--- C2 <--- C3 <--- C4 <--- C5 <--- C6 <--- C7
	
git rebase -i 版本号
	比如写C4版本号，那就是把C4、C5、C6、C7这几条记录合并为一条
git rebase -i HEAD~数字
	比如HEAD~3，就是把最新的三条C5、C6、C7记录合并为一条
	
注意：合并的记录不要涉及已经push到远端的版本记录
```

##### 4.2 第二种应用场景

```
比如不想要dev分支，把dev分支记录合并到master

步骤：
	1.切换到dev
		git checkout dev
	2.redase
		git rebase master
	3.切换到master
		git checkout master
	4.merge
		git merge dev
```

##### 4.3 第三种应用场景

```
比如dev分支，当本地与远端dev分支内容不一致时，pull代码的时候会产生记录分支，可以采用fetch+rebase解决

git pull origin dev  (不建议使用)
	等价于
git fetch origin dev
git rebase origin/dev
```

##### 4.4 注意

```
1.在使用git rebase时，可能会产生冲突
2.不要慌，解决冲突
3.git add .
4.git rebase --continue
```

#### 第五章	Beyond Compare

##### 5.1 快速解决冲突

```
1.安装Beyond Compare
2.在git中配置
	git config --local merge.tool bc3
	git config --local mergetool.path '/usr/local/bin/bcomp'
	git config --local mergetool.keepBackup false
3.应用Beyond Compare解决冲突
	git mergetool
```

#### 第六章	多人协同工作流

##### 6.1 流程图

![](C:\Users\mutang\Desktop\python\git多人协同.png)

```
master：用于放发行的稳定版本
基于mater的dev：专用于开发
基于dev的feature：个人开发（多条）
```

##### 6.2 代码review

```
通过pull request发起codereview请求
```

#### 第七章	如何给开源项目提供代码

```
1.fork源代码
	将别人源代码拷贝到自己的远程仓库
2.在自己仓库进行修改代码
3.给源代码的作者提交修复bug的申请（pull request）
```

#### 第八章	其他知识点

##### 8.1 配置文件

```
1.项目配置文件：项目/.git/config
	git config --local user.name 'name'
	git config --local user.email 'email'
	
2.全局配置文件：/.gitconfig
	git config --global user.name 'name'
	git config --global user.email 'email'
	
3.系统配置文件：/etc/.gitconfig
	git config --system user.name 'name'
	git config --system user.email 'email'
	
	注意：需要有root权限
```

##### 8.2 免密码登陆

```
1.URL中体现
	原来的地址：https://github.com/yangxuecheng/dbhot.git
	修改的地址：https://用户名:密码@github.com/yangxuecheng/dbhot.git
	
	git remote add origin https://用户名:密码@github.com/yangxuecheng/dbhot.git
	git push origin master
	
2.SSH实现
    1.生成公钥和私钥（默认放在~/.ssh目录下，id_rsa.pub公钥，id_rsa私钥）
        ssh_keygen
    2.拷贝公钥中的内容，并设置到GitHub中
    3.在git中配置ssh地址
        git remote add origin ssh地址
    4.以后使用
        git push origin master
        
3.git自动管理凭证
```

##### 8.3 git忽略文件

```
让git不再管理当前目录下的某下文件。

*.h
!a.h
files/
*.py[c|a|b]

更多参考：https://github.com/github/gitignore
```

##### 8.4 任务管理相关

```
1.issues
2.wiki
```































































































































