# Git brief introduction 使用簡介
# checkout list
[x]  rebase

[x]  fetch

[x]  tag
# project directory tree
>  project_directory

>     |- content (_**working tree**_)

>     |- .git (_**git repository**_)

>           |- index (_**where staged changes indexed**_)

>           |- commit_history


# diff 比較異同
> git diff A B <file> <--cached>
- 如果沒有<file>, 比較整個版本
- A,B 可以用commit_id(6位左右數字)或者HEAD(當前版本) 
- A,B 同時出現,比較A版本和B版本
- 只出現A時:
  - 如果有--cached,則比較A和stage
  - 如果沒有--cached,則比較A和working tree

用法 | 效果
--- | ---
diff 		| 比較 index(staged changes)和wt(working tree)
diff -- 	| 比較 index和wt
diff --cached 	| 比較 HEAD 和 index
diff HEAD	| 比較 HEAD 和 wt
diff A B 	| 比較 commitA 和 commitB
diff A 		| 比較 commitA 和 wt
diff A --cached	| 比較 commitA 和 index (A默認爲HEAD)


# revert 版本回復
> git [--soft | --mixed | --hard] reset A <file>

> --soft 只是移動HEAD指向A

> --mixed 移動HEAD指向A,並用HEAD版本覆蓋index,默認選項

> --hard 移動HEAD指向A,並用HEAD版本覆蓋index和working dir

命令 \ 文件的位置 | commitA<br/>id:d870564 | commitB<br/>id:a713516 | index | working dir
---|---|---|---|---
testfile文件內容 | 'commit A' | 'commit B'<br/>**HEAD** | 'commit B'<br/>'index' | 'commit B'<br/>'index'<br/>'wd'
```git reset --soft d870564```<br/>HEAD回退指向commitA版本 | 'commit A'<br/>**HEAD** | 'commit B' | 'commit B'<br/>'index' | 'commit B'<br/>'index'<br/>'wd'
```git reset HEAD testfile```<br/>使用HEAD版本的testfile覆蓋index的testfile<br/>等同於放棄index中testfile的修改<br/>commit的逆操作 | 'commit A'<br/>HEAD | 'commit B' | 'commit A' | 'commit B'<br/>'index'<br/>'wd'
```git checkout -- testfile```<br/>使用index中的testfile覆蓋wd中的版本<br/>等同於放棄wd的修改<br/>add的逆操作|'commit A'<br/>HEAD | 'commit B' | 'commit A' | 'commit A'

> ```git log```

> 顯示當前分支commit 順序

> ```git reflog```

> 顯示git歷史操作日志


# delete 刪除文件
> git rm [--cached | -f | -r] <file>

> --cached (by default) delete file from **index** only.**Failed** if unstaged changes in **wt**. use **-f** or delete the file from fs first.

> -f force to delete file from **index** and **wt**

> -r recursively on dirs


# branches 分支管理

## **創建分支**
```
$ git checkout -b dev
```
等於
```
$ git branch dev
$ git checkout dev
```
## **切換分支**
```
$ git checkout <branch>
```
## **合並分支**
合並分支有幾種:
1. fast-forward (by default)
2. non fast-forward (--no-ff)
3. rebase

### fast-forward
合並前,master 這樣子

![合並前][before merge]

如果dev分支從master開出,而且master並無其他commit,像這樣

![dev分支][ff dev branch]

這時候可以使用fast-forward merge
```
$ git checkout master
# 把dev分支合並到當前分支
# 允許情況下進行fast-forward合並
$ git merge dev
```

合並後長這樣子

![fast-forward合並][ff merge]

### non fast-forward
fast-forward merge 會保留被合並分支dev的commit路徑,但是無法分辨出曾經有分支.
而non fast-forward merge會在master上增加一個commit並表示合並結果.如果合並dev時,master已經有commit,就不能使用fast-forward了.
比如:

![dev分支][nff dev branch]

合並會會長這樣:

![合並後][nff merge]

### rebase合並

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

# remote repository 遠端庫與多人協作
多人協作模式下,遠端庫扮演着伺服器的角色.當然,git可以讓任何機器都可以設置成協作的伺服器.

## push 推送到遠端庫
1. 關聯遠端庫,增加本地庫對應的一個遠端庫,名字是origin
```
$ git remote add origin git_uri_or_url
```
2. push本地一個分支
```
# push 本地master 去origin的master分支,並把本地master和origin/master相關聯
$ git push -u origin master
```
每一個本地分支都可以和遠端庫一個分支相對應,爲方便計,使用同名而已.

## clone 克隆遠端庫
```
$ git clone git_uri_or_url
cd git_name
```
## 建置遠端庫
1. 如果遠端庫已經存在,直接使用clone
2. 如果要從頭開始建置遠端庫,可以現在伺服器端使用```git init git_name```初始化一個空白的庫,然後分2種情況創建本地庫
  - 使用```git clone```
  - 本地也使用```git init```初始化一個空白庫,然後設置對應關系```git remote add```,最後用```git push -u```建立分支對應關系
  
## 多人協作模式
### 查看遠端庫信息
```
# 對應的遠端庫名字
$ git remote
origin
# 詳細的對應關系,包括pull和push
$ git remote -v
origin	https://github.com/bencolove/bencolove.github.io.git (fetch)
origin	https://github.com/bencolove/bencolove.github.io.git (push)
```

### 建置遠端庫和本地庫
假設用如上方法建置了遠端庫之後,可以
- 直接clone遠端庫,git自動在本地初始化一個文件夾並與之對應. 或者
- 本地使用init初始化一個空白庫,關聯遠端庫,並從遠端庫Pull數據

### 分支的抓取(更新)
如果使用```git remote -v```有fetch信息的出現,就可以Pull數據了
注意:clone遠端庫後,默認只有抓取master分支.如果是在dev分支上協作,則還需要創建本地分支對應遠端分支
```
# 創建與遠端庫對應的本地分支(origin/dev是遠端分支,dev是本地分支)
$ git checkout -b dev origin/dev
$ git pull
... no tracking information for this branch
# 如果沒有設置對應關系,git不知道從遠端哪個分支抓取,所以需要
$ git pull origin dev
# 或者設置對應關系,以後只需要git pull
$ git branch --set-upstream dev origin/dev
# --set-upstream和git push時 -u是一樣意思
```
注意,抓取更新後直接會做合並,因此可能導致衝突產生,必要時要解決衝突,過程如:
1. pull/fetch 導致衝突,git會提示如:```Automatic merge failed: fix conflicts and then commit the result```
2. 解決衝突,如上
3. 本地commit結果
4. push去遠端

### 分支的推送
```
# 一般用origin作爲遠端庫名字,本地分支和遠端分支採同名
$ git push <remote repository> <local branch>
$ git push origin dev
```
如果失敗,很可能是因爲遠端庫有更新的版本,需要先pull再push

### 多人協作模式小結
**關於分支**
- master 必須時刻與遠程同步
- dev分支 做協作開發用,必須保持同步
- bug分支 可用於本地修復bug,並合並到master/dev分支,並不需要push.除非要求留下記錄
- feature分支 取決於是否協作,自行決定

**工作模式**
1. 嘗試用```git push origin <branch>```推送本地分支
2. 如果因爲遠程更新於本地而失敗,則先用```git pull```抓取更新並合並
3. 如果合並有衝突,解決衝突,本地commit
4. 最後再push一次成功

**常用命令**

命令 | 作用
---|---
查看遠端庫信息   | ```git remote -v```
推送本地分支     | ```git push origin <branch>```<br/>如果失敗,先```git pull```抓取更新再push
本地創建分支並和遠端對應 | ```git checkout -b <local_branch> origin/<remote_branch>```分支名字最好一樣
關聯本地和遠端分支 | ```git branch --set-upstream <local_branch> origin/<remote_branch>```
抓取遠端更新  | ```git pull```可能需要解決衝突


[before merge]:https://github.com/bencolove/bencolove.github.io/blob/master/imgs/ffmaster.png
[ff dev branch]:https://github.com/bencolove/bencolove.github.io/blob/master/imgs/ffdev.png
[ff merge]:https://github.com/bencolove/bencolove.github.io/blob/master/imgs/ffmerge.png
[nff dev branch]:https://github.com/bencolove/bencolove.github.io/blob/master/imgs/nffdev.png
[nff merge]:https://github.com/bencolove/bencolove.github.io/blob/master/imgs/nffmerge.png
[git flow]: https://github.com/bencolove/bencolove.github.io/blob/master/imgs/teamgit.png
