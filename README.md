# 基于Jenkins的持续集成CI
浙江新闻、浙江24小时、县市报等APP采用了基于jenkins的持续集成CI。  

**CI（continuous integration）持续集成**  
一次构建：可能包含编译，测试，审查和部署，以及其他一些事情，一次构建就是将源代码放在一起，并验证软件是否可以作为一个一致的单元运行的过程。可以理解为频繁的在多个团队的工作中集成，并且给与反馈的过程。团队开发成员经常集成它们的工作，每次集成都通过自动化的构建（包括编译，发布，自动化测试）来验证，从而尽早地发现集成错误。

**CI场景如下：**  
1、开发人员向版本控制库提交代码，同时，集成构建计算机上的CI服务器正在轮询检查版本控制库中的变更  
2、在提交发生之后，CI服务器检测到版本控制库中发生了变更，所以CI服务器会从库中取得最新的代码副本，执行构建脚本，该脚本将对软件进行集成  
3、CI服务器向指定的项目成员发成电子邮件，提供构建结果的反馈信息。  
4、CI服务器继续轮询版本控制库中的变更。  

**Jenkins**  
Jenkins 是一个开源项目，提供了一种易于使用的持续集成系统，使开发者从繁杂的集成中解脱出来，专注于更为重要的业务逻辑实现上。同时 Jenkins 能实施监控集成中存在的错误，提供详细的日志文件和提醒功能，还能用图表的形式形象地展示项目构建的趋势和稳定性。Jenkins 还提供了非常丰富的插件支持，这使得 Jenkins 变得越来越强大。我们可以方便的安装各种第三方插件，从而方便快捷的集成第三方的应用。

**基于Jenkins快速搭建CI环境**  
首先要知道一个持续集成环境需要包括三个方面要素：代码存储库、构建过程和持续集成服务器，​代码存储库一般使用Git:  
1、开始新建一个 Jenkins 项目， 由于我们需要连接 Git 的代码存储器， 我们选择 Build a free-style software project。  
2、然后配置这个 JenkinsTest 项目了，根据实际的 Git 服务器服务器信息配置 Source Code Management，这能让 Jenkins 知道如何从哪里获取最新的代码。  
3、根据开发需要，隔一段时间需要重新构建一次。选择 Build periodically，在 Schedule 中填写 0 * * * *对应的构建时间。  
4、添加 build 的步骤了。Jenkins 提供了四个选项供我们选择，可以根据需要执行或调用外部命令和脚本，例如ant、shell、maven等等。这些脚本都是根据需要自己配置的。  
5、可以在 Jenkins 中观察构建的进度和最终的状态——成功或者失败。太阳代表之前的构建没有任何失败，蓝色的小球代表构建成功。也可以在JenkinsTest 查看单次构建的 Console 的输出结果。从中能看到构建的第一步是从 Git 服务器上 pull 代码，然后在build。  

# 实际项目Jenkins搭建
### 一、安装jenkins
官网：[https://jenkins.io/download/](https://jenkins.io/download/), 稳定版本和周更版本，按需安装  
![](https://ws4.sinaimg.cn/large/006tNc79ly1fytmk1y4bpj30pt0l6acs.jpg)
![](https://ws4.sinaimg.cn/large/006tNc79ly1fytmnb3gtsj30h80c640g.jpg)

安装完毕，以项目在用jenkins为例，打开：[http://10.100.62.220:8080/](http://10.100.62.220:8080/)

### 二、创建任务
点击New任务，输入任务名，选择"构建一个自由风格的软件项目"，点击OK  
![](https://ws3.sinaimg.cn/large/006tNc79ly1fytk1xm2wdj30aa0bumy3.jpg)
![](https://ws1.sinaimg.cn/large/006tNc79ly1fytk4dzl2zj30qf0pzwij.jpg)

进入刚才创建的任务"24h_ Android _test"详情页，选择菜单"Configure"进入配置页面  
![](https://ws4.sinaimg.cn/large/006tNc79ly1fytk6wwas6j30j909ddh1.jpg)

### 三、源码管理
在 **Source Code Management** 下，选择Git作为源码管理工具，按照1、2、3顺序依次填写url、填写用户凭据、指定打包分支  
![](https://ws4.sinaimg.cn/large/006tNc79ly1fytkih68kyj30q70fm3zw.jpg)

### 四、参数化构建
在 **参数化构建** 下，分别创建：ENV、apiDebug、appVersionCode、appVersionName、appDescription五个可配置参数，参数具体作用如下图：  
![](https://ws3.sinaimg.cn/large/006tNc79ly1fytkpizp5rj30kh0a3q3j.jpg)
![](https://ws3.sinaimg.cn/large/006tNc79ly1fytkpyxglmj30kj07rt97.jpg)
![](https://ws2.sinaimg.cn/large/006tNc79ly1fytkqbrqttj30kj08rmxo.jpg)
![](https://ws2.sinaimg.cn/large/006tNc79ly1fytkqkco5lj30kg08tdgd.jpg)
![](https://ws1.sinaimg.cn/large/006tNc79ly1fytkqsd6dfj30kk08t74v.jpg)

### 五、Shell脚本
在 **Build** 下，选择 **Add build step**，**执行shell** 创建以下shell脚本    

1、替换指定Host文件
<pre>
cd ./env
sh ./env.sh $ENV
</pre>

2、在根目录生成24h.properties文件
<pre>
echo "apiDebug="$apiDebug"\nappVersionCode="$appVersionCode"\nappVersionName="$appVersionName>24h.properties
</pre>

3、将sdk.dir和api.debug写入根目录local.properties文件
<pre>
echo "sdk.dir=/Users/mac/Documents/tools/Android/sdk">local.properties
echo "api.debug=true">>local.properties
echo "12345678"|sudo -S ./gradlew clean build
</pre>

4、根据apiDebug设置蒲公英更新说明
<pre>
if [ $apiDebug = true ]
then
	appDescription="浙江24小时V"$appVersionName" "$ENV"环境"
elif [ -z $appDescription ]
then
	appDescription="浙江24小时V"$appVersionName"正式包"
fi
</pre>

5、上传蒲公英，需要在蒲公英申请API Key和User Key
<pre>
cd $WORKSPACE/app/build/outputs/apk/release
curl -F "file=@app-release.apk"  -F "updateDescription=$appDescription" -F "uKey=60cfcf0a08939473bc6f55bb8e84aac7
" -F "_api_key=284ff64e8da3000912795e3b7c18ff2a" http://www.pgyer.com/apiv1/app/upload
</pre>

6、如果需要打渠道包，请执行Walle多渠道打包脚本，渠道列表在项目根目录channel文件中配置  
> 渠道包位置：/Users/mac/.jenkins/workspace/24h_Android_release/app/build/outputs/channels/  

<pre>
echo "12345678"|sudo -S ./gradlew clean assembleReleaseChannels -PchannelFile=channel
</pre>