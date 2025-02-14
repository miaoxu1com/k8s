
云原生代表需要使用容器技术进行部署
automq
docker 监控容器的cpu、内存、网络、io情况
https://baiyp.ren/MinIO.html
Docker 搭建磁盘监控工具 Doku
https://www.amjun.com/1513.html
Docker监控容器资源的占用情况
docker top
Docker容器与主机监控
https://whathowhy.com/2021/06/19/docker-host-monitor/
Docker 容器监控系统初探
https://blog.csdn.net/qianshangding0708/article/details/121092017
https://cloud.tencent.com/developer/article/1699160



docker run -d 
-p 9000:9000 
-p 9001:9001 
--name minio 
-e "MINIO_ROOT_USER=autel" 
-e "MINIO_ROOT_PASSWORD=autel123" 
-v /root/minio/data1:/data1 
-v /root/minio/config:/root/.minio 
quay.io/minio/minio server /data1 
--address ":9000" --console-address ":9001" 

import io
import time

from minio import Minio
from minio.error import S3Error

from piltest.config.dbconfig import minio_config
from piltest.util.comm import *
from piltest.util.get_dat_frame import get_dat_frame
from piltest.util.post_sql_info import post_sql_info

client = Minio(
    **minio_config
)

conn = post_sql_info()
conn.connect()


def save_freq_to_mino():
    file_paths = get_file_paths(r"d:\test_data")
    for path in file_paths:
        file_path = path[0]
        file_md5 = get_file_md5(file_path)
        freq_head, freq_body = get_dat_frame(file_path)
        try:
            if not client.bucket_exists(file_md5):
                client.make_bucket(file_md5)
        except S3Error as err:
            logger.debug(f"存储桶不存在")
        else:
            res = None
            for i in range(len(freq_body)):
                res = client.put_object(
                    bucket_name=f"{file_md5}",
                    object_name=f"{i}",
                    data=io.BytesIO(freq_body[i]),
                    length=len(freq_body[i]),
                    content_type="application/octet-stream"
                )
            client.put_object(
                bucket_name=f"{file_md5}",
                object_name=f"{i+1}",
                data=io.BytesIO(freq_head),
                length=len(freq_head)
            )
            if res:
                table = "piltest_file_info"
                cols = {"flag": 1}
                condition = {"md5": file_md5}
                conn.update_data(table, cols, condition)
                logger.info(f"更新数据成功")


def query_minio_from_name(bucket_name):
    try:
        objects = client.list_objects(bucket_name, recursive=True)
        sorted_objects = sorted(objects, key=lambda obj: int(obj.object_name))
        return sorted_objects

    except S3Error as err:
        logger.debug(err)
