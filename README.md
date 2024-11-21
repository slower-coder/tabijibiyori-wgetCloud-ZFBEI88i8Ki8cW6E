
nginx是由俄罗斯开发的一款http web服务器，我们经常用这款服务器做负载均衡和反向代理。今天我们就来聊聊Nginx作为反向代理时，如何进行路由配置。假设你已经部署好Nginx了，我们进入Nginx安装目录，进入nginx.conf文件。找到http节点下的server节点，值是一个json。在json中 有一个location的指令，就是代表转发。一般是这样的形式：




```
location {$path} {
        proxy_pass {$url};
}
```


{$path}代表匹配源url的部分，proxy\_pass 后的{$url}则是代表要转发的目标url，


这里一般会涉及到转发时是否携带原有路径的问题。举个例子：




```
location /abc {
        proxy_pass http://127.0.0.1:9090/;
}
```


如果我们请求 http://127\.0\.0\.1:80/abc （假设nginx的服务器的是80）


则请求的路径path是/abc，此时会匹配到该location指令的规则 /abc，则请求会转发的本机的9090端口。如果我们请求 https://github.com/jilodream/ )此时会匹配到该location指令的规则，则请求会转发的本机的9090端口。但是问题来了，转发9090端口时，/abc要不要补充到后边？后边的/cloud部分要不要追加？这里是和$url是否包含路径有关系，无斜杠就代表不包含路径，有斜杠就代表包含路径。


一、无路径场景


如果：$url为http://127\.0\.0\.1:9090 表示无路径此种情况，会将源url的路径部分直接追加举几个例子(1\)




```
location /abc {
        proxy_pass http://127.0.0.1:9090;
}
```


请求http://127\.0\.0\.1:80/abc/bcd


则跳转到http://127\.0\.0\.1:9090/abc/bcd(2\)




```
location /abc/bcd {
        proxy_pass http://127.0.0.1:9090;
}
```


请求http://127\.0\.0\.1:80/abc/bcd


则跳转到http://127\.0\.0\.1:9090/abc/bcd(3\)




```
location /abc/bcd/ {
        proxy_pass http://127.0.0.1:9090;
}
```


请求http://127\.0\.0\.1:80/abc/bcd/


则跳转到http://127\.0\.0\.1:9090/abc/bcd/


**总结就是一句话，proxy\_pass 后配置的目标url，如果没有路径信息（包括/），则会将源url的路径部分，直接追加到目标url中**


二、有路径场景


如果：$url为http://127\.0\.0\.1:9090/ 表示有路径如果：$url为http://127\.0\.0\.1:9090/gov 表示有路径如果：$url为http://127\.0\.0\.1:9090/gov/ 表示有路径此种情况，会将源url的路径部分去掉已匹配部分后，将剩余部分直接追加到目标url后，如图：


![](https://img2024.cnblogs.com/blog/704073/202411/704073-20241120172947050-1388844150.png)


 


举几个例子


(1\)




```
location /abc/ {
        proxy_pass http://127.0.0.1:9090/;
}
```


请求http://127\.0\.0\.1:80/abc/bcd


则跳转到http://127\.0\.0\.1:9090/bcd分析：源url的路径部分是： “/abc/bcd”与匹配规则“/abc/”匹配成功匹配后剩余部分是“bcd”“http://127\.0\.0\.1:9090/”追加“bcd”则最终会跳转到http://127\.0\.0\.1:9090/bcd(2\)




```
location /abc {
        proxy_pass http://127.0.0.1:9090/gov;
}
```


请求http://127\.0\.0\.1:80/abc/bcd


则跳转到http://127\.0\.0\.1:9090/gov/bcd(3\)




```
location /abc/b {
        proxy_pass http://127.0.0.1:9090/gov/;
}
```


请求http://127\.0\.0\.1:80/abc/bcd/


则跳转到http://127\.0\.0\.1:9090/gov/cd/(4\)




```
location /abc/b {
        proxy_pass http://127.0.0.1:9090/gov/;
}
```


请求http://127\.0\.0\.1:80/abc/b/cd/


则跳转到http://127\.0\.0\.1:9090/gov//cd/


**总结就是一句话，proxy\_pass 后配置的目标url，如果有路径信息（包括/），则会将源url的路径部分匹配后剩余的部分路径，直接追加到目标url中**


现在还有一个问题就是，(防盗连接：本文首发自https://github.com/jilodream/ )如果有多个匹配规则都命中的话，那么nginx会怎么处理呢？


如下：




```
location / {
        proxy_pass http://127.0.0.1:9091/gov/;
}
location /abc {
        proxy_pass http://127.0.0.1:9092/gov/;
}
location /abc/ai {
        proxy_pass http://127.0.0.1:9093/gov/;
}
```


请求http://127\.0\.0\.1:80/abc/ai/


则nginx 会按照最大匹配原则的情况，选择匹配对象，此时就会将请求转发至9093端口


 本博客参考[veee加速器](https://liuyunzhuge.com)。转载请注明出处！
