小米路由VPN智能分流配置
===

通过VPN上网是大家都懂的。如果通过VPN访问大陆的网站将会变得很慢，小米路由的智能VPN分流可以按服务地址来选择哪些流量是从VPN出去的，其余的就按正常方式访问。要做到全家智能上网，你需要的是一个VPN账号，还有就是一个需要分流到VPN的地址（域名）清单了。

## 使用方法

proxy.txt 包含了一些需要分流的常用地址。你可以根据自己的需要添加一些自己需要的域名。

### 批量倒入

开启小米路由的开发者模式，通过ssh访问小米路由。开启方法在小米官方有教程。把proxy.txt文件放在路由器的 /etc/smartvpn目录下，然后在路由器Web管理界面的“高级设置/VPN”中就可以看到这个清单了。重新启动VPN就可以用了。

### 手工录入

不懂的ssh操作的朋友，可以小米路由的VPN界面可以把 proxy.txt 中的域名和ip一个一个地录入进去。域名最前面的"."不需要录入，地址就按原来的样子录入就可以了。由于域名很多，手工输入还是很费劲的，希望网友能够总结出一个能够平顺访问海外主流网站的最小域名清单出来。

由于清单中有1000多个个网址，全部手工输入其实是不现实的，可以挑选一些自己经常会访问的网站输入进去。以后发现哪个网站访问不了，再加上去就是了。由于网站有很多图片和视频都是放到不同的域名之下的，要让一个网站能够平顺访问，可能需要添加好几个域名。希望有热心的git友能够整理出一个20网址的精简版，能够让几个主流的海外网站正常访问，以便让不懂技术的普通人能够很快地手工输入。



## VPN智能分流存在问题

启动VPN智能分流后，小米路由的dnsmasq服务会把VPN连接的DNS服务器加入到父DNS清单中，与广域网获得DNS一起承担域名解析任务。这其实是不合理的，因为这会导致智能分流中不需要走VPN的域名会使用VPN的DNS来解析域名，有可能获得域名的海外ip（而不是国内ip），从而导致=网站访问变得很慢。

解决的方案是修改智能VPN的启动脚本，把海外DNS从配置中剔除。具体做法如下：

* 修改文件之前需要把磁盘挂载城可读写

```
mount -o remount, rw /   
```

* 修改 /usr/sbin/smartvpn.sh文件中的 dnsmasq_restart()函数，在函数中的 /etc/init.d/dnsmasq restart 命令前插入以下语句：

    # remove DNS entry pushed by Softether
    sed -i -e '/nameserver 8.8.8.8/d' /tmp/resolv.conf.auto

* 说明一下：上面的8.8.8.8为小米路由智能分流指定的DNS服务器。VPN连接应该把该地址作为DNS推送过来，否则小米路由会通过国内的路由访问收到污染的8.8.8.8服务器，导致无法正常工作。
* 修改完成后需要回复磁盘为只读

```
mount -o remount, ro /
```



---
如果有有些常用的主流网站没有包括进去，欢迎git友帮忙添加，造福人类。
清单中有很多不常用的网址，有些甚至已经关闭的，欢迎git友对清单进行精简。
