该监控基于iostat,然后iostat 命令用来监视系统输入/输出设备负载
### 1.安装IOSTAT
yum install sysstat
测试iostat 查看所有硬盘io
```
[root@ZQLC-1 zabbix-iostat-master]# iostat 
Linux 2.6.32-696.1.1.el6.x86_64 (ZQLC-1)        2017年04月25日  _x86_64_        (16 CPU)

avg-cpu:  %user   %nice %system %iowait  %steal   %idle
           0.04    0.00    0.04    0.13    0.00   99.79

Device:            tps   Blk_read/s   Blk_wrtn/s   Blk_read   Blk_wrtn
sda               8.82         0.34       384.29     315294  360110112
dm-0             47.99         0.32       383.80     303970  359655464
dm-1              0.00         0.00         0.00       2400          0
dm-2              0.06         0.00         0.49       2250     454576
```
### 2.部署脚本
```
git clone https://github.com/z-engine/zabbix-iostat.git
# Move the main script to one of PATH dir
mv zabbix-iostat/zabbix-iostat.sh /usr/local/bin/
chmod +x /usr/local/bin/zabbix-iostat.sh
# Configure zabbix_agentd with iostat UserParameter
echo 'UserParameter=iostat[*],/usr/local/bin/zabbix-iostat.sh "$1" "$2"' > /etc/zabbix/zabbix_agentd.d/iostat.conf
# or
# mv zabbix-iostat/iostat.conf /etc/zabbix/zabbix_agentd.d/iostat.conf

# Delete cloned repo
rm -rf zabbix-iostat
# Restart agent
systemctl restart zabbix-agentd
# Install sysstat
apt-get update
apt-get install sysstat
# Test discovery
zabbix_agentd -t iostat[discovery]
```
### 3 加入crontab
crontab -e
```
* * * * * ( sleep 10 && iostat -dxk 1 20 > /tmp/iostat.tmp && mv /tmp/iostat.tmp /tmp/iostat.log )
* * * * * ( sleep 40 && iostat -dxk 1 20 > /tmp/iostat.tmp && mv /tmp/iostat.tmp /tmp/iostat.log )

```
测试
```
[root@ZQLC-1 zabbix-iostat-master]# zabbix_agentd -t iostat[sda,rkb/s]
iostat[sda,rkb/s]                             [t|0.0085]
```
### 4 链接模板（林妹妹翻译）
监控项如图
![](leanote://file/getImage?fileId=58fee20a3210052f10000000)

