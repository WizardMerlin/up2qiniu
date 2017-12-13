# up2qiniu
适用于macOs的上传小工具，给需要的人方便。

> 需求：我给你一张图片，你返回给我七牛云的具体链接，最好是markdown格式的。

> 实现:
理论上而言，可以查看官方的对象存储库，然后用熟悉的语言，调用相关的API即可。
但是官方已经提供了`qshell`工具用来上传文件夹和文件, 此时就很容易了，用shell封装一下，调试看看就知道了。

> 总结： 在qshell上，做二次开发(拿shell包装一下)试试看能不能满足要求，如果不能再老老实实去看API文档写调用.

## 详细过程

### 测试 qshell 工具
> 到网站下载工具，然后扔到 Path 目录可以指定的地方，改名为qshell.

![](http://omotkhw3y.bkt.clouddn.com/qshell1-test.jpg)

> 查看官方文档，以及qshell的命令行帮助，发现 [fput](https://github.com/qiniu/qshell/blob/master/docs/fput.md)子命令就满足要求了。
* 先要鉴定一下 ak 和 sk：`qshell account <Your AccessKey> <Your SecretKey>`
* 上传单个文件试试看：`qshell fput <Bucket> <Key> <LocalFile> [Overwrite] [MimeType] [UpHost] [FileType]`
![](http://omotkhw3y.bkt.clouddn.com/qshell1-test2.jpg)



### 查看网络前缀
> 网络前缀： `http://omotkhw3y.bkt.clouddn.com`, 这个信息加上上传上去的图片文件名，则可以完成目的


### 代码实现
> 上面已经测试的很清楚了，只要在上传的时候给文件命名加上日期，并且让脚本返回相应的格式即可。
存储格式： `http://omotkhw3y.bkt.clouddn.com/日期-文件名`

代码如下：(直接在 bashrc 里面添加即可)
```bash
## qshell upload related for macOS
### qshell account required in advance

function upload()
{

    ## check file
    if [ -z "$1" ]
        then 
            echo "upload file null, plz tell me which file u wanna upload \n"
            return
    fi

    LOCAL_FILE=`basename $1`

    if [ ! -f $LOCAL_FILE ] 
        then
            echo "file do not exist.\n"
            return
    fi

    ## now we make sure the file exists.
    CUR_TIME=`date  +%Y-%m-%d`
    UP_FILE="$CUR_TIME"-"$LOCAL_FILE"

    BUCKET="merlin-blog"
    URL_PREFIX="http://omotkhw3y.bkt.clouddn.com/" 
    FINAL_LINK="![](${URL_PREFIX}${UP_FILE})"


    ## using qshell upload now!
    qshell fput $BUCKET $UP_FILE ./$LOCAL_FILE 

    if [ 0 -eq $? ]
        then    
            echo ${FINAL_LINK} | pbcopy
            echo "\nSuccess, pic link has sent to ur clipboard.\n"
    else
        echo "\nupload Falied.\n"
    fi

    ## FINAL_LINK for example: http://omotkhw3y.bkt.clouddn.com/2017-12-12-jsxsx.jpg
}
alias up='upload'
```

> 期间用到了技巧：
* `date  "+%Y-%m-%d"`,  输出：2017-12-12
* alias 别名
* `$?`返回上一命令执行结果

btw: 一定要先认证一下 `qshell account ak ck` .


### 最终效果
直接看图好了
![](http://omotkhw3y.bkt.clouddn.com/2017-12-13-up_final.jpg)

## 总结

> 适用于 macOs 的小工具，基本就是根据七牛云提供的 qshell 工具进行了简单的二次__封装定制__，技术含量也不高。纯当对 shell 的练手了， 前后花了1个半小时，包括查资料。
