```config
bindaddress 0.0.0.0
bindaddress ::
driftfile /var/lib/chrony/drift
logdir /var/log/chrony
makestep 1.0 3
rtcsync
minsources 3
local stratum 10
ratelimit interval 1 burst 16
server ntp.tencent.com iburst minpoll 6 maxpoll 24
server ntp1.tencent.com iburst minpoll 6 maxpoll 24
server ntp2.tencent.com iburst minpoll 6 maxpoll 24
server ntp3.tencent.com iburst minpoll 6 maxpoll 24
server ntp4.tencent.com iburst minpoll 6 maxpoll 24
server ntp5.tencent.com iburst minpoll 6 maxpoll 24
server ntp.aliyun.com iburst minpoll 6 maxpoll 24
server ntp1.aliyun.com iburst minpoll 6 maxpoll 24
server ntp2.aliyun.com iburst minpoll 6 maxpoll 24
server ntp3.aliyun.com iburst minpoll 6 maxpoll 24
server ntp4.aliyun.com iburst minpoll 6 maxpoll 24
server ntp5.aliyun.com iburst minpoll 6 maxpoll 24
server ntp6.aliyun.com iburst minpoll 6 maxpoll 24
server ntp7.aliyun.com iburst minpoll 6 maxpoll 24
pool 0.pool.ntp.org iburst minpoll 6 maxpoll 24 maxsources 3
pool 1.pool.ntp.org iburst minpoll 6 maxpoll 24 maxsources 3
pool 2.pool.ntp.org iburst minpoll 6 maxpoll 24 maxsources 3
pool 3.pool.ntp.org iburst minpoll 6 maxpoll 24 maxsources 3
allow 0.0.0.0/0
allow ::/0