# jenkins环境搭建

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