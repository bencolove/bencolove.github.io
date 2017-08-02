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
diff 		| 比較 index(stage) 和 wt(working tree)
diff -- 	| 比較 index 和 wt
diff --cached 	| 比較 HEAD 和 index
diff HEAD	| 比較 HEAD 和 wt
diff A B 	| 比較 commitA 和 commitB
diff A 		| 比較 commitA 和 wt
diff A --cached	| 比較 commitA 和 index (A默認爲HEAD)

### revert
> git reset A <file>

Example: testfile
file content | location
--- | ---
this is commit A 		| commitA id:d870564
this is commit B 		| commitB id:a713516 HEAD
this is commit B\nthis is index | index
this is commit B\nindex\nwd 	| wd

command | effect
---|---
```git reset --hard d870564```		| HEAD回退指向commitA版本,並覆蓋HEAD,index和wd
```git reset HEAD testfile		| 使用HEAD版本的testfile覆蓋index的testfile.等同於放棄index中testfile的修改
```git checkout -- testfile		| 使用index中的testfile覆蓋wd中的版本.等同於放棄wd的修改
