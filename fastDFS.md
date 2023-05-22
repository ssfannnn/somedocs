### 搭建FastDFS集成到Java当中

#### 1.安装依赖

```sh
yum install gcc libevent libenent-devel -y
```



#### 2.下载依赖包

如下：

![image-20230511152255580](fastDFS.assets/image-20230511152255580.png)

自己去官网下载即可

下载到自己本地以后讲依赖包放到Linux服务器中

我这里选择路径/usr/local/soft

如果没有soft包，需要先新建包

```shell
mkdir soft
```



#### 3.解压缩依赖包

``` shell
tar zxvf 依赖包名称
```

![image-20230511152506655](fastDFS.assets/image-20230511152506655.png)

解压缩以后如上图

现在需要执行解压缩以后的包中的脚本

```shell
cd FastDFS
```

![image-20230511152614332](fastDFS.assets/image-20230511152614332.png)

执行命令

```shell
./make.sh
```

执行完毕以后下载

```shell
./make.sh install
```

 对于libfastcommon来说也是上述步骤，这里不多赘述



执行完毕后，打开/usr/bin，会发现很多fdfs开头的命令

![image-20230511152745016](fastDFS.assets/image-20230511152745016.png)



#### 4.拷贝两个脚本

进入FDFS包中的conf包，将http.conf和mime.types拷贝到/etc/fdfs/中

```shell
cp http.conf /etc/fdfs/
cp mime.types /etc/fdfs/
```



#### 5.启动前的配置

编辑tracker.conf.sample

```shell
vi tranker.conf.sample
```

![image-20230511154637604](fastDFS.assets/image-20230511154637604.png)

将base_path改成自己服务器当中存在的一个路径



编辑storage.conf.sample

```
vi stprage.conf.sample
```

修改配置

![image-20230511155328093](fastDFS.assets/image-20230511155328093.png)

![image-20230511155447752](fastDFS.assets/image-20230511155447752.png)



**注意如果没有上面创建的目录，需要先创建目录**



#### 6.启动

- 启动、关闭、重启tracker server

```shell
//语法：fdfs_trackerd 配置文件路径 start|stop|restart，如果不传参数，则默认是start启动

//启动
fdfs_trackerd /etc/fdfs/tracker.conf start
//停止
fdfs_trackerd /etc/fdfs/tracker.conf stop
//重启
fdfs_trackerd /etc/fdfs/tracker.conf restart

```

- 启动、关闭、重启storage server

```shell
//语法：fdfs_trackerd 配置文件路径 start|stop|restart，如果不传参数，则默认是start启动

//启动
fdfs_trackerd /etc/fdfs/tracker.conf start
//停止
fdfs_trackerd /etc/fdfs/tracker.conf stop
//重启
fdfs_trackerd /etc/fdfs/tracker.conf restart
```





查看是否启动服务成功

```shell
ps -ef | grep fdfs
```

![image-20230511162508789](fastDFS.assets/image-20230511162508789.png)

如上图所示 说明启动成功





#### 7.测试上传

修改 **base_path** 和 **tracker_server** 



![image-20230511165541370](fastDFS.assets/image-20230511165541370.png)



在根目录下创建一个文件 test.txt

然后执行命令

```shell
语法：fdfs_test 配置文件位置 参数 upload, download, getmeta, setmeta, delete and query_servers

//上传
fdfs_test /etc/fdfs/client.conf upload 要上传的文件路径 （这里是test.txt）
```





日志

```shell
[2023-05-11 17:05:55] DEBUG - base_path=/opt/fastdfs/client, connect_timeout=30, network_timeout=60, tracker_server_count=1, anti_steal_token=0, anti_steal_secret_key length=0, use_connection_pool=0, g_connection_pool_max_idle_time=3600s, use_storage_id=0, storage server id count: 0

tracker_query_storage_store_list_without_group: 
	server 1. group_name=, ip_addr=42.192.23.207, port=23000

group_name=group1, ip_addr=42.192.23.207, port=23000
storage_upload_by_filename
group_name=group1, remote_filename=M00/00/00/CgAEDWRcr_SAKeKxAAAAHNz9dIo846.txt
source ip address: 10.0.4.13
file timestamp=2023-05-11 17:05:56
file size=28
file crc32=3707597962
example file url: http://42.192.23.207/group1/M00/00/00/CgAEDWRcr_SAKeKxAAAAHNz9dIo846.txt
storage_upload_slave_by_filename
group_name=group1, remote_filename=M00/00/00/CgAEDWRcr_SAKeKxAAAAHNz9dIo846_big.txt
source ip address: 10.0.4.13
file timestamp=2023-05-11 17:05:56
file size=28
file crc32=3707597962
example file url: http://42.192.23.207/group1/M00/00/00/CgAEDWRcr_SAKeKxAAAAHNz9dIo846_big.txt

```

根据日志，可以知道我们刚才上传的文件位置是在group1,远程文件路径未我们创建的client目录下的

/opt/fastdfs/storage/files/data/00/00/CgAEDWRcr_SAKeKxAAAAHNz9dIo846.txt



由此我们上传文件的操作就做好了，接下来尝试删除文件和下载文件

```shell
//下载文件
fdfs_test /etc/fdfs/client.conf download group1 M00/00/00/CgAEDWRcr_SAKeKxAAAAHNz9dIo846_big.txt

//删除文件
fdfs_test /etc/fdfs/client.conf delete group1 M00/00/00/CgAEDWRcr_SAKeKxAAAAHNz9dIo846_big.txt
```



到这里跟大家总结一下，我们每次上传文件时最终要上传到/opt/fastdfs/storage/files/data 下，data下有许多文件夹，因此我们在上传时会返回group_name和remote_filename，通过命令与group_name和具体路径的拼接拿到文件的全路径，然后来对文件进行一系列的操作，如上面的下载文件和上传文件。而作为一个客户端来说，比如我想使用Java来操作远程仓库的文件，我一定是在数据库中保存该文件的group_name 以及 remote_filename，这样子每一个文件的路径我们都可以维护起来，之后再想要进行下载文件操作和删除文件操作，就都很简单了。



#### 8.安装Nginx

FastDFS在服务器上不支持浏览器直接访问，因此我们需要配置Nginx来暴露FastDFS对外服务

- 安装Nginx到 /usr/local/soft 目录下
- 解压缩

```shell
tar -zxvf Nginx压缩包名
```

解压完成后

![image-20230515162957413](fastDFS.assets/image-20230515162957413.png)

这里需要两个文件

```shell
//nginx
nginx-1.17.3
//fastdfs提供的nginx拓展模块
fastdfs-nginx-module
```



- 进入fastdfs-nginx-module的src目录下

```shell
cd /fastdfs-nginx-module/src
```



- 拷贝src完整路径

```shell
//获取当前的目录路径
pwd
//输出
/usr/local/soft/fastdfs-nginx-module/src
```



- 配置nginx

```shell
//进入到nginx目录下
cd nginx-1.17.3

//prefix标识nginx安装路径  --add-module表示添加一个模块到nginx  这个模块就是你的fastdfs-nginx-modele/src所在的路径
./configure --prefix=/usr/local/nginx_fdfs --add-module=/usr/local/soft/fastdfs-nginx-module/src

//编译
make


//安装
make install
```





- 配置nginx拓展模块

```shell
//进入拓展模块的src目录
cd fastdfs-nginx-module/src
```

修改mod_fastdfs.conf

```shell
vi mod_fastdfs.conf
```

具体修改的内容

1.修改基础目录  如果没有这个目录 需要我们手动创建

```conf
# 修改前
base_path=/tmp

# 修改后
base_path=/opt/fastdfs/nginx_mod
```



2.指定tracker server的地址

```conf
# 修改前
tracker_server=tracker:22122

# 修改后
tracker_server=127.0.0.1:22122
```



3.修改指定url请求中必须包含组名 必须改为true

```conf
# 修改前
url_have_group_name = false

# 修改后
url_have_group_name = true
```



4.指定文件存储路径

```conf
# 修改前
store_path0=/home/yuqing/fastdfs

# 修改后
store_path0=/opt/fastdfs/storage/files
```



所有配置都改完以后，将mod_fastdfs.conf文件放到/etc/fdfs下



- 创建/opt/fastdfs/nginx_mod 目录，否则nginx会启动不起来



- 配置nginx.conf文件

```shell
cd /usr/locaol/nginx_fdfs/conf

vi nginx.conf

```

```conf
server {
    listen       80;
    server_name  localhost;

    #charset koi8-r;

    #access_log  logs/host.access.log  main;

    location / {
        root   html;
        index  index.html index.htm;
    }

    # 配置FastDFS，拦截文件请求给FastDFS提供的nginx拓展模块 我们只需要添加这个配置即可
    location ~ /group[1-9]/M0[0-9] {
        ngx_fastdfs_module;
    }

    #error_page  404              /404.html;

    # redirect server error pages to the static page /50x.html
    #
    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
        root   html;
    }
}
```



#### 9.启动nginx

```shell
//-t参数，检查配置文件语法是否有问题
/usr/local/nginx_fdfs/sbin/nginx -c /usr/local/nginx_fdfs/conf/nginx.conf -t

//正式启动
/usr/local/nginx_fdfs/sbin/nginx -c /usr/local/nginx_fdfs/conf/nginx.conf   //-c后的路径是自己的nginx.conf文件所在的绝对路径
```

```shell
#查看是否启动成功
ps -ef | grep nginx
```

如果失败，可以查看nginx的日志文件

```shell
cd /usr/local/nginx_fdfs
cd logs
```

启动失败的原因: 1.mod_fastdfs.conf没有方法/etc/fdfs目录下  2.mode_fastdfs.conf的base_path错误





#### 10.Java操作FastDFS

##### 10.1 由于maven仓库没有FastDFS,因此我们需要从github中下载并在自己本地打包到本地仓库中

> https://github.com/happyfish100/fastdfs-client-java



##### **10.2 新建一个maven工程并导入依赖**

```xml
<dependencies>
    <dependency>
        <groupId>org.csource</groupId>
        <artifactId>fastdfs-client-java</artifactId>
        <version>1.27-SNAPSHOT</version>
    </dependency>
</dependencies>

```



##### **10.3 配置tracker server地址**

```txt
tracker_server=xxx.xxx.xxx.xxx:22122  //写上自己的服务器地址
```





##### **10.4 新增FastDFS工具类**

```java
package com.fannnnn.util;

import org.csource.common.MyException;
import org.csource.fastdfs.*;

import java.io.IOException;

public class FastDFSUtil {
    /**
     * 文件上传
     */
    public static String[] upload(String localFileName) {
        TrackerServer trackerServer = null;
        StorageServer storageServer = null;
        try {
            //读取配置文件，用于将所有tracker server的地址读取到内存中
            ClientGlobal.init("fastdfs.conf");
            TrackerClient trackerClient = new TrackerClient();
            trackerServer = trackerClient.getConnection();
            storageServer = trackerClient.getStoreStorage(trackerServer);
            //定义Storage的客户端对象，需要使用这个对象来完成具体的文件上传、下载和删除操作
            StorageClient storageClient = new StorageClient(trackerServer, storageServer);
            //上传，参数一：需要上传的文件的绝对路径，参数二：需要上传的文件的扩展名，参数三：文件的属性文件，通常不上传
            //返回一个String数组，这个数据对我们非常有用，必须妥善保管，建议保存到数据库
            //返回结果数组：第一个元素为文件所在的组名，第二个元素为文件所在的远程路径名称
            return storageClient.upload_file(localFileName, "png", null);
        } catch (IOException | MyException e) {
            e.printStackTrace();
        } finally {
            //关闭资源
            try {
                if (storageServer != null) {
                    storageServer.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (trackerServer != null) {
                    trackerServer.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return null;
    }

    /**
     * 下载
     *
     * @param groupName      组名
     * @param remoteFileName 远程文件名称
     * @param saveFileName   保存的文件名称
     * @return 返回是否下载成功
     */
    public static boolean download(String groupName, String remoteFileName, String saveFileName) {
        TrackerServer trackerServer = null;
        StorageServer storageServer = null;
        try {
            //读取配置文件，用于将所有tracker server的地址读取到内存中
            ClientGlobal.init("fastdfs.conf");
            TrackerClient trackerClient = new TrackerClient();
            trackerServer = trackerClient.getConnection();
            storageServer = trackerClient.getStoreStorage(trackerServer);
            //定义Storage的客户端对象，需要使用这个对象来完成具体的文件上传、下载和删除操作
            StorageClient storageClient = new StorageClient(trackerServer, storageServer);
            //文件下载，参数一：文件的组名，参数二：文件的远程文件名，参数三：保存到本地文件的名称
            //返回0，则为文件下载成功，其他值表示下载失败
            int code = storageClient.download_file(groupName, remoteFileName, saveFileName);
            return code == 0;
        } catch (IOException | MyException e) {
            e.printStackTrace();
        } finally {
            //关闭资源
            try {
                if (storageServer != null) {
                    storageServer.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (trackerServer != null) {
                    trackerServer.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return false;
    }

    /**
     * 文件删除
     *
     * @param groupName      组名
     * @param remoteFileName 远程文件名称
     */
    public static boolean delete(String groupName, String remoteFileName) {
        TrackerServer trackerServer = null;
        StorageServer storageServer = null;
        try {
            //读取配置文件，用于将所有tracker server的地址读取到内存中
            ClientGlobal.init("fastdfs.conf");
            TrackerClient trackerClient = new TrackerClient();
            trackerServer = trackerClient.getConnection();
            storageServer = trackerClient.getStoreStorage(trackerServer);
            //定义Storage的客户端对象，需要使用这个对象来完成具体的文件上传、下载和删除操作
            StorageClient storageClient = new StorageClient(trackerServer, storageServer);
            //文件删除，参数一：需要删除的文件组名，参数二：文件的远程名称，返回int结果，为0则成功，其他则为失败
            int code = storageClient.delete_file(groupName, remoteFileName);
            return code == 0;
        } catch (IOException | MyException e) {
            e.printStackTrace();
        } finally {
            //关闭资源
            try {
                if (storageServer != null) {
                    storageServer.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
            try {
                if (trackerServer != null) {
                    trackerServer.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return false;
    }
}
```



##### 10.5 测试

```java
package com.fannnnn.util;

/**
 * \*  Created with IntelliJ IDEA.
 * \*  @author shensifan
 * \*  Date: 2023/5/16
 * \*  Time: 10:36
 * \*  Description:
 * \
 */

public class test {

    public static void main(String[] args) {
//        String[] results = FastDFSUtil.upload("G:\\沈思帆.pdf");
//        if (results!=null && results.length==2) {
//            String group = results[0];
//            String remoteFileName = results[1];
//            System.out.println("group:"+group);
//            System.out.println("remoteFileName:"+remoteFileName);
//            System.out.println("链接地址: http://42.192.23.207/"+group+"/"+remoteFileName);
//        }
        download();
    }

    /**
     * 上传文件
     */
    private static void upload() {
        String[] results = FastDFSUtil.upload("/Users/wally/Desktop/Code/Java/fastdfs-java/src/main/resources/FastDFS命名解析.png");
        if (results != null && results.length == 2) {
            String group = results[0];
            String remoteFileName = results[1];
            System.out.println("group：" +group);
            System.out.println("remoteFileName：" + remoteFileName);
            System.out.println("链接地址：http://192.168.211.131/" + group + "/" + remoteFileName);
        }
    }



    /**
     * 下载文件
     */
    private static void download() {
        //下载文件到项目的根目录
        boolean result = FastDFSUtil.download("group1",
                "M00/00/00/CgAEDWRi79qAabDHAAdEJVWnCdM638.png",
                "a.pdf");
        if (result) {
            System.out.println("文件下载成功！");
        } else {
            System.out.println("文件下载失败！");
        }
    }


}

```

