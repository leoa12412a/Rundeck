# Rundeck

## 安裝

參考官方的<a href="https://docs.rundeck.com/docs/administration/install/linux-rpm.html#open-source-rundeck">installing on CentOS</a>以及<a href="https://docs.rundeck.com/downloads.html">Downloads</a>

### 安裝java
```
yum install java-1.8.0
```

### yum rpm 安裝 rundeck
```
rpm -Uvh https://repo.rundeck.org/latest.rpm
yum install rundeck
```

### 開啟rundeck
```
service rundeckd start
```

### 驗證服務是否正確啟動
```
tail -f /var/log/rundeck/service.log
```

### 看到類似以下內容的服務即已準備就緒
```
Grails application running at http://localhost:4440 in environment: production
```

## 配置

### Rundeck檔案位置

安裝完成後

- 配置檔位置:
```
/etc/rundeck
```
- 程式檔位置:
```
/var/lib/rundeck
```

### 修改配置檔IP

修改/etc/rundeck/rundeck-config.properties
```
grails.serverURL=http://122.147.213.60:4440
```

修改/etc/rundeck/framework.properties
```
framework.server.name = localhost
framework.server.hostname = localhost
framework.server.port = 4440
framework.server.url = http://122.147.213.60:4440
```

關閉SELinux
```
sudo vi /etc/sysconfig/selinux
reboot 
```

開啟防火牆4440port
```
firewall-cmd --add-port=4440/tcp --permanent
firewall-cmd --reload
```

## 初步教學

### 訪問Web GUI

訪問 http://{ip/domain}:4440

預設帳號與密碼皆為admin

### 建立專案Project

登入後會看到首頁長這樣(v3.2.6)，點選New Project+建立新專案
![image](img/dashboard.PNG)<br>

如果沒有特別的配置填好專案名稱就可以按Create
![image](img/new_project.PNG)<br>

### 建立節點Node

建立好專案後，預設會先請你編輯你的節點(Node)，上面已經幫你建立了local的節點就是這台Server
需要建立節點按下Add a new Node Source按鈕
![image](img/edit_node.jpg)<br>

這裡我們先選擇由檔案產生節點
![image](img/add_new_node_source.jpg)<br>

文件的配置
File Path填入專案的名稱方便辨識
```
/var/lib/rundeck/projects/{your_project_name}/etc/resource.xml
```
![image](img/add_node_file.png)<br>

建立成功後就可以在/var/lib/rundeck/projects下看到我們產生的檔案
打開resource.xml會長的像下面這樣
```
<?xml version="1.0" encoding="UTF-8"?>

<project>
  <node name="localhost" description="Rundeck server node" tags="" hostname="localhost" osArch="amd64" osFamily="unix" osName="Linux" osVersion="3.10.0-1062.el7.x86_64" username="rundeck"/>
</project>
```
- name: 節點名稱
- description: 描述
- tags: 標籤
- hostname: ip或domain
- username: 用哪個使用者登入

想要新增節點就在<project></project>新增一行<node ...></node>

然後點選左側的Node選項就可以看到配置在文件裡的節點

### 建立工作Job

點選左側欄位進入Job選單，並點選建立新的工作
![image](img/job.jpg)<br>

先填寫Job的名稱，並到Workflow做設定
![image](img/job_workflow.jpg)<br>

這邊選擇command並輸入whoami
![image](img/job-workflow-command.jpg)<br>

建立好Job後，按下剛剛建立的Job並再跳出的視窗按下Run Job Right執行Job，就會看到執行的結果
![image](img/job-result.jpg)<br>


## Rundeck 和 Ansible

### rundeck-ansible-plugin

官方表示
```
Rundeck makes Ansible even better
```
因為在Rundeck的社區裡很多人表示喜歡把Rundeck 和 Ansible一起使用，所以官方也推出了<a href="https://github.com/Batix/rundeck-ansible-plugin">rundeck-ansible-plugin</a>


### 安裝

1. <a href="https://github.com/Batix/rundeck-ansible-plugin/releases">下載最新版的.jar檔</a>
2. 把.jar檔案放在/var/lib/rundeck/libext下
3. 修改/etc/ansible/ansible.cfg
```
[defaults]
host_key_checking=false
```
4. 如果使用SSH公鑰連結節點的使用者請將/var/lib/rundeck/.ssh/id_rsa.pub配置給子節點，用法請參考<a href="https://github.com/leoa12412a/Ansible/blob/master/README.md#%E4%BD%BF%E7%94%A8ssh%E5%85%AC%E9%91%B0%E6%86%91%E8%AD%89">這裡</a>

### 測試 
測試rundeck是否可以訪問Ansible的配置文件和密鑰
```
su rundeck -s /bin/bash -c "ansible all -m ping"
```
如果無法成功的話可以參考<a href="https://github.com/Batix/rundeck-ansible-plugin#requirements">官方文件</a>

### 網頁上的配置
建立一個新專案，並在Default Node Executor選擇Ansible Ad-Hoc Node Executor，且配置如下
![image](img/default_node_executor.jpg)<br>

並在Edit Nodes以Ansible Resource Model Source的方式新建Node source
![image](img/Ansible_Resource_Model_Source.jpg)<br>

配置Ansible的節點檔案和配置檔
![image](img/Ansible_Resource_Model_Source_Setting.jpg)<br>

就可以看到Ansible裡的節點
![image](img/ansible_node.jpg)<br>

## 在Rundeck中執行Ansible playbook

若要在Rundeck中執行playbook，在產生Job時Workflow配置選擇Workflow Steps中的ansible插件

- Ansible Module : 使用一個Ansible模組，目前嘗試還沒成功過(錯誤:ERROR! Specified hosts and/or --limit does not match any hosts)                        
- Ansible Playbook Inline : 類似直接在Rundeck上撰寫playbook的方法
- Ansible Playbook : 使用已經存在Playbook

至於Node Steps 和 Workflow Steps的差異，<a href="https://docs.rundeck.com/docs/manual/job-workflows.html#workflow-steps">官方表示</a>
```
Node Steps operate once on each Node, which could be multiple times within a workflow. For a full list of Node Steps, see Job Plugins Node Steps
Workflow Steps operate only once in the workflow. For a full list of Workflow Steps, see Workflow Steps
```
大概的意思是Node Steps有可能會工作多次，Workflow Steps只會執行一次，實測執行本機playbook兩個都能夠執行

### Workflow Steps中Ansible Playbook的配置

![image](img/workflow-cfg1.jpg)<br>
![image](img/workflow-cfg2.jpg)<br>

前面就填寫playbook的yml路徑在哪裡，ansible路徑如果沒變的話可以不填

第二章圖的就很重要，如果沒有特別的指定執行個別節點的話，Disable Limit一定要打勾。
這指的是從Rundeck傳遞指令給ansible時不傳遞--limit參數，若不打勾會造成以下錯誤
![image](img/error.jpg)<br>
若要選擇指定節點的話在Extra Ansible arguments中輸入而外的指令，如下:
```
-i /etc/ansible/hosts --limit==all
```
以上配置完成就可以儲存。
就可以在Rundeck上使用Ansible的Playbook了

## 管理Rundeck中的使用者

### 修改管理員密碼

於/etc/rundeck/realm.properties修改，預設如下
```
admin:admin,user,admin,architect,deploy,build
```
使用名稱(帳號):密碼,後面都是擁有的權限，關於權限後面會再敘述

若要將密碼加密可以使用md5
```
admin:MD5:E10ADC3949BA59ABBE56E057F20F883E,user,admin,architect,deploy,build
```
若要加入新的使用者直接在下方加入即可
```
new_user:MD5:E10ADC3949BA59ABBE56E057F20F883E,權限...
```

### 配置管理員權限
可以於/etc/rundeck/admin.aclpolicy直接配置權限
或是在GUI上也可以配置
![image](img/access_control.png)<br>
若要配置使用者基礎功能可以使用<a href="example/9skin.aclpolicy">此文件</a>(只能看到和使用，無法添加、刪除、修改任何內容)
