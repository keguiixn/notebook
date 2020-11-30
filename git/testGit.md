git add *
git status
git commit -m 'v2'
git log  查看版本
git reset --hard  HEAD^ 回退到当前版本的上个版本 git reset --hard  版本id
git diff 查看修改的不同点
git restore  + 文件名   撤销修改（没存到暂存区）
git restore  --staged +文件名 （从暂存区撤销文件）
git rm +文件名
git clone 将代码从远程克隆下来
git push 将代码推送到远程仓库
git pull 将代码从远程仓库拉取下来
git chechout -b dev 创建并切换分支  
git branch 创建分支
git checkout 切换分支
进入主分支合并其他分支代码
git merge 其他分支
git branch -d 删除分支
git push -u origin/biye+分支名 将本地分支push到远程

git push origin  branch:branch  创建并推送代码到远程仓库