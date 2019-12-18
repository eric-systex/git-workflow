# GitLab 權限管理
Github Flow

## 需求：
* 在 master branch 當中的任何東西都是 deployable 的
* 在 master branch 中開一個敘述新功能的 branch
* 經常性的 push 到 branch(除了 master)
* 隨時都可以發 pull request
* 只有在 review 過後才 merge
* Review 過後馬上 deploy 程式碼 到 SIT

![](https://i.imgur.com/KIQVU5I.png)
![](https://i.imgur.com/UTJQDRg.png)



1. github有的兩個服務所產生的，一個是fork，另一個是pull request（簡稱PR，在GitLab為Merge Request）
2. 首先先建立一個共同的remote repository，
3. 然後各個組員用fork把此遠端數據庫fork一個回到自己的遠端數據庫。
4. 此時就像自己一個人在用一樣，開發好的功能、de好的bug，
5. 再利用PR回去共同的remote repository，
6. 讓管理員審核，通過了才merge進master。

## Project members permissions

![](https://i.imgur.com/EHFps4x.png)

### Reporter (開發團隊)
* 不可以Create new branches
* 不可以Push to non-protected branches
* 需要可以Fork, Download / View / Pull project code
* Create new merge request

相關操作請參考：
* 開發團隊.md
* KEID-163 Github Flow 更新Fork出來的專案

### Maintainer (發佈協調員)
* 建立專案，建立一個共同的remote repository
* Create new branches
* Manage/Accept merge requests
* Add tags
* Rewrite/remove Git tags

相關操作請參考：
* 發佈協調員.md

### Owner (Git系統管理員)
* Remove project
* Transfer project to another namespace

![](https://i.imgur.com/otR0oML.png)



## Protected Tags
部署流程一律走 Tag 事件，這樣可以避免工程師不小心 Merge commit 造成提前部署。

TAG限制開頭為UTA和GA只能特定人員才可以執行。


## References
* https://hellojs.readbook.tw/vcs/git/githubflow/
* https://ithelp.ithome.com.tw/m/articles/10194717
* https://docs.gitlab.com/ee/user/permissions.html
* https://docs.gitlab.com/ee/user/project/protected_tags.html

