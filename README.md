# 怎么发布一个docker image到cf应用


先来过一下[CF文档](https://docs.cloudfoundry.org/devguide/deploy-apps/push-docker.html#cf-ssh)

此处有2个关键信息需要我们注意

1.  cf应用只会监听一个端口
2.  cf应用会自动监听在dockerfile里EXPOSE的端口。根据测试，当EXPOSE了多个端口时，cf会默认监听第一个端口

比如这个[Kong的Dockerfile](https://github.com/Kong/docker-kong/blob/9a6d1a06b2e768949fda9ae7b30b747437fe388c/alpine/Dockerfile)暴露了4个端口，8000 8443 8001 8444，这个dockerfile被发布成cf应用时，只会监听8000端口，然后我们需要的是api端口8001。所以我们需要改写这个Dockerfile。

修改前
```
EXPOSE 8000 8443 8001 8444
```
修改后
```
EXPOSE 8001
```

重新build后发布成cf应用即可。

细心的朋友可能会有一个问题，Kong的配置里http监听8001端口，https监听8443端口，bluemix默认都是https为什么此处用了8001。

根据测试cf应用本身是使用http协议的，在应用外会包装nginx网关，网关使用https协议，网关接到请求后使用http协议转发到cf应用，所以此处需要使用http协议的8001端口
