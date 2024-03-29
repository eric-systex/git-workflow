# DevOps工程師
以下為 Jenkins 設定

### Create Folder
委請 **Jenkins系統管理員** 建立 Folder。各微服務 Jobs 全數歸屬於該 Folder
![](https://i.imgur.com/gmXiKZt.png)



### Create Roles
委請 **Jenkins系統管理員** 新增二個 Global Roles 並設定其權限

1. builder
    * 整體: Read
    * Job: Build
2. Member
    * 整體: Read

委請 **Jenkins系統管理員** 新增二個 Project Roles 並設定其權限

1. ^目錄名稱.*  (eg: ^jupiter.*)
    * Credentials: Create, Delete, Update,  View
    * Job: Create, Read
2. ^目錄名稱/.*  (eg: ^jupiter/.*)
    * Job: Build, Cancel, Configure, Delete, Discover, Read, Workspace
    * Execute: Delete, Replay, Update

![](https://i.imgur.com/1hN8sx6.png)


### Assign Roles
委請 **Jenkins系統管理員** 將 **DevOps工程師** 授與 上述 Global Roles 及 Project Roles
![](https://i.imgur.com/cWZ3rv4.png)



### Add Credentials
進入專案 Folder 點擊 Credentials，再 global 處下拉 Add credentials，新增 GitLab 認證資訊 (Kind: Username with password)
![](https://i.imgur.com/VeYGgRl.png)
![](https://i.imgur.com/NywF5L4.png)
![](https://i.imgur.com/MDrLUwG.png)



### Create Jobs
以微服務為單位建立 Pipeline Job
![](https://i.imgur.com/SjfM7Zw.png)



#### Build Triggers 設定
* 勾選「Build when a change is pushed to GitLab. 」
* 勾選「Push Events」「Opened Merge Request Events」（預設值）
* Allowed branches 選擇「Filter branches by regex」填入 (.*master.*|.*tags.*) 
* 按下 GENERATE 產生 secret token

![](https://i.imgur.com/Alxwtuy.png)
![](https://i.imgur.com/pd7LEyw.png)


#### Pipeline 設定
固定將 Jenkinsfile 檔案擺放在 該微服務 GitLab Repository 根位置，其片段內容如下

當 GitLab 的 Webhook 觸發 Jenkins 時，會帶入一些[環境變數](https://github.com/jenkinsci/gitlab-plugin#defined-variables)，我們利用下列環境變數來取得該次 Merge Request 與 Jira Backlog 的相關資訊，以便通知 **可靠性工程師** 該次更版的測試項目

1. gitlabActionType：GitLab事件（PUSH、TAG_PUSH）
2. gitlabAfter：GitLab事件當下的 Commit
3. gitlabBranch：GitLab事件當下的 Branch（若是 TAG_PUSH事件，則是 Tag）

```
node {

    def scm_credential = 'gitlab'
    
    def project = env.gitlabSourceNamespace
    def ap_name = env.gitlabSourceRepoName
    
    def scm_checkout = env.gitlabAfter
    if (env.gitlabActionType == 'TAG_PUSH')
        scm_checkout = env.gitlabBranch

    stage ('checkout') {
        echo "♣♣♣ 自 SCM 取出源碼: ${scm_checkout}"
        checkout([$class: 'GitSCM', branches: [[name: "${scm_checkout}"]], userRemoteConfigs: [[credentialsId: "${scm_credential}", url: "${env.gitlabSourceRepoHttpUrl}"]]])
    }
 
    stage ('notify') {
        echo "♣♣♣ 發送此次建構結果"
        def notify_msg = "[${project}] ${ap_name} deploy SUCCESS\n - Target: target_host\n - Image: docker_tag\n - Action: ${env.gitlabActionType}"
        def merge_title = scm_checkout.replace("refs/tags/", "")
        if (env.gitlabActionType == 'PUSH') {
            merge_title = sh returnStdout: true, script: "git log --format=%B -n 1 ${scm_checkout}"
        }
        notify_msg = "${notify_msg}\n\nThe following functions need to be tested\n${merge_title}"
        slackSend channel: '#cicd', color: 'good', message: "${notify_msg}"
    }
}
```

### 提供 Hook 相關資訊
以微服務為單位，提供以下資訊給 **發佈協調員** 協助設定 Hook

1. Hook URL
2. Secret Token
3. 指定 Push events 及 Tag push events 作為 Trigger 事件
