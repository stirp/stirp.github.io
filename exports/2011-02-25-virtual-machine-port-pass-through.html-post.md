打开Vmware，点击Edit，virtual Network Editer.

默认选择的NAT网络，实际上这种网络最好，和主机互不影响。但是目前想要共享的端口比如说80网页浏览，23telnet，22SSH。本文写之前就是我为了在机房可以远程SSH到Ubuntu虚拟机才研究的。

选注默认的VMNET8，专用来NAT的虚拟网卡，点击NAT Settings，中间有个大大的add按钮，然后根据虚拟机的实际IP和所需端口开放，这样你的虚拟机就对外网开放啦！
