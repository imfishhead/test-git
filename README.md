
## Overview

![image](https://hackmd.io/_uploads/rJ6fsnn8bg.png)

## Development Flow

1. **確保最新狀態**：開發前先 `git pull origin dev`，確保基底程式碼是最新的。
2. **建立分支**：從 `dev` 切出 `feature/xxx` 或 `fix/xxx` 分支。
* 命名參考：`feature/login-page`, `fix/header-style`。
* `git checkout -b feature/login-page`


3. **原子化開發與 Commit**：
* 遵循 [Good Commit Message](https://ithelp.ithome.com.tw/articles/10228738) 規範（如使用 Conventional Commits: `feat:`, `fix:`, `docs:`）。
* 盡量保持 Commit 顆粒度適中，方便後續 Code Review。

4. **發起 Pull Request (PR)**：
* 推送後，開啟 PR 指向 `dev`。
* **補充**：PR 描述應包含「改動重點」及「測試方法」。


5. Code Review：由至少一名成員 Approve。
6. Merge 進 dev
    * 通常團隊內會有共識由 tech lead or developer 自己 merge）
    * 兩種做法：
        1. `git checkout dev` & `git merge feature/xxxx`
        2. 由 Github 按 merge
* **建議**：若 `dev` 有更新，合併前先在 local 端將 `dev` merge 進 feature branch 處理完 conflict。
* 完成後刪除遠端與本地的 feature branch。

### rebase / merge 情境

#### 指令解釋：
* rebase: 把目前的分支「嫁接」到目的分支，SHA1 會更改
    * `git rebase dev`: 目前分支嫁接到 dev 後
    * `git pull --rebase`: 拉下遠端的分支，將目前的分支嫁接到最後
* merge: 把目標分支合併進來，保持完整歷史
    * `git merge dev`：把 dev merge 進來目前分支
#### 情境：
* 在 feature branch 開發時：常用 `git pull --rebase` 來拉取 dev 的更新，讓自己的分支基底永遠保持最新，且線圖是一直線。
* 準備合併進 master/main 時：使用 `git merge --no-ff` (No Fast-forward)。這樣既能確保歷史簡潔，又能留下一個明確的「功能合併點」標記。
* 註：不要在公共分支（如 master 或已經 push 上遠端供他人使用的分支）執行 rebase。如果你 rebase 了遠端已經存在的分支，別人 pull 程式時會發現歷史對不起來，導致整個團隊的程式碼亂掉。["The Golden Rule of Rebasing"](https://www.gitkraken.com/blog/golden-rule-of-rebasing-in-git)。

## Release Flow

* **頻率**：每週固定 1-2 次（例：每週二、四）。
* **目的**：鎖定版本範圍，把變動變為可預期及最大程度可控，既方便測試也能確保上線品質。

1. 對主分支開 PR
* `dev` 對 `master` (或 `main`) 開 PR（確保沒有雜亂 code 被混進來），再一次 approve。
2. **合併至主分支**：
* 尚未 deploy 前跑一遍基本 CI，確保 CI 過了，並手動測試這次的改動範圍，確認跟整體程式碼合併後沒有非預期錯誤
3. deploy 
4. deploy 後手動測試改動範圍，確認 production 無異狀
5. **打上版本標籤 (Git Tag)**：
* **重要補充**：`git tag -a v1.1.0 -m "release version 1.1.0"`。這對於回溯版本至關重要。
6. **同步回 dev**：
* `master` 合併完後，務必將 `master` 合併回 `dev`，確保 `dev` 擁有最新的版本標籤與 hotfix 改動。

## Hotfix 流程

1. **建立分支**：直接從 `master` 切出 `hotfix/xxx`。
* `git checkout -b hotfix-urgent-bug master`


2. **修復與測試**：在 hotfix 分支完成修復並經過本地/測試環境驗證。
3. **雙向合併**：
* **合併至 master**：確保線上問題立即解決，並打上新 Tag（如 `v1.1.1`）。
* **合併至 dev**：**這步最容易漏掉**，必須確保開發環境也同步了這項修復，避免下次 Release 時舊 Bug 復發。

## 常用 git 指令

1. 開始開發（當你要從 dev 切出新功能時）：

* `git checkout -b <branch_name>`：建立並同時切換到新分支（最常用）。
* `git branch -a`：列出所有本地與遠端的分支。
* `git branch -d <branch_name>`：刪除已合併的本地分支（-D 為強制刪除）。
* `git remote update`：更新遠端分支資訊（當別人的分支你看不到時使用）。

2. 開發過程中的存檔動作：

* `git add .`：將所有修改過的檔案加入暫存區。
* `git commit -m "feat: login function"`：提交並加上說明文字。
* `git commit --amend`：修改最後一次的 commit（補救打錯字或漏掉的小修改）。
* `git status`：查看目前哪些檔案改了、哪些還沒 commit。

3. 獲取更新（Pull & Fetch）同步團隊進度：

* `git pull origin <branch>`：拉取遠端更新並直接合併。
* `git pull --rebase origin <branch>`：拉取更新並用 rebase 方式接在後面（保持線路一條直線，推薦使用）。
* `git fetch`：只抓取更新資訊，不進行合併（適合先觀察別人改了什麼）。

4. Merge & Rebase 將功能併入主線：

* `git merge <branch>`：將指定分支併入目前分支。
* `git merge --no-ff <branch>`：強制產生一個 merge commit（為了在線圖留下明顯的「功能合併」痕跡）。
* `git rebase -i HEAD~3`：互動式 Rebase，可以把最近 3 個 commit 壓縮、修改或刪除（上 PR 前整理紀錄的神器）。

5. 寫錯程式或想反悔時：

* `git checkout -- <file>`：放棄某個檔案的修改，回到最後一次 commit 的狀態。
* `git reset HEAD^`：撤銷最後一次 commit，但保留改過的程式碼（退回暫存狀態）。
* `git reset --hard HEAD^`：撤銷最後一次 commit，且不保留修改過的程式碼（完全回到上一步）。
* `git stash`：把目前寫到一半的東西先「藏」起來（當你要緊急切換分支修 Bug 時）。
* `git stash pop`：把剛才藏起來的東西拿出來繼續寫。

## 補充建議

### **環境對應**
* `dev` 分支對應 **Staging/QA 環境**（測試用）。
* `master` 分支對應 **Production 環境**（正式上線）。
### **自動化 (CI/CD)**
* 可以補充：當 PR 建立時，自動執行 Unit Test；當合併至 `master` 時，自動觸發部署。


## 延伸參考

* [Branch Naming Convention Cheatsheet](https://medium.com/@abhay.pixolo/naming-conventions-for-git-branches-a-cheatsheet-8549feca2534)
* [Day29｜常見的三種工作流程 - Git flow、GitHub Flow 與 Gitlab Flow](https://ithelp.ithome.com.tw/articles/10281080)
* [如何讓 GitHub 每個 pull request (PR) 有預設模板](https://riverye.com/2020/07/11/%E5%A6%82%E4%BD%95%E8%AE%93-GitHub-%E6%AF%8F%E5%80%8B-pull-request-PR-%E6%9C%89%E9%A0%90%E8%A8%AD%E6%A8%A1%E6%9D%BF/)
* [A successful Git branching model](https://nvie.com/posts/a-successful-git-branching-model/)

# test-git

jfirfiernfiwnfi3fni3rfjiorjfijfw3ijf3iknf34infi
flwenflkn;knfnf;nfknf;onf;knf;o2nf;o2nf;o2nf
jnflknflkn2l;kfn2lkfn2lkfn;2onf;o2nf;23nf;23onf;23of
fnl2knflk2nflk2nflk2nf;k2nf;k2nf;o2nf;o2nf;2onf
nfloi2nflik23nfli23nfl2nfik2nf;k2inf;o23nf;23onf2;onf2p
fn2lnfi23nfik2nfp;2onfo2jnf[o123nfk23nf;23nfo2jnfp;2ojnfp2ofjne[ofn
jklenfl2nfi2nfik2nfoik2nfpo2jfo2jfio2jfi2jf2oinfio2nfpoi23nfp23;of
eojnfl2onfi2knf3infi23nfl2i3nhfp2;iofnp2in3onfpio2niknflpinwepfoi2n3f2
fn2lfni2nfopi2hnfloijn23ijh2pifjp2ojf[po4jp;foijh2pio3fhnpiwh2o2h2fpihn2po34ff
2[ojfoi24nfi2nfiknflp2eifni24nfjnfkjenfkj24nl23fniwvhnoifnp23i;lwvjpwojfio2f
2jfp23ojfi23ni23nfwlknfwlknewklfn2iop4feofmnienfi42nfi42nfi42nfi2n]]]]



qef
qwe
fq
wef
qwef
qew
f
qwef
qew
f

