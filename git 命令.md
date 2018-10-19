## 分支相关
#### 从master创建分支
```$xslt
$ git checkout -b xxx
```
创建完分支之后，需要将该本地分支和远程分支对应
```$xslt
$ git push origin xxx
```
注意/ 和 空格
#### 如果从master拉下的分支，此时是指向master的，指向自己的远程分支的命令:
* 如果远程已有分支
```$xslt
$ git branch -u origin/xxxx
或者
$ git branch --set-upstream xxx origin/xxx
```
* 如果远程没有该分支，参照↑
#### 从远程分支拉下一个分支并切换
```$xslt
$ git checkout -b xxx origin/xxx
```
## 合并分支相关
#### merge方式
从master合并到该分支
```$xslt
$ git merge origin/master
```
#### pull ==fetch+merge
```$xslt
$ git pull origin master
```
