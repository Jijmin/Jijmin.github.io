1. 查看防火墙规则`iptables -L`
2. 临时将防火墙规则清空掉`iptables -F`
3. 查看 ESLinux 当前状态`getenforce`
4. 修改 ESLinux 的值为 Permissive`setrnforce 0`
5. 查看防火墙的各个状态值含义 `chkconfig --list iptables`
6. 关闭防火墙开机自启动`chkconfig iptables off`