# 轻量级持续集成工具 -- 简单配置 一键发布
## 简介
    在开发中我们需要频繁的调整代码并发布到各种环境去测试。通常我们会花费比部署代码要多的多的时间去进入和退出各个目录、执行启动脚本、查看启动日志。当你做完这些事情你可能忘了你只是为了看看代码执行的效果。为了避免这种情况而把更多的精力放到代码测试上，借鉴Jenkins的思想，写了这么个工具。它具有以下功能：
* 自动从版本管理工具如svn、git等下载代码
* 自动用工程构建工具如maven、gradle等打包工程
* 自动上传包到远程和本地服务器（远程服务器需要账号和密码）
* 支持远程多机发布
* 自动重启远程或本地服务器
* 自动查看启动日志
* 支持windows（需要安装Git客户端）、linux、mac系统

你可以用它来发布代码到你自己的阿里云。在项目组内推广后备受好评，谁用谁知道。

`注意：本脚本默认认为发布程序到Tomcat的webapps下，若使用其他服务器请自行修改脚本中容器启动部分的代码`

## 前置条件
* 需要安装JDK并配置环境变量
* 需要安装maven
* 需要安装git或svn并配置到环境变量
* 本地部署需要tomcat支持
* 需要操作系统支持ssh命令，远程发布需要

## 使用方法

1. 在本机上将atb.sh放到任意路径下
2. 不论什么系统都将config文件放到家目录下,即`~`所代表的目录
3. atb.sh的路径设置到环境变量中
4. 修改config文件中的配置项
5. 在服务器部署一个重启脚本，见restart.sh 配置tomcat路径以及进程标志 
6. 具体用法见下面帮助

```
usage: atb [Options]

Options：
 -c                              clean 工程
 -du  [ -l ]                     跳过编译步骤直接上传已存在war包到本地服务器
 -du -r <server_flag>            跳过编译步骤直接上传已存在war包到指定的远程服务器
 -h                              帮助
 -l                              自动编译打包本地部署
 -r <server_flag>                自动编译打包远程部署到指定的远程服务器
    
```

```
 Options说明
 c  -- clean 来自mvn clean
 du -- direct upload （My Poor Chinglish）直接上传 已经打过包不需要重新打包的情况
 h  -- help  帮助
 l  -- local 本地
 r  -- remote 远程
 <server_flag>  远程server_flags用于标识上传到那一台远程服务器
```

## 示例
```
    `atb.sh -du -r` war包已存在的情况下直接发布war包到远程服务器，不存在重新打包再发布
    `atb.sh` 本地发布 等同于 `atb.sh -l`
    `atb.sh -r 244` 如果你的remote_server_flags中包含244 那么就会发布到244所代表的机器上
```

## 视频演示
* [远程发布](http://v.youku.com/v_show/id_XMjY5ODE5NDc4MA==.html?spm=a2hzp.8244740.userfeed.5!3~5~5~5!3~5~A)
* [本地发布](http://v.youku.com/v_show/id_XMjY5ODE5NjkzMg==.html?spm=a2hzp.8244740.userfeed.5!2~5~5~5!3~5~A#paction)

## 支持的操作系统
* windows 通过git for windows执行可本地发布,远程发布未测试
* linux
* mac

## 服务器脚本配置参数
`远程服务器需要修改restart.sh`
* 服务器上tomcat根目录

    `server_path="a/b/c"`
* 服务器上tomcat进程标识 杀死进程用到 一般用tomcat根目录文件夹的名字就可以了
    
    `process_flag="apache-tomcat-8.5.9"`

## 本地脚本配置参数
`本地脚本配置需要修改config文件`
* git或svn更新命令 取决于你用的是svn还是git 需要atb.sh中的checkout_code函数
    ```shell
        function checkout_code(){
            echo "正在从资源库[ $repository_url ]检出代码"
            [svn update| git pull] -- 修改此处
            return 0
        }
    ```

* 远程服务器路径

    `config_remote_server_paths=(A B C D)`
* 远程服务器用户

    `config_remote_users=(A B C D)`
* 远程服务器ip 
    
    `config_remote_ips=(A B C D)`
* 远程服务器端口 一般都是22

    `config_remote_ports=(A B C D)`
* 远程服务器密码 如果你对各个环境密码倒背如流，出于安全考虑你可以不设置，这里设置的目的是在终端打出密码，方便在输密码时拷贝。
    
    `config_remote_pwds=(A B C D)`
* maven打包用的远程profiles
    
    `config_remote_profiles=(A B C D)`
* 远程server_flags 用于区分远程服务器的标识
    
    `config_remote_server_flags=(A B C D)`

    `以上参数必须保持一致 因为数组里的元素对应了不同环境的一个属性`
* 本地profile maven的profile参数 没有可以为空
    
    `config_local_profile="dev"`
* 本地tomcat webapps目录
    
    `config_local_server_path="/home/houjinxin/document/apache-tomcat-8.0.38"`
* 项目远程build路径 一般专门用来发布版本的代码和开发代码不同时使用一个路径，自己用的话无所谓但会涉及到冲突合并的问题
    
    `config_remote_project_basepath="/home/houjinxin/document/build"`
* maven本地路径 若此处没有设置，则使用环境变量中配置的maven
    
    `config_maven_home=""`
* 项目所在本地路径 不含工程名
    
    `config_local_project_basepath="/home/houjinxin/document/trunk"`
* 项目名称
    
    `config_project_name="TaskCenter"`
* war包所在的maven子模块,只支持一个war包的工程 为空时代表在父级目录下的target中存在war包
    
    `config_war_sub_project_name=""`
* war包名
    
    `config_war_name="TaskCenter.war"`
* 远程重启shell目录 将restart脚本放到远程服务器指定的目录下，即可远程重启tomcat
    
    `config_remote_shell_dir="/home/product"`
* 项目git或svn地址
    
    `config_repository_url="svn://192.168.50.27/micro_crawler/Develop/projects/trunk/TaskCenter"`

* 本地tocmat进程唯一筛选条件，本地多实例部署时根据这一个条件杀死指定进程（Linux/Unix可用，windows未测试）
    
    `config_local_tomcat_process_name="apache-tomcat-8.0.38"`

## 我的联系方式
* Email: woshihoujinxin@163.com
* QQ: 574311651
* 微信：h574311651
