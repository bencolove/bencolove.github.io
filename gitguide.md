# brif guidelines
## project 
project directory
  |- content (_##working tree##_)
  |- .git (_##git repository##_)

### show differences
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

### revert
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

### delete
> git rm [--cached | -f | -r] file

> --cached (by default) delete file from **index** only.**Failed** if unstaged changes in **wt**. use **-f** or delete the file from fs first.

> -f force to delete file from **index** and **wt**

> -r recursively on dirs
