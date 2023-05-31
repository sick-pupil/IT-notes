## 1. 背景
对象存储出现之前存在的两种主要存储方式：
- **块存储**：像是**一块块硬盘直接挂载在主机上，以卷或硬盘形式体现**，对于存储的数据内容和格式一无所知，**只关心读取和写入，数据按字节来访问**，不关心关系和用途，性能很高；但是太偏向于底层。例：DAS直连式存储、SAN存储区域网络
- **文件存储**：一般以**文件和目录**形式存在，数据以文件的形式进行存取，也可以进行一些高级管理功能，比如文件层面的访问权限控制，文件共享，但读写速度慢，例：NAS网络附加存储服务器

## 2. 对象存储
<img src="D:\Project\IT-notes\框架or中间件\Minio\img\对象存储概念图.png" style="width:700px;height:400px;" />

- **租户**：用于隔离存储资源。在租户下可以建立桶、存储对象
- **用户**：在租户下面创建的用于访问不同桶的账号。可以使用minio提供的mc命令设置不同用户访问各个桶的权限
- **桶**：是若干个对象的逻辑抽象，是盛装对象的容器
- **对象**：类似于`hash`表中的表项，名字是关键字，内容相当于值

## 3. minio集群节点存储
`MINIO`整个集群是由多个角色完全相同的节点组成，没有特殊节点，任何一个节点宕机，都不会影响整个集群，对象被分片打散之后存放在不同节点的多块硬盘上，对外提供统一的命名空间，通过`Web`负载均衡或`DNS`轮询`Round Robin`的方式在各个节点上实现负载均衡

`minio`节点存储相关概念：
- `drive`：存储数据的磁盘
- `set`：一组`drive`的集合构成一个`set`，每个`set`的`drive`会尽可能分布在不同节点上；一个对象存储在一个`set`上
- `bucket`：文件对象存储的逻辑位置；对于客户端而言，相当于存放文件的顶层文件夹

节点存储大致原理：
- `MINIO`不是以多副本的形式存储，而是通过数据编码，将原来的数据编码成多份，然后通过`Set`的形式落地存储在对应的`Drive`上
- 一个集群包含多个`Set`，具体存储到哪个set是通过对象名称进行哈希，映射到唯一一个`Set`上，这种方式可以使得数据均匀分布在所有的`Drive`上
- 一个集群具体包含多少个`Set`，`minio`默认会根据集群规模自动计算得出，也可以自行指定配置。本着鸡蛋放在多个篮子，保证数据可靠性的原则，一个`Set`的`Drive`尽可能分布在不同的节点上
- 存储对象时，会将对象拆分成`N`个数据块与N个奇偶校验块，分布式集群中一个`set`中的每个磁盘都会创建`bucketName/objName`路径，再分别存储一个数据块与一个奇偶校验块
- 通过生成奇偶校验块达到“丢失一半数据也可以从剩下的数据块中恢复数据”的功能
- 管理分片数据是通过对象的元数据进行分片的存储与管理

## 4. 分布式锁
`minio`也会存在面临数据一致性的问题:一个客户端在读取一个对象的同时，另一个客户端可能正在修改或者删除这个对象，`minio`专门设计并实现了`dsync`分布式锁管理器，来控制数据一致性
- 任何一个节点的锁请求都会广播给集群内的所有在线节点
- 如果收到N/2+1个节点的同意，则获取所成功
- 没有主节点，每个节点互相对等，节点间通过stale lock检测机制，判断节点的状态及持有锁情况
- 有一定的缺陷性，最多支持32个节点。无法避免锁丢失的场景。不过基本满足可用需求

## 5. minio安装
下载：`https://min.io/download#/windows`
安装：设置环境变量并创建`bat`启动文件
```shell
# 设置环境变量
setx MINIO_ROOT_USER admin
setx MINIO_ROOT_PASSWORD password

# bat启动文件内容 console-address为网页管理端口 address为api调用端口
@echo off 
set path=D:\MinIO
set minPath=D:\MinIO\Data
%path%\minio.exe server %minPath% --console-address ":9090" --address ":9000"
pause
```

## 6. Springboot整合minio
```yml
spring:
	# 配置文件上传大小限制（minio文件上传）
	servlet:
		multipart:
			max-file-size: 200MB
			max-request-size: 200MB

minio:
	endpoint: http://127.0.0.1:9000
	accessKey:  minioadmin
	secretKey:  minioadmin
	bucketName: community-web
```

```java
package com.example.config;

import io.minio.MinioClient;
import lombok.Data;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.context.annotation.Bean;
import org.springframework.stereotype.Component;

@Data
@Component
public class MinIoClientConfig {
    @Value("${minio.endpoint}")
    private String endpoint;
    @Value("${minio.accessKey}")
    private String accessKey;
    @Value("${minio.secretKey}")
    private String secretKey;


    /**
     * 注入minio 客户端
     *
     * @return
     */
    @Bean
    public MinioClient minioClient() {

        return MinioClient.builder()
                .endpoint(endpoint)
                .credentials(accessKey, secretKey)
                .build();
    }
}
```

```java
package com.example.sys.entity;

import lombok.Data;

@Data
public class ObjectItem {
    private String objectName;
    private Long size;
}
```

```java
package com.example.utils;

import com.example.sys.entity.ObjectItem;
import io.minio.*;
import io.minio.messages.DeleteError;
import io.minio.messages.DeleteObject;
import io.minio.messages.Item;
import org.apache.tomcat.util.http.fileupload.IOUtils;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.http.HttpHeaders;
import org.springframework.http.HttpStatus;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Component;
import org.springframework.web.multipart.MultipartFile;

import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.InputStream;
import java.io.UnsupportedEncodingException;
import java.net.URLEncoder;
import java.util.ArrayList;
import java.util.Arrays;
import java.util.List;
import java.util.stream.Collectors;

/**
 * @description： minio工具类
 * @version：1.0
 */
@Component
public class MinioUtils {
    @Autowired
    private MinioClient minioClient;

    @Value("${minio.bucketName}")
    private String bucketName;

    /**
     * description: 判断bucket是否存在，不存在则创建
     *
     * @return: void
     */
    public void existBucket(String name) {
        try {
            boolean exists = minioClient.bucketExists(BucketExistsArgs.builder().bucket(name).build());
            if (!exists) {
                minioClient.makeBucket(MakeBucketArgs.builder().bucket(name).build());
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 创建存储bucket
     *
     * @param bucketName 存储bucket名称
     * @return Boolean
     */
    public Boolean makeBucket(String bucketName) {
        try {
            minioClient.makeBucket(MakeBucketArgs.builder()
                    .bucket(bucketName)
                    .build());
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }

    /**
     * 删除存储bucket
     *
     * @param bucketName 存储bucket名称
     * @return Boolean
     */
    public Boolean removeBucket(String bucketName) {
        try {
            minioClient.removeBucket(RemoveBucketArgs.builder()
                    .bucket(bucketName)
                    .build());
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }
        return true;
    }

    /**
     * description: 上传文件
     *
     * @param multipartFile
     * @return: java.lang.String
     */
    public List<String> upload(MultipartFile[] multipartFile) {
        List<String> names = new ArrayList<>(multipartFile.length);
        for (MultipartFile file : multipartFile) {
            String fileName = file.getOriginalFilename();
            String[] split = fileName.split("\\.");
            if (split.length > 1) {
                fileName = split[0] + "_" + System.currentTimeMillis() + "." + split[1];
            } else {
                fileName = fileName + System.currentTimeMillis();
            }
            InputStream in = null;
            try {
                in = file.getInputStream();
                minioClient.putObject(PutObjectArgs.builder()
                        .bucket(bucketName)
                        .object(fileName)
                        .stream(in, in.available(), -1)
                        .contentType(file.getContentType())
                        .build()
                );
            } catch (Exception e) {
                e.printStackTrace();
            } finally {
                if (in != null) {
                    try {
                        in.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
            }
            names.add(fileName);
        }
        return names;
    }

    /**
     * description: 下载文件
     *
     * @param fileName
     * @return: org.springframework.http.ResponseEntity<byte [ ]>
     */
    public ResponseEntity<byte[]> download(String fileName) {
        ResponseEntity<byte[]> responseEntity = null;
        InputStream in = null;
        ByteArrayOutputStream out = null;
        try {
            in = minioClient.getObject(GetObjectArgs.builder().bucket(bucketName).object(fileName).build());
            out = new ByteArrayOutputStream();
            IOUtils.copy(in, out);
            //封装返回值
            byte[] bytes = out.toByteArray();
            HttpHeaders headers = new HttpHeaders();
            try {
                headers.add("Content-Disposition", "attachment;filename=" + URLEncoder.encode(fileName, "UTF-8"));
            } catch (UnsupportedEncodingException e) {
                e.printStackTrace();
            }
            headers.setContentLength(bytes.length);
            headers.setContentType(MediaType.APPLICATION_OCTET_STREAM);
            headers.setAccessControlExposeHeaders(Arrays.asList("*"));
            responseEntity = new ResponseEntity<byte[]>(bytes, headers, HttpStatus.OK);
        } catch (Exception e) {
            e.printStackTrace();
        } finally {
            try {
                if (in != null) {
                    try {
                        in.close();
                    } catch (IOException e) {
                        e.printStackTrace();
                    }
                }
                if (out != null) {
                    out.close();
                }
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
        return responseEntity;
    }

    /**
     * 查看文件对象
     *
     * @param bucketName 存储bucket名称
     * @return 存储bucket内文件对象信息
     */
    public List<ObjectItem> listObjects(String bucketName) {
        Iterable<Result<Item>> results = minioClient.listObjects(
                ListObjectsArgs.builder().bucket(bucketName).build());
        List<ObjectItem> objectItems = new ArrayList<>();
        try {
            for (Result<Item> result : results) {
                Item item = result.get();
                ObjectItem objectItem = new ObjectItem();
                objectItem.setObjectName(item.objectName());
                objectItem.setSize(item.size());
                objectItems.add(objectItem);
            }
        } catch (Exception e) {
            e.printStackTrace();
            return null;
        }
        return objectItems;
    }

    /**
     * 批量删除文件对象
     *
     * @param bucketName 存储bucket名称
     * @param objects    对象名称集合
     */
    public Iterable<Result<DeleteError>> removeObjects(String bucketName, List<String> objects) {
        List<DeleteObject> dos = objects.stream().map(e -> new DeleteObject(e)).collect(Collectors.toList());
        Iterable<Result<DeleteError>> results = minioClient.removeObjects(RemoveObjectsArgs.builder().bucket(bucketName).objects(dos).build());
        return results;
    }
}
```

```java
package com.example.sys.controller;

import com.example.utils.MinioUtils;
import lombok.extern.slf4j.Slf4j;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Value;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RestController;
import org.springframework.web.multipart.MultipartFile;

import java.util.List;

@RestController
@Slf4j
public class MinioController {
    @Autowired
    private MinioUtils minioUtils;
    @Value("${minio.endpoint}")
    private String address;
    @Value("${minio.bucketName}")
    private String bucketName;

    @PostMapping("/upload")
    public Object upload(MultipartFile file) {

        List<String> upload = minioUtils.upload(new MultipartFile[]{file});

        return address + "/" + bucketName + "/" + upload.get(0);
    }

}
```