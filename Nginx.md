## 目录
- `rpm -ql nginx` 列出所有 nginx 目录
- cd /etc/nginx


## 重启
- 查询服务的运行状况 ps aux | grep nginx
- systemctl start nginx.service
- systemctl stop nginx.service
- systemctl restart nginx.service
- 只修改配置文件重新加载即可，不用重启 nginx， nginx -s reload
- 查看端口号 `netstat -tlnp`

	在默认情况下，Nginx启动后会监听80端口，从而提供HTTP访问，如果80端口已经被占用则会启动失败。我么可以使用netstat -tlnp命令查看端口号的占用情

- 从容停止服务 nginx -s quit

## 权限
- 从下往上执行，注意配置顺序

## 解决访问403问题
更改文件目录后，发现访问网站出现 403

![](http://ww3.sinaimg.cn/large/006tNc79gy1g4icxt5j1cj30fj09i0td.jpg)
	
查看 /etc/nginx/nginx.conf，发现错误日志目录
	
![](http://ww2.sinaimg.cn/large/006tNc79gy1g4iczhp0sdj30fn0b6jsa.jpg)
	
再查看错误日志，发现是权限问题
	
![](http://ww4.sinaimg.cn/large/006tNc79gy1g4id0syamuj30tu090dip.jpg)
	
现进入部署目录查看权限，发现要 root 才能访问
	
![](http://ww3.sinaimg.cn/large/006tNc79gy1g4id2h61cfj30l108nwf6.jpg)
	
于是修改 nginx.conf，将原来的 `user nginx` 改成 `user root root`
	
![](http://ww1.sinaimg.cn/large/006tNc79gy1g4id4h37frj30ns09bmy1.jpg)

