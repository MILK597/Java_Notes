# MinIO简介   

MinIO基于Apache License v2.0开源协议的对象存储服务，可以做为云存储的解决方案用来保存海量的图片，视频，文档。由于采用Golang实现，服务端可以工作在Windows,Linux, OS X和FreeBSD上。配置简单，基本是复制可执行程序，单行命令可以运行起来。

MinIO兼容亚马逊S3云存储服务接口，非常适合于存储大容量非结构化的数据，例如图片、视频、日志文件、备份数据和容器/虚拟机镜像等，而一个对象文件可以是任意大小，从几kb到最大5T不等。

**S3 （ Simple Storage Service简单存储服务）**

基本概念

- Bucket – 类比于文件系统的目录
- Object – 类比文件系统的文件
- Keys – 类比文件名

官网文档：http://docs.minio.org.cn/docs/



# Docker部署   

我们可以使用docker进行环境部署和启动

```yaml
docker run -p 9000:9000 --name minio -d --restart=always -e "MINIO_ACCESS_KEY=minio" -e "MINIO_SECRET_KEY=minio123" -v /home/data:/data -v /home/config:/root/.minio minio/minio server /data
```



# Java客户端

## 依赖

```xml
<dependencies>
    <dependency>
        <groupId>io.minio</groupId>
        <artifactId>minio</artifactId>
        <version>7.1.0</version>
    </dependency>
</dependencies>
```

## 上传文件

```java
FileInputStream fileInputStream = null;
fileInputStream =  new FileInputStream("D:\\list.html");;

//1.创建minio链接客户端
MinioClient minioClient = MinioClient
    .builder()
    .credentials("minio", "minio123")
    .endpoint("http://192.168.200.130:9000")
    .build();

//2.上传
PutObjectArgs putObjectArgs = PutObjectArgs
    .builder()
    .object("list.html")//文件名
    .contentType("text/html")//文件类型
    .bucket("leadnews")//桶名词  与minio创建的名词一致
    .stream(fileInputStream, fileInputStream.available(), -1) //输入流，文件的内容
    .build();
minioClient.putObject(putObjectArgs);

上传之后文件的访问url就是：minio地址(endpoint)/bucket/文件名
System.out.println("http://192.168.200.130:9000/leadnews/list.html");
```

## 删除文件

```java
// 指定 bucket、文件名即可。
RemoveObjectArgs removeObjectArgs = RemoveObjectArgs.builder().bucket(bucket).object(filePath).build();
try {
    minioClient.removeObject(removeObjectArgs);
} catch (Exception e) {
    log.error("minio remove file error.  pathUrl:{}",pathUrl);
    e.printStackTrace();
}
```

## 下载文件

```java

InputStream inputStream = null;
try {
    //传入bucket 、 文件名
    inputStream = minioClient.getObject(GetObjectArgs.builder().bucket(bucket).object(filename).build());
} catch (Exception e) {
    e.printStackTrace();
}

ByteArrayOutputStream byteArrayOutputStream = new ByteArrayOutputStream();
byte[] buff = new byte[100];
int rc = 0;
while (true) {
    try {
        if (!((rc = inputStream.read(buff, 0, 100)) > 0)) break;
    } catch (IOException e) {
        e.printStackTrace();
    }
    byteArrayOutputStream.write(buff, 0, rc);
}
return byteArrayOutputStream.toByteArray();
```

