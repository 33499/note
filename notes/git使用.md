#### git使用

##### 创建本地仓库的方法

1. **本地仓库搭建**

   ```bash
   git init      #初始化仓库
   
   git add .   #把该工作目录下的所有文件加入暂存区
   
   git commit -m "写上提交注释说明"     #项目文件提交到本地仓库
   
   git remote add origin  https://github.com/33499/deeping-learning.git       #联系自己的github远程仓库
   
   git push origin master   #推送到github
   ```

2. **克隆远程仓库**

   ```bash
   git clone [url] 
   #在GitHub上试一试
   ```

```
# 清理缓存
git rm -r --cached .
git add .
git commit -m 'update .gitignore'
```

##### git文件操作

1. 查看文件状态

   ```bash
   #查看某一文件状态
   git status [filename]
   
   #查看所有文件状态
   git status
   ```

   

2. 忽略文件

   在主目录下建立".gitignore"文件，忽略某些文件，不会上传

   ```bash
   *.txt  #忽略全部以txt为结尾的文件
   ！lib.txt #但文件除外
   /temp  #根目录下的temp文件
   build/  #忽略build目录下的所以文件
   doc/*.doc #会忽略doc/note.doc但不会忽略doc/sever/arch.doc文件
   ```

   





`git pull origin branch`拉取代码

`git push origin branch`将本地代码push到远程仓库

git remote -v 查看远程仓库

git  pull     从远程拉取最新版本 到本地  自动合并 merge            git pull origin master

git  fetch   从远程获取最新版本 到本地   不会自动合并 merge   git fetch  origin master   git log  -p master ../origin/master     git merge orgin/master

git diff temp  查看temp分支与本地原有分支的不同

git merge temp 将temp分支和本地的master分支合并

git branch -d temp  删除temp分支





#### 解决 Git 更新本地冲突：commit your changes or stash them before you can merge

方法一：stash

git stash
git pull
git stash pop