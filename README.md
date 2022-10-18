## 同步上游去广告规则,合并去重

#### 文件说明:
```yaml
-- root
   |-- AdFilters.sh			#合并规则去重脚本,设置这个脚本定时任务即可.
   |-- AdFiltersUpdate.log	#更新规则四产生的日志,只会保留最新的一次日志.
   |-- AdList.txt			#上游规则列表,一行一条.
   |-- myhosts.txt			#手动设置的黑白名单规则,会自动合并到下方黑白名单.
   |-- allow.owhs.cn.txt	#白名单规则.
   |-- deny.owhs.cn.txt		#黑名单规则.
   |-- new.txt				#去重后的合并规则缓存,丢弃.
   `-- old.txt				#未去重的初始规则合计,丢弃.

核心文件只有2个,1个是去重脚本:AdFilters.sh,1个是规则地址:AdList.txt,
myhosts.txt根据需要自己创建,其他文件都不重要
```
#### 主要文件配置
```bash
尽量都使用绝对路径,避免权限和安全性问题

cat >~/AdList.txt<<'QAZ'
https://chfs.owhs.cn:58443/chfs/shared/webdav/myhosts.txt
https://file.trli.club:2087/github/Accelerate-Hosts.txt
https://file.trli.club:2087/ad-youtube-hosts/ad-youtube-adguardhome-dnstype.txt
https://file.trli.club:2087/dns-hosts/dns-adguardhome/whitelist_full.txt
https://file.trli.club:2087/dns-hosts/dns-adguardhome/blacklist_full.txt
https://file-git.trli.club/allow/Domains
https://www.trli.club/ad-hosts/ad-hosts-pro/ad-adguardhome-dnstype.txt
https://raw.githubusercontent.com/217heidai/adblockfilters/main/rules/adblockfilters.txt
https://raw.githubusercontent.com/Cats-Team/AdRules/main/adguard-full.txt
https://raw.githubusercontent.com/Cats-Team/AdRules/main/adguard.txt
https://raw.githubusercontent.com/DoingDog/XXKiller/main/w.txt
https://raw.githubusercontent.com/fordes123/adg-rule/main/rule/mylist.txt
https://raw.githubusercontent.com/fordes123/adg-rule/main/rule/adgh.txt

QAZ

cat >~/AdFilters.sh<<'WSX'
#!/bin/bash
#本脚本需要用到命令:cat, echo, wc, sed, sort, curl.如果没有这些命令, 需要先安装后才能执行, 需要先安装后才能执行,'~'的意思是执行脚本用户的家目录,比如/root
#清空上次处理的缓存文件
/usr/bin/echo "`date +%Y%m%d%H%m%S` 接下来会出现很多0,如果非0,则有问题,需要排查原因"
>~/old.txt
/usr/bin/echo $?
>~/new.txt
/usr/bin/echo $?
/usr/bin/echo "`date +%Y%m%d%H%m%S` 上次更新产生的缓存已经清空"

#下载各种规则合并到old.txt中
#如果在国外,可以手动指定代理方式拉取规则,比如:/usr/bin/curl -SLk--socks5 192.168.1.1:10808 https://xxx.xxx.xxx/xxx,curl的-s参数可以不显示下载过程,不建议使用,会无法判断下载是否完整
while read line
do
    /usr/bin/curl -SLk ${line} >>~/old.txt
    /usr/bin/echo $?
    /usr/bin/echo ${line}
done <~/AdList.txt
/usr/bin/echo "`date +%Y%m%d%H%m%S` 本次更新所需上游规则已缓存"

#删除注释和空行, 然后去重
/usr/bin/sed -i '/^!/d;/^#/d;/^\s*$/d;/^0\./d;/^127\./d' ~/old.txt
/usr/bin/echo $?
/usr/bin/sort -u ~/old.txt >>~/new.txt
/usr/bin/echo $?
/usr/bin/echo "`date +%Y%m%d%H%m%S` 本次更新缓存规则已去重"

#筛选白名单到allow.owhs.cn.txt, 并从原文件删除
/usr/bin/cat >~/allow.owhs.cn.txt<<RFV
! Title: AdBlock DNS Filters
! Description: 适用于AdGuard的去广告合并规则，每8个小时更新一次。合并的是别人根据上游合并后的规则, 所以重复规则非常多, 需要去重：217heidai/adblockfilters,Cats-Team/AdRules,Potterli20/hosts,DoingDog/XXKiller,fordes123/adg-rule
! Homepage: https://chfs.owhs.cn:58443/#/webdav
! Source: https://chfs.owhs.cn:58443/chfs/shared/webdav/allow.owhs.cn.txt
! Version: `date +%Y%m%d%H%m%S`
! Last modified: `date +%Y/%m/%d\ %H:%m:%S`
! unBlocked domains: `/usr/bin/sed -n '/^@/p' ~/new.txt |/usr/bin/wc -l`
RFV
/usr/bin/echo $?
/usr/bin/sed -n '/^@/p' ~/new.txt >>~/allow.owhs.cn.txt
/usr/bin/echo $?
/usr/bin/sed -i '/^@/d' ~/new.txt
/usr/bin/echo $?
/usr/bin/echo "`date +%Y%m%d%H%m%S` 本次更新的白名单已生成"

#剩下的都是黑名单和dnstype, 全部追加到deny.owhs.cn.txt文件
/usr/bin/cat >~/deny.owhs.cn.txt<<EDC
! Title: AdBlock DNS Filters
! Description: 适用于AdGuard的去广告合并规则，每8个小时更新一次。合并的是别人根据上游合并后的规则, 所以重复规则非常多, 需要去重：217heidai/adblockfilters,Cats-Team/AdRules,Potterli20/hosts,DoingDog/XXKiller,fordes123/adg-rule
! Homepage: https://chfs.owhs.cn:58443/#/webdav
! Source: https://chfs.owhs.cn:58443/chfs/shared/webdav/deny.owhs.cn.txt
! Version: `date +%Y%m%d%H%m%S`
! Last modified: `date +%Y/%m/%d\ %H:%m:%S`
! Blocked domains: $(/usr/bin/cat ~/new.txt|/usr/bin/wc -l)
EDC
/usr/bin/echo $?
/usr/bin/cat ~/new.txt >>~/deny.owhs.cn.txt
/usr/bin/echo $?
/usr/bin/echo "`date +%Y%m%d%H%m%S` 本次更新的黑名单已生成"
WSX
```
#### 在机器上设置计划任务
```bash
因为使用chfs共享,并不安全,直接设置定时任务,较为危险,所以复制一份脚本文件去主机环境执行,防止他人篡改.
cp ~/AdFilters.sh /etc/AdguardCache/config/AdFilters.sh

crontab -e
55 11,23 * * * bash /etc/AdguardCache/config/AdFilters.sh >~/AdFiltersUpdate.log
```
![1658483990606.png](https://img.owhs.cn:58443/i/2022/07/22/62da7516aa0d3.png)
#### 去ADGuardHome页面订阅规则
[白名单规则,右键复制链接](https://chfs.owhs.cn:58443/chfs/shared/webdav/allow.owhs.cn.txt "白名单")
![1658484087798.png](https://img.owhs.cn:58443/i/2022/07/22/62da7577dc222.png)
[黑名单规则,右键复制链接](https://chfs.owhs.cn:58443/chfs/shared/webdav/deny.owhs.cn.txt "黑名单")
![1658484121193.png](https://img.owhs.cn:58443/i/2022/07/22/62da75993d6bc.png)
###### 规则并不是越多越好,每个列表添加一个即可,而且我的规则全是汇总上游的汇总规则,去重后的结果,如果性能足够,直接订阅一个规则即可.

### 阿里云图文教程:如何使用dns
**先看这个教程**:[DoT/DoH接入方法](https://help.aliyun.com/document_detail/176821.html "DoT/DoH接入方法")

<video src="https://chfs.owhs.cn:58443/chfs/shared/webdav/iOS-Set-Dns.mp4" controls="controls" width="592" height="1280">iOS使用加密DNS视频教程</video>

<iframe src="https://ip.skk.moe/simple" style="width: 100%; border: 0"></iframe>

<details>
<summary><code><strong>赞助/sponsorship</strong></code></summary>
<img width=30% src="https://img.owhs.cn:58443/i/2022/10/18/634debb1cc1a4.jpg" />
<img width=30% src="https://img.owhs.cn:58443/i/2022/10/18/634debb12677e.png" />
<img width=30% src="https://img.owhs.cn:58443/i/2022/07/15/62d11c2a6ab50.png" /><br/>
</details>
