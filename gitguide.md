# brif guidelines
# project 
project directory
  |- content (_##working tree##_)
  |- .git (_##git repository##_)

# show differences
> git diff A B <file> <--cached>
- 如果沒有<file>, 比較整個版本
- A,B 可以用commit_id(6位左右數字)或者HEAD(當前版本) 
- A,B 同時出現,比較A版本和B版本
- 只出現A時:
  - 如果有--cached,則比較A和stage
  - 如果沒有--cached,則比較A和working tree

用法 | 效果
--- | ---
diff 		| 比較 wt(working tree) 和 index(stage)
diff -- 	| 比較 wt 和 index
diff --cached 	| 比較 HEAD 和 index
diff HEAD	| 比較 HEAD 和 wt
diff A B 	| 比較 commitA 和 commitB
diff A 		| 比較 commitA 和 wt
diff A --cached	| 比較 commitA 和 index (A默認爲HEAD)

# revert
> git [--soft | --mixed | --hard] reset A <file>

> --soft 只是移動HEAD指向A

> --mixed 移動HEAD指向A,並用HEAD版本覆蓋index,默認選項

> --hard 移動HEAD指向A,並用HEAD版本覆蓋index和working dir

命令\文件的位置 | commitA<br/>id:d870564 | commitB<br/>id:a713516 | index | working dir
---|---|---|---|---
testfile文件內容 | 'commit A' | 'commit B'<br/>**HEAD** | 'commit B'<br/>'index' | 'commit B'<br/>'index'<br/>'wd'
```git reset --soft d870564```<br/>HEAD回退指向commitA版本 | 'commit A'<br/>**HEAD** | 'commit B' | 'commit B'<br/>'index' | 'commit B'<br/>'index'<br/>'wd'
```git reset HEAD testfile```<br/>使用HEAD版本的testfile覆蓋index的testfile<br/>等同於放棄index中testfile的修改<br/>commit的逆操作 | 'commit A'<br/>HEAD | 'commit B' | 'commit A' | 'commit B'<br/>'index'<br/>'wd'
```git checkout -- testfile```<br/>使用index中的testfile覆蓋wd中的版本<br/>等同於放棄wd的修改<br/>add的逆操作|'commit A'<br/>HEAD | 'commit B' | 'commit A' | 'commit A'

> ```git log```

> 顯示當前分支commit 順序

> ```git reflog```

> 顯示git歷史操作日志

# delete
> git rm [--cached | -f | -r] file

> --cached (by default) delete file from **index** only.**Failed** if unstaged changes in **wt**. use **-f** or delete the file from fs first.

> -f force to delete file from **index** and **wt**

> -r recursively on dirs

# branches 分支管理

## 創建分支
```
$ git checkout -b dev
```
等於
```
$ git branch dev
$ git checkout dev
```
## 切換分支
```
$ git checkout <branch>
```
## 合並分支
```
# 把dev分支合並到當前分支
# 允許情況下進行fast-forward合並
$ git merge dev
```
## 解決衝突
1. 如果
```
$ git merge dev
...
CONFLICT ...
...
```
2. 衝突的文件看上去會像:
```
<<<<<<<< HEAD
xxxxx
========
yyyyy
>>>>>>>> dev
```

3. 修改文件內容解決衝突後再commit一次

4. 使用
```
$ git log --graph --pretty=oneline --abbrev-commit
* 59bdc1c conflict fixed
|\
| * 47efcd1 dev commit
* | 32ac89c master commit
|/
* fe82345 branch created
```


## 刪除分支
```
$ git branch -d dev
# 強制刪除分支,丟棄修改
$ git branch -D dev
```
## 查看分支
```
$ git branch
```

## 分支使用策略
- 使用原則
- bug分支
- feature分支

### 原則
master 分支僅用於版本發布, 不要在上面開分支合並
dev用作於團體協作,每個成員在dev上面開分支,提交,再合並去dev

分支開發流程的commit圖看起來會是像這樣:

![git flow][git flow]

### bug分支
bug是臨時,短暫而且急迫的任務, 需要馬上解決.

**stash**機制幫忙
1. 緩存當前工作
2. 開闢分支
3. 解決bug
4. 合並分支
5. 刪除bug分支
6. 再恢復當前工作

#### 保存當前工作
當前在dev的工作必須保存起來,使用stash
```
$ git stash
Saved working directory and index state WIP on dev: 6224937 add merge
HEAD is now at 6224937 add merge
```
用status確認working dir是幹淨的.

#### 開闢分支
需要確定在哪個分支上修復bug,如果是要馬上修復,需要在master上修復, 並命名分支bug-013
```
$ git checkout master
$ git checkout -b bug-013
```
#### 合並分支
當bug在分支bug-013上被修復後,可以合並去master
```
$ git checkout master
$ git merge --no-ff -m "merged bug fix 013" bug-013
```
#### 刪除bug分支
bug修復後,可以刪除bug分支
```
$ git branch -d bug-013
```
#### 恢復當前工作
回到dev分支,回復當時工作
恢復有兩種辦法
1. apply 和 drop. apply不會刪除工作現場,必須使用drop刪除
2. pop 恢復工作現場的同時刪除stash中的副本
```
$ git checkout dev
$ git status # 應該是幹淨的,準備恢復
$ git stash list # stash 可以重復使用,保存多個現場. list列出保存的現場
stash@{0}: WIP on dev: 6224937 add merge
$ git stash apply stash@{0}
$ git stash drop stash@{0}
# or in one go
$ git stash pop
```

### feature分支
feature分支屬實驗性質,爲開發測試新功能,而且一般開在dev分支上.
1. 開闢分支
2. 開發功能
3. 合並或者取消

#### 開闢分支
```
$ git checkout dev
$ git checkout -b feature-mobile
```

#### 合並或者取消
當分支開發完成,可以繼續測試,直到確定要合並或者取消.
```
$ git checkout dev
$ git merge --no-ff -m "add feature mobile" feature-mobile
$ git branch -d feature-mobile
```
如果是必須要取消,而且銷毀,就必須不合並而直接刪除分支.
```
$ git branch -d feature-mobile
error: the branch 'feature-mobile' is not fully merged.
...
# delete it anyway 強行刪除
$ git branch -D feature-mobile
```

### remote repository
[git flow]: https://github.com/bencolove/bencolove.github.io/blob/master/imgs/teamgit.png
