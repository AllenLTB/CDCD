[10.208.3.19 root@test-2:/usr/local]# useradd -rM jira -s /sbin/nologin

[10.208.3.19 root@test-2:/usr/local]# tar -xf atlassian-jira-software-8.8.1.tar.gz

[10.208.3.19 root@test-2:/usr/local]# ln -s atlassian-jira-software-8.8.1-standalone jira

[10.208.3.19 root@test-2:/usr/local]# mkdir jira/home

[10.208.3.19 root@test-2:/usr/local]# chown -R jira: jira/

[10.208.3.19 root@test-2:/usr/local]# echo 'export JIRA_HOME=/usr/local/jira/home' > /etc/profile.d/jira.sh
[10.208.3.19 root@test-2:/usr/local]# source /etc/profile.d/jira.sh

默认jira会监听在8080端口，下面我将服务器端口更改为8089，Connector端口更改为8090。

```XML
[10.208.3.19 root@test-2:/usr/local]# source /etc/profile.d/jira.sh
<Server port="8089" shutdown="SHUTDOWN">
...
   <Service name="Catalina">
      <Connector port="8090"
         maxThreads="150"
         minSpareThreads="25"
         connectionTimeout="20000"
         enableLookups="false"
         maxHttpHeaderSize="8192"
         protocol="HTTP/1.1"
         useBodyEncodingForURI="true"
         redirectPort="8443"
         acceptCount="100"
         disableUploadTimeout="true"/>
```



必须是mysql，不能是mariadb或者percona

create database jira;

create user jira@'10.208.3.%' identified by '123123';

grant all on jira.* to jira@'10.208.3.%';



更改mysql配置文件并重启mysql

```none
[mysqld]
default-storage-engine=INNODB
character_set_server=utf8mb4
innodb_default_row_format=DYNAMIC
innodb_large_prefix=ON
innodb_file_format=Barracuda
innodb_log_file_size=2G
sql_mode = NO_AUTO_VALUE_ON_ZERO
```



启动jira

拷贝MySQL JDBC driver到Jira安装目录

[10.208.3.19 root@test-2:/usr/local]# cp /software/Jira/mysql-connector-java-5.1.49.jar /usr/local/jira/lib/

[10.208.3.19 root@test-2:/usr/local]# jira/bin/start-jira.sh 





![image-20200509231032447](.assets/image-20200509231032447.png)



![image-20200509231149212](.assets/image-20200509231149212.png)



![image-20200509234007965](.assets/image-20200509234007965.png)

![image-20200509234655548](.assets/image-20200509234655548.png)



点击generate a Jira trial license生成一个license

这回生成一个试用的license

![image-20200509235833911](.assets/image-20200509235833911.png)



![image-20200509235940935](.assets/image-20200509235940935.png)



AAABeA0ODAoPeNp9kUFvgkAQhe/8CpJe2sOSBYtBE5K2QCstohHqqZctjroNApldbP33RdAUq3jb2d1575s3NxGT6pjtVDpQdTo0B0Odqo4bqwY1qLJCgGydFwWgFvAEMgHegkueZ7YXxt5sOvMjTwnLzSfgZPkuAIVNdMXJM8kSGbIN2KnV06k1MHTdelhtGE+1JN8oXxyZdtY3LTFZMwEuk2DvAQg1iU6Vg3W8K6DWdCbjsTdz/Mfg+OT9FBx3rb4+oYMjhzeubDtAIsAtoO/aT8GzRZwX0yXx6I2SuW+OGsoC80WZSG1fEJEv5TdD0CpZvgVbYgnNt+54LoR4aZIKMpOQsSzpmOYKzVmSB59qrsB3Iy8kgW7eG72+1Veqyj69uSIcSYYS0F6yVIAywRXLuGDNhFuWlvVRcRDqw/+9pQ3FvILadxgnUUA1LRbIxSFFF0SCvKi1XysINTpAqLfNku4+hmrLtKbuWsOlgNvm7b4/zab+BQAzCeMwLAIUamMOwIctEc6sl5cg04rmgF9yN7sCFDuYFfgiThzj82BYbUy0HLFQS4LVX02ia



![image-20200510000042286](.assets/image-20200510000042286.png)



![image-20200510104955442](.assets/image-20200510104955442.png)



![image-20200510105251121](.assets/image-20200510105251121.png)



![image-20200510105345780](.assets/image-20200510105345780.png)



![image-20200510105429904](.assets/image-20200510105429904.png)



![image-20200510105521561](.assets/image-20200510105521561.png)



![image-20200510110020189](.assets/image-20200510110020189.png)









Jira虽然能使用Git插件来创建分支，可以会出现消耗资源与不稳定的情况，而且需要手动创建（手动输入Jira ID）

![image-20200510112554557](.assets/image-20200510112554557.png)





![image-20200510112517750](.assets/image-20200510112517750.png)



特性分支开发，然后合并为一个release分支，在release分支验证，没问题只有用release分支发布到生产。如果release分支发布到生产没问题了，将release分支合并到生产。



![image-20200510112923411](.assets/image-20200510112923411.png)







# Jira通过webhook触发Jenkins在GitLab上创建分支，分支名称与是Jira中的问题名称

**在Jira上创建一个Webhook，并选择该Webhook会被什么类型的事件触发**

URL：https://jenkins-netadm.leju.com/generic-webhook-trigger/invoke?token=jira-devops-service&projectKey=${project.key}

![image-20200510151417713](.assets/image-20200510151417713.png)



![image-20200510151442263](.assets/image-20200510151442263.png)

**Jenkinsfile**

```GROOVY
[10.208.3.24 root@test-6:~/jenkinslib]# cat Jenkinsfiles/jira.jenkinsfile
@Library('jenkinslib@master')
def tools = new org.devops.tools()
def gitlab = new org.devops.gitlab()
pipeline {
    agent { node{label "scanner"} }
	options {
		timeout(time: 1, unit: 'HOURS') 
		timestamps()
		buildDiscarder(logRotator(numToKeepStr: '10'))
	}
    triggers {
        GenericTrigger(
        	genericVariables: [
                [key: "webHookData", value: "\$", expressionType: "JSONPath", regexpFilter: "", defaultValue: ""]
            ],
        	genericRequestVariables: [
                [key: "projectKey", regexpFilter: ""]
            ],
            genericHeaderVariables: [
            ],
            token: 'jira-devops-service',
            causeString: 'Triggered on Jira',
            printContributedVariables: true,
            printPostContent: true,
            silentResponse: true
        )
    }
    stages {
        stage('FilterData') {
            steps{
                script{
					response = readJSON text: """${webHookData}"""
					env.eventType = response["webhookEvent"]
					tools.PrintMes("${eventType}","green")
					switch (eventType){
						case("jira:issue_created"):
							env.issueName = response["issue"]["key"]
							env.userName = response["user"]["name"]
							env.moduleNames = response["issue"]["fields"]["components"]
							env.fixVersion = response["issue"]["fields"]["fixVersions"]
							tools.PrintMes("Trigger by ${userName} ${eventType} ${issueName}","green")
							currentBuild.description = "Trigger by ${userName} ${eventType} ${issueName}"
							break
						case("jira:issue_updated"):
							env.issueName = response["issue"]["key"]
							env.userName = response["user"]["name"]
							env.moduleNames = response["issue"]["fields"]["components"]
							env.fixVersion = response["issue"]["fields"]["fixVersions"]
							tools.PrintMes("Trigger by ${userName} ${eventType} ${issueName}","green")
							currentBuild.description = "Trigger by ${userName} ${eventType} ${issueName}"
							break
					}
                }
            }
        }
		stage('CreateBranchOrMR'){
			when {
				anyOf {
					environment name: 'eventType', value: 'jira:issue_created'   //issue 创建 /更新
					environment name: 'eventType', value: 'jira:issue_updated'
				}
			}
			steps {
				script{
					def projectIds = []
					projectName = moduleNames 
					fixVersion = readJSON text: """${fixVersion}"""
					println(fixVersion.size())

					//获取GitLab项目ID
					def projects = readJSON text: """${moduleNames}"""
					println(projects)
					for (project in projects){
						projectName = project["name"]
						println(projectName)
						currentBuild.description += "\n project: ${projectName}"
						try {
							projectId = gitlab.GetProjectId(projectName)
							println(projectId)
							projectIds.add(projectId)
						} catch(e){
							println(e)
							println("未获取到项目ID，请检查模块名称！")
						}
					}
					println(projectIds)
					if (fixVersion.size() == 0) {
                        for (id in projectIds){
							searchResult = gitlab.SearchRepositoryBranch(id,"${issueName}")
							if (searchResult == 'true') {
								println("已存在${issueName}分支.")
                            	currentBuild.description += "\n 该特性分支已存在, 未进行创建 --> ${id} --> ${issueName}"
							} else {
                            	println("新建特性分支--> ${id} --> ${issueName}")
                            	currentBuild.description += "\n 新建特性分支--> ${id} --> ${issueName}"
                            	gitlab.CreateBranch(id,"${issueName}","master")
							}
                        }
                    } else {
                        fixVersion = fixVersion[0]['name']
                        println("Issue关联release操作,Jenkins创建合并请求")
                        currentBuild.description += "\n Issue关联release操作,Jenkins创建合并请求 \n ${issueName} --> RELEASE-${fixVersion}" 
                        for (id in projectIds){
							searchResult = gitlab.SearchRepositoryBranch(id,"ELEASE-${fixVersion}")
							if (searchResult == 'true') {
								println("已存在RELEASE-${fixVersion}分支.")
                            	currentBuild.description += "\n 该特性分支已存在, 未进行创建 --> ${id} --> RELEASE-${fixVersion}"
							} else {
                            	println("创建RELEASE-->${id} -->${fixVersion}分支")
                            	gitlab.CreateBranch(id,"RELEASE-${fixVersion}","master")
                            		
                           		println("创建合并请求 ${issueName} ---> RELEASE-${fixVersion}")
                           		gitlab.CreateMr(id,"${issueName}","RELEASE-${fixVersion}","${issueName}--->RELEASE-${fixVersion}")
							}
						}
                    }
				}
			}
		}
    }
}
```

**GitlabAPI共享库（只列出了上面涉及到的方法）**

```GROOVY
[10.208.3.24 root@test-6:~/jenkinslib]# cat src/org/devops/gitlab.groovy 
package org.devops

//封装HTTP
def HttpReq(reqType,reqUrl,reqBody){
	def gitServer = "https://gitlab-netadm.leju.com/api/v4"
	def tools = new org.devops.tools()
	withCredentials([string(credentialsId: 'gitlab-root-token2', variable: 'gitlabToken')]) {
		result = httpRequest(
					customHeaders: [[maskValue: true, name: 'PRIVATE-TOKEN', value: "${gitlabToken}"]],
					httpMode: reqType,
					contentType: "APPLICATION_JSON",
					consoleLogResponseBody: true,
					ignoreSslErrors: true,
					requestBody: reqBody,
					//quiet: true,
					url: "${gitServer}/${reqUrl}")
	}
	return result
}

//获取项目ID
def GetProjectId(projectName) {
	apiUrl = "projects?search=${projectName}"
	response = HttpReq('GET',apiUrl,'')
    response = readJSON text: """${response.content}"""
	result = response[0]["id"]
	println(result)
	return result
}

//搜索分支
def SearchRepositoryBranch(projectId,searchKey){
	apiUrl = "projects/${projectId}/repository/branches?search=${searchKey}"
	response = HttpReq('GET',apiUrl,'')
	response = readJSON text: """${response.content}"""
	if (response.size() == 0) {
		println("不存在匹配${searchKey}关键字的分支")
		return 'false'
	} else {
		result = response["name"]
		println("匹配${searchKey}关键字的分支有: ${result}")
		return 'true'
	}	
}

//创建分支
def CreateBranch(projectId,newBranchName,refBranchName) {
	apiUrl = "projects/${projectId}/repository/branches?branch=${newBranchName}&ref=${refBranchName}"
	response = HttpReq('POST',apiUrl,'')
	response = readJSON text: """${response.content}"""
	branchList = ListRepositoryBranch(projectId)
	for (name in branchList) {
		if (name == newBranchName) {
			println("Create ${newBranchName} success.")
		}
	}
}
```

**在Jira上创建问题**

在Jria上创建项目（不需要和GitLab上的仓库同名）

![image-20200510105602110](.assets/image-20200510105602110.png)

![image-20200510105627888](.assets/image-20200510105627888.png)

![image-20200510105656910](.assets/image-20200510105656910.png)

创建组件（在我们的示例中这表示为GitLab上面的仓库名称）

![image-20200510194538079](.assets/image-20200510194538079.png)

在项目中创建问题

下面我选择了两个组件（gitlab上存在的两个仓库），根据需要选择几个都行

![image-20200510194801264](.assets/image-20200510194801264.png)

点击创建之后Jenkins就会被Jira通过Webhook触发，从而在GitLab上创建分支，分支的名字就是问题的名字，咱们创建的问题名字是DEV1-31（自动生成的）

![image-20200510195053441](.assets/image-20200510195053441.png)

**实际效果**

从Jenkins的构建历史可以看出，第一次触发时成功创建，第二次触发（更新信息了）是因为已存在所以未创建

![image-20200510194253257](.assets/image-20200510194253257.png)

**GitLab上两个仓库的分支的确已经创建成功**

Tips：上下两个图中，DEV1-31分支旁边的时间不一样，不用因为这个有疑惑。这时间表示分支上一次更新时什么时候（如果是新创建的分支就表示它的父分支上一次更新是什么时候）

![image-20200510195336125](.assets/image-20200510195336125.png)

![image-20200510195409439](.assets/image-20200510195409439.png)