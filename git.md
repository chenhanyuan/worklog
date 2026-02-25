git checkout -b my-new-branch
git add ./
git commit -m ""
git push -u origin my-new-branch

git 同步最新的到自己的分支
git checkout crolin-dev
git pull
git checkout carson-dev 
git merge crolin-dev 
git push origin carson-dev 

退回原来的
git push origin carson-dev --force 

如果遇到主分支上已经提交了很多版本，如下操作：

1. 切换到主分支拉取最新代码
	git checkout main      # 切换到主分支
	git pull origin main   # 拉取远程主分支最新提交
2. 切换回你的分支并执行变基
	git checkout your-branch   # 切换到你的分支
	git rebase main            # 将你的分支变基到主分支最新提交
3. 解决冲突（如果有）
	解决冲突
	解决完成后，
	git add ***
	git commit -m ""
	git rebase --continue   # 继续变基
	如果想放弃：git rebase --abort   # 终止变基，回退到原始状态
4. 推送更新后的分支
	变基会修改提交历史，需强制推送：
	git push origin your-branch --force
	# 或更安全的强制推送（推荐）：
	git push origin your-branch --force-with-lease

git checkout t41-crolin-dev
git pull
git checkout t41-carson-dev
git pull
git rebase t41-crolin-dev
git push origin t41-carson-dev


# 1. 确保当前在你的分支A
git checkout A

# 2. 获取最新远程信息
git fetch origin

# 3. 将分支B的提交rebase到分支A上
git rebase B

# 4. 如果有冲突，解决冲突后继续
# 编辑冲突文件...
git add .
git rebase --continue

# 5. 推送到远程
git push origin A


## 暂存修改
git stash

git stash pop
