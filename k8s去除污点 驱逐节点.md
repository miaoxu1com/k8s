https://blog.csdn.net/agonie201218/article/details/120632984
https://www.cnblogs.com/cheyunhua/p/14474452.html

https://blog.csdn.net/lihongbao80/article/details/108075051?spm=1001.2101.3001.6650.4&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-4-108075051-blog-107739227.pc_relevant_aa&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7Edefault-4-108075051-blog-107739227.pc_relevant_aa&utm_relevant_index=9
添加污点可以不让k8s调度pod部署到节点上  --但是手动指定节点还是可以部署到污点节点上
驱逐节点可以优雅的实现pod的迁移
https://blog.csdn.net/yjk13703623757/article/details/107739227
节点不可调度
https://blog.csdn.net/weixin_43936969/article/details/106307385?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-1-106307385-blog-107739227.pc_relevant_aa&spm=1001.2101.3001.4242.2&utm_relevant_index=4
网络重启会影响到 节点的kube-proxy flannel damonset节点的ip分配  需要删除pod重启 重新获取新的ip  而且也会影响到服务  重启可以获取到新的ip
