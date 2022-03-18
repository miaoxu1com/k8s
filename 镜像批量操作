1.打包镜像源
docker images |grep '6.1.0.0'|grep 'cmn_test/'|awk '{print "docker save"" " $1 "|gzip >" substr($1,44,length($1))".tar.gz"}'|sh
2.上传到worker节点，导入镜像

3.修改镜像标签为目标仓库地址
docker images|grep "6.1.0.0"|grep "repository.cmn.sundray.cn"|sed 's/repository.cmn.sundray.cn:80/192.168.0.252/g'|awk '{print "docker tag"" "$3 " "$1":"$2}'

4.登录仓库

5.推送镜像到节点
docker images|grep 192.168.0.252|grep "cmn1.6"|grep -v "cloud/webgo"|awk '{print $1}'| xargs -I {} docker push {}
