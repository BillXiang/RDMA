# https://zhuanlan.zhihu.com/p/32105832
# 先看看你有几个网卡
$ ibdev2netdev
mlx5_0 port 1 ==> ens7f0 (Up)
mlx5_1 port 1 ==> ens7f1 (Up)
# 看看网卡支持的RoCE版本
# 记得给你的网卡绑定个IP, 两边能ping通
$ show_gids
DEV	PORT	INDEX	GID					IPv4  		VER	DEV
---	----	-----	---					------------  	---	---
mlx5_0	1	0	fe80:0000:0000:0000:268a:07ff:fe91:f810			v1	ens7f0
mlx5_0	1	1	fe80:0000:0000:0000:268a:07ff:fe91:f810			v2	ens7f0
mlx5_0	1	2	0000:0000:0000:0000:0000:ffff:c0a8:0101	192.168.1.1  	v1	ens7f0
mlx5_0	1	3	0000:0000:0000:0000:0000:ffff:c0a8:0101	192.168.1.1  	v2	ens7f0
mlx5_1	1	0	fe80:0000:0000:0000:268a:07ff:fe91:f811			v1	ens7f1
mlx5_1	1	1	fe80:0000:0000:0000:268a:07ff:fe91:f811			v2	ens7f1
mlx5_1	1	2	0000:0000:0000:0000:0000:ffff:c0a8:0102	192.168.1.2  	v1	ens7f1
mlx5_1	1	3	0000:0000:0000:0000:0000:ffff:c0a8:0102	192.168.1.2  	v2	ens7f1
n_gids_found=8
# 在一边服务器上启动收包测试, 用index3, RoCEv2
$ ib_send_bw -d mlx5_0 -x 3
# 另外一边发包
$ ib_send_bw -d mlx5_1 192.168.1.1 --report_gbits -F -x 

# 抓包
# 爱动脑筋的同学会问, 网卡做的封装, 两边还直连, 我怎么知道GID 3真的在传v2, 2真的用v1发?
# 新开一个窗口, 驱动已经帮你想好了, 就用大家最熟悉的tcpdump, 
$ ethtool --set-priv-flags ens7f0 sniffer on
$ tcpdump -i ens7f0 -w rdma.pcap
（我的测试环境是单机一块网卡两个口光纤自环）
