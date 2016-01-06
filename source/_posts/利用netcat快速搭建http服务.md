title: "利用netcat快速搭建http服务"
date: 2015-11-11 16:18:13
tags: linux
---

##netcat
netcat是linux系统自带的一个网络工具，通过，netcat的应用非常多，比如做代理，端口扫描，聊天服务器等，被誉为linux网络工具中的瑞士军刀，关于各种应用场景的介绍，可以参考下面参考1的链接。本文要分享的是netcat在工作中的一次应用：提供http服务。

##需求场景
场景是这样的，我在和其他部门联调测试，本质上就是对方把数据推过来了，我手动触发处理，然后再把结果推过去。每次联调都要沟通，相当麻烦，觉得应该想办法提高效率。其实联调中我的作用就下面三点：
1. 清空历史数据
2. 查看数据是否接受到
3. 调用数据处理程序，并把处理结果同步过去。

其实完全可以让对方登陆到我的测试机器，但是总觉得这样不妥，万一我那高雅的代码被人看到了呢～～。所以我希望别人能够在我的机器执行以上三个操作，在不登陆我的机器前提下。想起之前用nc做过代理服务，就试了下提供个http服务：

##处理逻辑
```
#step 1 读取请求信息
read request

while /bin/true; do
  read header
  [ "$header" == $'\r' ] && break;
done

#step 2 提取url
url="${request#GET }"
url="${url% HTTP/*}"

#step 3 解析命令，返回结果
echo -e "HTTP/1.1 200 OK\r"
echo -e "Content-Type: text/plain; charset=utf-8\r"
echo -e "\r"
case $url in
#接口1：清空数据
    '/clear')
        echo '开始清除历史数据...'
        rm ./out/*.txt
        echo '数据已清除!OK'
        ;;
#接口3：调用数据处理程序，并把处理结果同步过去
    '/exec')
        echo '执行排名统计并同步数据..'
        ./batch.sh
        if [[ $? -eq 0 ]];then
            echo '执行成功！OK'
        else
            echo '执行失败！error'
        fi
        ;;
#接口2：查看数据
    '/watch')
        i=0
        while read line
        do
            if [[ $line == 4* ]]; then
                i=$(($i+1))
                echo $line
            fi
        done < ./out/json_result.txt
        echo "共$i行"
        ;;
         *)
        echo '非法操作'
        ;;
esac
```
##服务驱动
nc监听端口，将接受到的数据丢给处理逻辑，处理逻辑处理完再交给nc，此处用双向管道实现循环管道
```
DPIPE="/tmp/static_server"
echo "删除双向管道${DPIPE}（如果存在的话）..."
rm -f $DPIPE
echo "创建双向管道【${DPIPE}】..."
mkfifo $DPIPE
echo "服务开始..."
while /bin/true; do
    cat $DPIPE | ./static_server.sh | nc -l 127.0.0.1 8080 > $DPIPE
done
```
非常的轻量级，开发这个接口加测试不到一个小时，python3的http.server也能很快实现这样的功能，有机会也试下。

[netcat应用场景介绍](http://www.oschina.net/translate/linux-netcat-command)

