---
title: Istio--安装部署
date: 2025-03-08 19:08:06
---
<h2 id="istio-简介"><font style="background-color:rgba(255, 255, 255, 0);">Istio 简介</font></h2>
<font style="background-color:rgba(255, 255, 255, 0);">Connect, secure, control, and observe services.</font>

<font style="background-color:rgba(255, 255, 255, 0);">连接、安全加固、控制和观察服务的开放平台。</font>

+ <font style="background-color:rgba(255, 255, 255, 0);">连接（Connect）：智能控制服务之间的调用流量，能够实现灰度升级、AB 测试和红黑部署等功能；</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">安全加固（Secure）：自动为服务之间的调用提供认证、授权和加密；</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">控制（Control）：应用用户定义的 policy，保证资源在消费者中公平分配；</font>
+ <font style="background-color:rgba(255, 255, 255, 0);">观察（Observe）：查看服务运行期间的各种数据，比如日志、监控和 tracing，了解服务的运行情况。</font>

<h2 id="service-mesh"><font style="background-color:rgba(255, 255, 255, 0);">Service Mesh</font></h2>
`<font style="background-color:rgba(255, 255, 255, 0);">Service Mesh</font>`<font style="background-color:rgba(255, 255, 255, 0);">(服务网格)可以简单理解为</font>**<font style="background-color:rgba(255, 255, 255, 0);">"分布式代理"</font>**<font style="background-color:rgba(255, 255, 255, 0);">.</font>

![](https://cdn.nlark.com/yuque/0/2025/gif/43141749/1741444376203-46e720d6-3fb5-4672-b8ad-01e6eedd677f.gif)

<h2 id="istio-架构"><font style="background-color:rgba(255, 255, 255, 0);">Istio 架构</font></h2>
![](https://cdn.nlark.com/yuque/0/2025/svg/43141749/1741444377049-daa84c0b-0834-4f07-9510-5202b4a8617a.svg)

<h2 id="istio-安装部署"><font style="background-color:rgba(255, 255, 255, 0);">Istio 安装部署</font></h2>
<h3 id="使用istioctl安装"><font style="background-color:rgba(255, 255, 255, 0);">使用</font>`<font style="background-color:rgba(255, 255, 255, 0);">istioctl</font>`<font style="background-color:rgba(255, 255, 255, 0);">安装</font></h3>
<font style="background-color:rgba(255, 255, 255, 0);">官方详细中文安装文档:</font><font style="background-color:rgba(255, 255, 255, 0);"> </font>[<font style="background-color:rgba(255, 255, 255, 0);">https://istio.io/latest/zh/docs/setup/install/istioctl/</font>](https://istio.io/latest/zh/docs/setup/install/istioctl/)

<font style="background-color:rgba(255, 255, 255, 0);">以下只记录相关命令:</font>

```bash
$ curl -L https://istio.io/downloadIstio | sh -
# 也可以从官方github仓库进行获取release包, https://github.com/istio/istio/releases/tag/1.7.3

$ cd istio-1.7.3/
# 输出环境变量, 以便直接使用
$ export PATH=$PWD/bin:$PATH
# 添加自动补全功能(需要子命令时按下TAB键激活)
$ cp ./tools/istioctl.bash ~ && source ~/istioctl.bash
# 安装demo配置
$ istioctl manifest install --set profile=demo
# 为了验证是否安装成功，需要先确保以下 Kubernetes 服务正确部署，然后验证除 jaeger-agent 服务外的其他服务，是否均有正确的 CLUSTER-IP：
$ kubectl get svc -n istio-system
# 请确保关联的 Kubernetes pod 已经部署，并且 STATUS 为 Running
$ kubectl get pods -n istio-system

# 卸载
$ istioctl manifest generate --set profile=demo | kubectl delete -f -
```

<h3 id="使用helm-chart安装"><font style="background-color:rgba(255, 255, 255, 0);">使用</font>`<font style="background-color:rgba(255, 255, 255, 0);">helm chart</font>`<font style="background-color:rgba(255, 255, 255, 0);">安装</font></h3>
<font style="background-color:rgba(255, 255, 255, 0);">已被启用, 推荐使用</font>`<font style="background-color:rgba(255, 255, 255, 0);">istioctl</font>`<font style="background-color:rgba(255, 255, 255, 0);">安装.</font>

<font style="background-color:rgba(255, 255, 255, 0);">部分内容筛选自:</font><font style="background-color:rgba(255, 255, 255, 0);"> </font>[<font style="background-color:rgba(255, 255, 255, 0);">Istio 是啥?一文带你彻底了解</font>](https://weixin.sogou.com/link?url=dn9a_-gY295K0Rci_xozVXfdMkSQTLW6cwJThYulHEtVjXrGTiVgS8MgskBbNKMg1jd_UAgBBtBgbuW-CnWsI1qXa8Fplpd9M2HsHPtL_uGFwcB-LoRjg8V_MfAw8Wg3k70j_V31ZLuJMytP8qR2YRnycg--9VFPkkNS1gPt1QyTqAvLDpSkyT_ezw95tL17tyKO1qlVHvGS8DMVBk6NX30KCpE-80kRTqGhvjHCpURaK6ytIf8OgoTKJs_5u3vtMjWlhLQ8AEgCYioxHkzTmA..&type=2&query=istio&token=empty&k=91&h=L)

<h2 id="参考链接"><font style="background-color:rgba(255, 255, 255, 0);">参考链接</font></h2>
+ <font style="background-color:rgba(255, 255, 255, 0);">Istio Documentation:</font><font style="background-color:rgba(255, 255, 255, 0);"> </font>[<font style="background-color:rgba(255, 255, 255, 0);">https://istio.io/latest/docs/</font>](https://istio.io/latest/docs/)
+ <font style="background-color:rgba(255, 255, 255, 0);">Istio Arch:</font><font style="background-color:rgba(255, 255, 255, 0);"> </font>[<font style="background-color:rgba(255, 255, 255, 0);">https://istio.io/latest/docs/ops/deployment/architecture/</font>](https://istio.io/latest/docs/ops/deployment/architecture/)
+ <font style="background-color:rgba(255, 255, 255, 0);">Istio 入门:</font><font style="background-color:rgba(255, 255, 255, 0);"> </font>[<font style="background-color:rgba(255, 255, 255, 0);">Istio 是啥?一文带你彻底了解</font>](https://weixin.sogou.com/link?url=dn9a_-gY295K0Rci_xozVXfdMkSQTLW6cwJThYulHEtVjXrGTiVgS8MgskBbNKMg1jd_UAgBBtBgbuW-CnWsI1qXa8Fplpd9M2HsHPtL_uGFwcB-LoRjg8V_MfAw8Wg3k70j_V31ZLuJMytP8qR2YRnycg--9VFPkkNS1gPt1QyTqAvLDpSkyT_ezw95tL17tyKO1qlVHvGS8DMVBk6NX30KCpE-80kRTqGhvjHCpURaK6ytIf8OgoTKJs_5u3vtMjWlhLQ8AEgCYioxHkzTmA..&type=2&query=istio&token=empty&k=91&h=L)
+ <font style="background-color:rgba(255, 255, 255, 0);">Istio Installation: </font>[<font style="background-color:rgba(255, 255, 255, 0);">https://istio.io/latest/zh/docs/setup/getting-started/</font>](https://istio.io/latest/zh/docs/setup/getting-started/)

