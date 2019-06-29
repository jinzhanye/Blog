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