选择将一个文件存储在一个单独的存储桶中还是将多个文件存储在一个共享的存储桶中，取决于您的具体需求和使用情况。
单独一个文件一个存储桶的优点包括：
管理简单：每个文件都有自己的存储桶，可以更轻松地管理和组织文件。
安全性：每个文件都可以有独立的访问权限和加密设置，提供更高的安全性。
隔离性：每个文件都是独立的实体，不会受到其他文件的影响。

多个文件共享一个存储桶的优点包括：
简化管理：多个文件共享一个存储桶可以简化管理和维护任务，减少存储桶的数量。
节省成本：存储桶的创建和管理可能会产生一些开销，共享一个存储桶可以节省这些成本。
方便访问：多个文件共享一个存储桶可以更方便地访问和处理这些文件。
因此，您可以根据您的具体需求和使用情况来选择适合您的方式。如果您的文件之间没有太多关联性，
或者需要更高的安全性和隔离性，那么将每个文件存储在单独的存储桶中可能更合适。
如果您的文件之间有关联性，或者需要简化管理和节省成本，那么将多个文件共享一个存储桶可能更合适。

有一个问题是内存开销  还是读写开销  bucket存在再次写入会更新已存在的bucket数据

Bucket names must be between 3 (min) and 63 (max) characters long.
Bucket names can consist only of lowercase letters, numbers, dots (.), and hyphens (-).
Bucket names must not contain two adjacent periods, or a period adjacent to a hyphen.
Bucket names must not be formatted as an IP address (for example, 192.168.5.4).
Bucket names must not start with the prefix xn--.
Bucket names must not end with the suffix -s3alias. This suffix is reserved for access point alias names.
Bucket names must be unique within a partition.


参考文章：
https://blog.csdn.net/networken/article/details/111500745
https://baiyp.ren/MinIO.html
https://blog.51cto.com/gblfy/5653234
