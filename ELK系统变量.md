修改 /etc/sysctl.conf，在最后追加如下配置 vm.max_map_count = 655360
修改 /etc/security/limits.conf，增加如下配置
*       soft    memlock          unlimited
*         hard    memlock          unlimited
*         hard    nofile           65536
*         soft    nofile           65536


修改 /etc/security/limits.d/20-nproc.conf，增加如下配置
*          soft    nproc     4096
root       soft    nproc     unlimited
