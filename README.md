nginx-rtmp-win32
================
NMS分支为功能加强版，如需使用原版，请切换master分支。新版本NMS不再使用visual c++编译，改为MSYS2 & MinGW-W64编译。

* Nginx: 1.13.3  
* Nginx-Rtmp-Module: 1.2.0  
* openssl-1.0.2l  
* pcre-8.41  
* zlib-1.2.11  
* ffmpeg-3.3.2  

## configure arguments
```
nginx version: nginx/1.13.3
built by gcc 7.1.0 (Rev2, Built by MSYS2 project)
built with OpenSSL 1.0.2l  25 May 2017
TLS SNI support enabled
configure arguments: --with-cc=gcc --builddir=objs_x86 --prefix= --sbin-path=nginx.exe --http-client-body-temp-path=temp/client_body_temp --http-proxy-temp-path=temp/proxy_temp --http-fastcgi-temp-path=temp/fastcgi_temp --http-scgi-temp-path=temp/scgi_temp --http-uwsgi-temp-path=temp/uwsgi_temp --with-cc-opt=-DFD_SETSIZE=4096 --with-select_module --with-http_v2_module --with-http_realip_module --with-http_addition_module --with-http_sub_module --with-http_dav_module --with-http_stub_status_module --with-http_flv_module --with-http_mp4_module --with-http_gunzip_module --with-http_gzip_static_module --with-http_auth_request_module --with-http_random_index_module --with-http_secure_link_module --with-http_slice_module --with-mail --with-stream --with-http_ssl_module --with-mail_ssl_module --with-stream_ssl_module --add-module=modules/nginx-rtmp-module
```

## 使用方法
双击nginx.exe

## 简要说明
conf/nginx.conf 为配置文件实例  
RTMP监听 1935 端口，启用live 和hls 两个application  
HTTP监听 8080 端口，
* :8080/stat 查看stream状态  
* :8080/index.html 为一个直播播放与直播发布测试器
* :8080/vod.html 为一个支持RTMP和HLS点播的测试器

## H.265
RTMP支持ID为12的H.265直播流

## 低延迟实时转码
原版Rtmp-Module不支持Windows下exec调用，NMS版嵌入ffmpeg，直接传入ffmpeg指令进行转码
* 支持使用$app,$name变量
* 转码后可以输出到本app
* 支持多路输出
* 支持qsv,cuvid硬解码(cuvid硬解码没有会话数限制)
* 支持qsv,nvenc硬编码(nvenc硬编码会话数受限于显卡型号，一般GeForce家用显卡最多支持2路)
* 支持转码为H.265 RTMP流，需客户端支持,ID为12

```
 application live {
    live on;
    
    transcode "-i rtmp://127.0.0.1/$app/$name -c:a libfdk_aac -profile:a aac_he -ab 32k -c:v copy -f flv rtmp://127.0.0.1/$app/$name_aac";
}
```


## 播放防盗链与推流鉴权
### 加密 URL 构成:
>rtmp://域名/业务名/流名?sign=失效时间戳-HashValue  

1.推流与播放地址:  
> rtmp://192.168.0.10/live/stream123

2.链接失效时间:2017/3/23 10:10:0 计算出来的失效时间戳为  
>1490235000

3.nginx.conf配置key  
>nodemedia2017privatekey

4.组合为HashValue  
>HashValue = md5("/live/stream123-1490235000-nodemedia2017privatekey”)   
>HashValue = d03af0812548d315279936ad76f912be

5.最终请求地址  
>rtmp://192.168.0.10/live/stream123?sign=1490235000-d03af0812548d315279936ad76f912be  
>sign关键字不能修改  

## nginx.conf 鉴权配置说明
```
 application live {
     live on;
     live_auth on;  #鉴权开关
     live_auth_secret nodemedia2017privatekey; #鉴权KEY
}
```
## 安全URL的产生  
应该由业务服务器生成安全的URL,防止在客户端泄漏key.可参考auth_gen.php

## 后续版本或将增加
* static_transcode 可用于输入RTSP摄像头数据流转RTMP流
* flv文件录制后调用内置ffmpeg转换为MP4文件格式并添加faststart标识
* 精简ffmpeg编解码器，减小程序体积 
* NMS的Linux版
* 64位版，经测试视频编码器性能更强
* 你提

## 注意
不支持exec

## 直播测试工具 
内置了一个方便测试的web端推流与播放的工具，Flex开发  
![img](https://github.com/NodeMedia/NodeMediaDevClient/raw/master/QQ20160310-0.png)  
源码在此:https://github.com/NodeMedia/NodeMediaDevClient  

## Flash推流插件
仅5K大小的flash推流插件，ActionScript3开发
https://github.com/NodeMedia/NodeMediaClient-Web  
可直接嵌入web项目使用
