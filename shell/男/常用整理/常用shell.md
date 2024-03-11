```shell
# 磁盘名和容量
disk=$(df -h | grep /dev/nvme${i}n1 | awk -F " " '/mnt/nvme{print $1" "$5}')
# 当前时间
curtime=$(date "+%Y-%m-%d %H:%M:%S")

# 发送告警
function sendwarning()
{
curtime=$(date "+%Y-%m-%d %H:%M:%S")
hostname=`hostname`
processname=$1

text="告警时间：${curtime}\n告警主机：${hostname}\n告警名称：${processname}"
curl $Url \
   -H 'Content-Type: application/json' \
   -d '
   {
        "msgtype": "text",
        "text": {
                "content":
                "'"$text"'"
        }
   }'
}

# 获取进程数量(这个进程名要是唯一的才足够准确)
`ps -ef |grep -w ${process_name}|grep -v grep|wc -l`

# 配置自动挂载
sed -i '$a/dev/nvme'"${i}"'n1    /mnt/nvme'"${i}"'    ext4    defaults    0 0' /etc/fstab

# 获取ip
arg=$(ip addr | awk '/^[0-9]+: / {}; /inet.*global/ {print gensub(/(.*)\/(.*)/, "\\1", "g", $2)}' | grep 10.140.14) #grep 根据实际集群修改

# /var/spool/cron/root是crontab -e写入的文件。
```

