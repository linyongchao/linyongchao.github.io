---
layout: post
title:  Spring 整合 MongoDB
date:   2019-08-09 16:36:54
categories: Java
---

* content
{:toc}

需要将同事写的 MongoDB 操作类整合到 SpringBoot 或者 SpringMVC，结果周三周四两天一个都没有搭起来，囧  
网上搜索错误，说什么的都有，就是不解决问题  
结果今天上午福灵心至、灵光一闪、灵机一动，鬼知道啥原因呢，突然就都解决了，o(╯□╰)o  
特记录一下

---

后记：  
环境搭好了，博客写完了，怎么也想不明白当初问啥死活搭建不成功了，囧  
只能说人都有犯傻的时候吧，o(╯□╰)o

## SpringBoot

### pom.xml

很简单，只有 SpringBoot、MongoDB、gson

	<?xml version="1.0" encoding="UTF-8"?>
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <modelVersion>4.0.0</modelVersion>
	    <parent>
	        <groupId>org.springframework.boot</groupId>
	        <artifactId>spring-boot-starter-parent</artifactId>
	        <version>2.1.7.RELEASE</version>
	        <relativePath/>
	    </parent>
	    <groupId>com.example</groupId>
	    <artifactId>demo</artifactId>
	    <version>0.0.1-SNAPSHOT</version>
	    <name>demo</name>
	    <description>Demo project for Spring Boot</description>
	
	    <properties>
	        <java.version>1.8</java.version>
	    </properties>
	
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-web</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-test</artifactId>
	            <scope>test</scope>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework.boot</groupId>
	            <artifactId>spring-boot-starter-data-mongodb</artifactId>
	        </dependency>
	        <dependency>
	            <groupId>com.google.code.gson</groupId>
	            <artifactId>gson</artifactId>
	            <version>2.8.0</version>
	        </dependency>
	    </dependencies>
	
	    <build>
	        <resources>
	            <resource>
	                <directory>src/main/resources/</directory>
	                <includes>
	                    <include>*.yml</include>
	                </includes>
	                <filtering>true</filtering>
	            </resource>
	        </resources>
	        <plugins>
	            <plugin>
	                <groupId>org.springframework.boot</groupId>
	                <artifactId>spring-boot-maven-plugin</artifactId>
	            </plugin>
	        </plugins>
	    </build>
	
	</project>


### application.yml

	spring:
	  application:
	    name: springboot.mongo.test
	  data:
	    mongodb:
	      uri: mongodb://file:123456@192.168.3.187:27017/filetest

### com.example.demo.model

#### FileInfo.java

	package com.example.demo.model;
	
	/**
	 * Mongdb中的文件对象
	 */
	public class FileInfo {
	    //文件对象ID
	    private String id;
	    //文件ID
	    private String fileId;
	
	    //文件名称
	    private String fileName;
	    //文件大小
	    private Long fileSize;
	
	    //文件MD5值
	    private String fileMD5;
	    //业务类型
	    private String bizType;
	
	    public String getBizType() {
	        return bizType;
	    }
	
	    public void setBizType(String bizType) {
	        this.bizType = bizType;
	    }
	
	    public String getMark() {
	        return mark;
	    }
	
	    public void setMark(String mark) {
	        this.mark = mark;
	    }
	
	    //备注
	    private String mark;
	
	    public String getFileMD5() {
	        return fileMD5;
	    }
	
	    public void setFileMD5(String fileMD5) {
	        this.fileMD5 = fileMD5;
	    }
	
	    public Long getFileSize() {
	        return fileSize;
	    }
	
	    public void setFileSize(Long fileSize) {
	        this.fileSize = fileSize;
	    }
	
	    public String getId() {
	        return id;
	    }
	
	    public void setId(String id) {
	        this.id = id;
	    }
	
	    public String getFileId() {
	        return fileId;
	    }
	
	    public void setFileId(String fileId) {
	        this.fileId = fileId;
	    }
	
	
	    public String getFileName() {
	        return fileName;
	    }
	
	    public void setFileName(String fileName) {
	        this.fileName = fileName;
	    }
	}


#### UserTest.java

	package com.example.demo.model;
	
	public class UserTest {
	    private Integer id;
	    private Integer age;
	    private String name;
	
	    public Integer getId() {
	        return id;
	    }
	
	    public void setId(Integer id) {
	        this.id = id;
	    }
	
	    public Integer getAge() {
	        return age;
	    }
	
	    public void setAge(Integer age) {
	        this.age = age;
	    }
	
	    public String getName() {
	        return name;
	    }
	
	    public void setName(String name) {
	        this.name = name;
	    }
	}


### com.example.demo.dao

#### MongoDbDao.java

	package com.example.demo.dao;
	
	import com.example.demo.model.FileInfo;
	import com.google.gson.Gson;
	import com.mongodb.BasicDBObject;
	import com.mongodb.client.MongoCursor;
	import com.mongodb.client.gridfs.GridFSBucket;
	import com.mongodb.client.gridfs.GridFSDownloadStream;
	import com.mongodb.client.gridfs.GridFSFindIterable;
	import com.mongodb.client.gridfs.model.GridFSFile;
	import org.bson.types.ObjectId;
	import org.slf4j.Logger;
	import org.slf4j.LoggerFactory;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.data.domain.Sort;
	import org.springframework.data.mongodb.core.MongoTemplate;
	import org.springframework.data.mongodb.core.query.Criteria;
	import org.springframework.data.mongodb.core.query.Query;
	import org.springframework.data.mongodb.core.query.Update;
	import org.springframework.data.mongodb.gridfs.GridFsResource;
	import org.springframework.data.mongodb.gridfs.GridFsTemplate;
	
	import javax.servlet.ServletOutputStream;
	import javax.servlet.http.HttpServletResponse;
	import java.io.IOException;
	import java.io.InputStream;
	import java.lang.reflect.Field;
	import java.lang.reflect.Method;
	import java.net.URLEncoder;
	import java.security.MessageDigest;
	import java.util.Date;
	import java.util.List;
	
	public abstract class MongoDbDao<T> {
	    protected Logger logger = LoggerFactory.getLogger(MongoDbDao.class);
	    final static char HEXDIGITS[] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h', 'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p', 'q', 'r', 's', 't', 'u', 'v', 'w', 'x', 'y', 'z'};
	
	    /**
	     * 反射获取泛型类型
	     *
	     * @return
	     */
	    protected abstract Class<T> getEntityClass();
	
	    @Autowired
	    private MongoTemplate mongoTemplate;
	
	    @Autowired
	    private GridFsTemplate gridFsTemplate;
	
	    @Autowired
	    private GridFSBucket gridFSBucket;
	
	    /**
	     * 上传文件
	     * 索引字段: fileId,chunkId
	     * chunkId 判断文件块是否可以复用
	     *
	     * @param inputStream
	     * @param fileName
	     * @param fileId
	     * @param chunkId
	     * @param chunkSeq
	     * @param chunkSize
	     * @return
	     */
	    public ObjectId savefile(InputStream inputStream, String fileName, String fileId, String chunkId, Integer chunkSeq, String chunkSize, String bizType, String mark) {
	        //检查文件块是否存在--
	        String name = fileName.substring(0, fileName.lastIndexOf("."));
	        String contentType = fileName.substring(fileName.lastIndexOf(".") + 1);
	        BasicDBObject metadata = new BasicDBObject();
	        //文件唯一标识，用来筛选对应文件的所有块---
	        metadata.append("fileId", fileId);
	        //文件块的唯一标识,非必传字段, 用来判断本文件块是否已经存在,存在就直接copy
	        metadata.append("chunkId", chunkId);
	        //前端传:块下标，下载文件时需要合并块，用这个参数排序
	        metadata.append("chunkSeq", chunkSeq);
	        //前端传:块大小，断点续传时，用这个参数判断该块是否完整上传
	        metadata.append("chunkSize", chunkSize);
	
	        metadata.put("createdDate", new Date());
	        metadata.put("bizType", bizType);
	        metadata.put("mark", mark);
	
	        ObjectId objectId = gridFsTemplate.store(inputStream, name, contentType, metadata);
	        try {
	            if (inputStream != null) {
	                inputStream.close();
	            }
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	        return objectId;
	    }
	
	    /**
	     * //根据文件唯一标识和块下标确定唯一一个块
	     *
	     * @param fileId
	     * @param chunkSeq
	     * @return
	     */
	    public GridFSFile getFileChunk(String fileId, Integer chunkSeq) {
	        return gridFsTemplate.findOne(new Query(Criteria.where("metadata.fileId").is(fileId).and("metadata.chunkSeq").is(chunkSeq)));
	    }
	
	    //
	    public String downloadFile(String fileId, Long skip, HttpServletResponse response) {
	        GridFSFile gfsFile = null;
	        GridFSFindIterable iterable = gridFsTemplate.find(new Query(Criteria.where("metadata.fileId").is(fileId)).with(new Sort(Sort.Direction.ASC, "metadata.chunkSeq")));
	        MongoCursor<GridFSFile> cursor = iterable.iterator();
	        try {
	            ServletOutputStream out = response.getOutputStream();
	            InputStream inputStream = null;
	            //response需要的文件总长度
	            long contentLength = 0;
	            while (cursor.hasNext()) {
	                gfsFile = cursor.next();
	                if (gfsFile == null) {
	                    continue;
	                }
	                GridFSDownloadStream downloadStream = gridFSBucket.openDownloadStream(gfsFile.getObjectId());
	                GridFsResource gridFsResource = new GridFsResource(gfsFile, downloadStream);
	                inputStream = gridFsResource.getInputStream();
	                long chunkLength = gfsFile.getLength();
	
	                //skip有值,需要跳转
	                if (null != skip && skip >= 0) {
	                    logger.info("----开始文件ID:{}续传下载,当前文件块ID:{},文件块序号:{}, skip值:{}", gfsFile.getMetadata().get("fileId"), gfsFile.getObjectId(), gfsFile.getMetadata().get("chunkSeq"), skip);
	                    if (skip >= 0) {
	                        if (skip < chunkLength) {
	                            inputStream.skip(skip);
	                            contentLength += gfsFile.getLength() - skip;
	                            //下一个文件块不需要skip
	                            skip = null;
	                        } else {
	                            //下一个文件块要跳的字节数
	                            skip -= chunkLength;
	                            logger.info("----文件ID:{}跳过第{}个文件块:{}", gfsFile.getMetadata().get("fileId"), gfsFile.getMetadata().get("chunkSeq"), gfsFile.getObjectId());
	                            continue;
	                        }
	                    }
	                } else {
	                    contentLength += gfsFile.getLength();
	                }
	                String contentType = gridFsResource.getContentType();
	                String filename = gridFsResource.getFilename();
	                //2.设置文件头：
	                response.setHeader("Content-Disposition", "attachment;filename=" + URLEncoder.encode(filename + "." + contentType, "utf-8"));
	                int len = -1;
	                byte[] b = new byte[1024];
	                while ((len = inputStream.read(b)) != -1) {
	                    out.write(b, 0, len);
	                }
	                inputStream.close();
	            }
	            response.setHeader("Content-Length", "" + contentLength);
	            out.flush();
	            out.close();
	
	        } catch (IOException e) {
	            e.printStackTrace();
	        }
	        return "success";
	    }
	
	
	    //删除文档类型的文件
	    public void deleteFile(String fileId) {
	        gridFsTemplate.delete(new Query(Criteria.where("metadata.fileId").is(fileId)));
	    }
	
	    public void deleteChunk(String fileId, String chunkId) {
	        logger.info("删除chunk: 文件ID:{},chunkId:{}", fileId, chunkId);
	        gridFsTemplate.delete(new Query(Criteria.where("metadata.chunkId").is(chunkId).and("metadata.fileId").is(fileId)));
	    }
	
	    public void deleteChunkById(ObjectId objectId) {
	        logger.info("删除chunk: objectId:{}", objectId);
	        gridFsTemplate.delete(new Query(Criteria.where("_id").is(objectId)));
	    }
	
	    /**
	     * 查询文件信息
	     *
	     * @param fileId
	     * @return
	     */
	    public String getFileInfo(String fileId) {
	        logger.info("get file Info: {}", fileId);
	        GridFSFile gfsFile = gridFsTemplate.findOne(new Query(Criteria.where("metadata.fileId").is(fileId)));
	        if (gfsFile == null) {
	            return null;
	        }
	        FileInfo fileInfo = new FileInfo();
	        fileInfo.setId(gfsFile.getObjectId().toString());
	        fileInfo.setFileId(gfsFile.getMetadata().get("fileId").toString());
	        fileInfo.setFileName(gfsFile.getFilename());
	        fileInfo.setFileSize(gfsFile.getLength());
	        fileInfo.setFileMD5(gfsFile.getMD5());
	        if (gfsFile.getMetadata().get("bizType") != null) {
	            fileInfo.setBizType(gfsFile.getMetadata().get("bizType").toString());
	        }
	        if (gfsFile.getMetadata().get("mark") != null) {
	            fileInfo.setMark(gfsFile.getMetadata().get("mark").toString());
	        }
	        return new Gson().toJson(fileInfo);
	    }
	
	    /***
	     * 保存一个对象
	     * @param t
	     */
	    public void save(T t) {
	        logger.info("-------------->MongoDB save start");
	        this.mongoTemplate.save(t);
	    }
	
	    /***
	     * 根据id从几何中查询对象
	     * @param id
	     * @return
	     */
	    public T queryById(Integer id) {
	        Query query = new Query(Criteria.where("_id").is(id));
	        logger.info("-------------->MongoDB find start");
	        return this.mongoTemplate.findOne(query, this.getEntityClass());
	    }
	
	    /**
	     * 根据条件查询集合
	     *
	     * @param object
	     * @return
	     */
	    public List<T> queryList(T object) {
	        Query query = getQueryByObject(object);
	        logger.info("-------------->MongoDB find start");
	        return mongoTemplate.find(query, this.getEntityClass());
	    }
	
	    /**
	     * 根据条件查询只返回一个文档
	     *
	     * @param object
	     * @return
	     */
	    public T queryOne(T object) {
	        Query query = getQueryByObject(object);
	        logger.info("-------------->MongoDB find start");
	        return mongoTemplate.findOne(query, this.getEntityClass());
	    }
	
	    /***
	     * 根据条件分页查询
	     * @param object
	     * @param start 查询起始值
	     * @param size  查询大小
	     * @return
	     */
	    public List<T> getPage(T object, int start, int size) {
	        Query query = getQueryByObject(object);
	        query.skip(start);
	        query.limit(size);
	        logger.info("-------------->MongoDB queryPage start");
	        return this.mongoTemplate.find(query, this.getEntityClass());
	    }
	
	    /***
	     * 根据条件查询库中符合条件的记录数量
	     * @param object
	     * @return
	     */
	    public Long getCount(T object) {
	        Query query = getQueryByObject(object);
	        logger.info("-------------->MongoDB Count start");
	        return this.mongoTemplate.count(query, this.getEntityClass());
	    }
	
	    /***
	     * 删除对象
	     * @param t
	     * @return
	     */
	    protected int delete(T t) {
	        logger.info("-------------->MongoDB delete start");
	        return (int) this.mongoTemplate.remove(t).getDeletedCount();
	    }
	
	    /**
	     * 根据id删除
	     *
	     * @param id
	     */
	    public void deleteById(Integer id) {
	        Criteria criteria = Criteria.where("_id").is(id);
	        if (null != criteria) {
	            Query query = new Query(criteria);
	            T obj = this.mongoTemplate.findOne(query, this.getEntityClass());
	            logger.info("-------------->MongoDB deleteById start");
	            if (obj != null) {
	                this.delete(obj);
	            }
	        }
	    }
	
	    /*MongoDB中更新操作分为三种
	     * 1：updateFirst     修改第一条
	     * 2：updateMulti     修改所有匹配的记录
	     * 3：upsert  修改时如果不存在则进行添加操作
	     * */
	
	    /**
	     * 修改匹配到的第一条记录
	     *
	     * @param srcObj
	     * @param targetObj
	     */
	    public void updateFirst(T srcObj, T targetObj) {
	        Query query = getQueryByObject(srcObj);
	        Update update = getUpdateByObject(targetObj);
	        logger.info("-------------->MongoDB updateFirst start");
	        this.mongoTemplate.updateFirst(query, update, this.getEntityClass());
	    }
	
	    /***
	     * 修改匹配到的所有记录
	     * @param srcObj
	     * @param targetObj
	     */
	    public void updateMulti(T srcObj, T targetObj) {
	        Query query = getQueryByObject(srcObj);
	        Update update = getUpdateByObject(targetObj);
	        logger.info("-------------->MongoDB updateFirst start");
	        this.mongoTemplate.updateMulti(query, update, this.getEntityClass());
	    }
	
	    /***
	     * 修改匹配到的记录，若不存在该记录则进行添加
	     * @param srcObj
	     * @param targetObj
	     */
	    public void updateInsert(T srcObj, T targetObj) {
	        Query query = getQueryByObject(srcObj);
	        Update update = getUpdateByObject(targetObj);
	        logger.info("-------------->MongoDB updateInsert start");
	        this.mongoTemplate.upsert(query, update, this.getEntityClass());
	    }
	
	    /**
	     * 将查询条件对象转换为query
	     *
	     * @param object
	     * @return
	     * @author Jason
	     */
	    private Query getQueryByObject(T object) {
	        Query query = new Query();
	        String[] fileds = getFiledName(object);
	        Criteria criteria = new Criteria();
	        for (int i = 0; i < fileds.length; i++) {
	            String filedName = (String) fileds[i];
	            Object filedValue = getFieldValueByName(filedName, object);
	            if (filedValue != null) {
	                criteria.and(filedName).is(filedValue);
	            }
	        }
	        query.addCriteria(criteria);
	        return query;
	    }
	
	    /**
	     * 将查询条件对象转换为update
	     *
	     * @param object
	     * @return
	     * @author Jason
	     */
	    private Update getUpdateByObject(T object) {
	        Update update = new Update();
	        String[] fileds = getFiledName(object);
	        for (int i = 0; i < fileds.length; i++) {
	            String filedName = (String) fileds[i];
	            Object filedValue = getFieldValueByName(filedName, object);
	            if (filedValue != null) {
	                update.set(filedName, filedValue);
	            }
	        }
	        return update;
	    }
	
	    /***
	     * 获取对象属性返回字符串数组
	     * @param o
	     * @return
	     */
	    private static String[] getFiledName(Object o) {
	        Field[] fields = o.getClass().getDeclaredFields();
	        String[] fieldNames = new String[fields.length];
	
	        for (int i = 0; i < fields.length; ++i) {
	            fieldNames[i] = fields[i].getName();
	        }
	
	        return fieldNames;
	    }
	
	    /***
	     * 根据属性获取对象属性值
	     * @param fieldName
	     * @param o
	     * @return
	     */
	    private static Object getFieldValueByName(String fieldName, Object o) {
	        try {
	            String e = fieldName.substring(0, 1).toUpperCase();
	            String getter = "get" + e + fieldName.substring(1);
	            Method method = o.getClass().getMethod(getter, new Class[0]);
	            return method.invoke(o, new Object[0]);
	        } catch (Exception var6) {
	            return null;
	        }
	    }
	
	    public static String MD5Encoder(String source) {
	        String s = null;
	
	        try {
	            MessageDigest md = MessageDigest.getInstance("MD5");
	            md.update(source.getBytes("UTF-8"));
	            byte tmp[] = md.digest();
	            char str[] = new char[16 * 2];
	            int k = 0;
	            for (int i = 0; i < 16; i++) {
	                byte byte0 = tmp[i];
	                str[k++] = HEXDIGITS[byte0 >>> 4 & 0xf];
	
	                str[k++] = HEXDIGITS[byte0 & 0xf];
	            }
	            s = new String(str);
	
	        } catch (Exception e) {
	            throw new IllegalArgumentException(e);
	        }
	        return s;
	    }
	}

#### BookMongoDbDao.java

	package com.example.demo.dao;
	
	import com.example.demo.model.UserTest;
	import org.springframework.stereotype.Component;
	
	@Component
	public class BookMongoDbDao extends MongoDbDao<UserTest> {
	    @Override
	    protected Class<UserTest> getEntityClass() {
	        return UserTest.class;
	    }
	}


### DemoApplicationTests.java

	package com.example.demo;
	
	import com.example.demo.dao.BookMongoDbDao;
	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.boot.test.context.SpringBootTest;
	import org.springframework.test.context.junit4.SpringRunner;
	
	@RunWith(SpringRunner.class)
	@SpringBootTest
	public class DemoApplicationTests {
	
	    @Autowired
	    private BookMongoDbDao bookMongoDbDao;
	
	    @Test
	    public void testMongo() {
	        String info = bookMongoDbDao.getFileInfo("1265432345");
	        System.out.println(info);
	    }
	
	}

### 错误

启动单元测试，报错如下：

	Caused by: org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'bookMongoDbDao': Unsatisfied dependency expressed through field 'gridFSBucket'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'com.mongodb.client.gridfs.GridFSBucket' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}

报错说：bookMongoDbDao 内的 gridFSBucket 缺少依赖，没有实例化  
我理解为配置文件内 gridFSBucket 相关的配置没有设置，所以 Spring 无法实例化，但是检查配置文件没有发现问题  
今天上午我突然心有所悟，在旧代码中搜索了下 GridFSBucket，然后发现有代码没有拷贝过来，o(╯□╰)o  
com.example.demo.dao 文件夹下增加文件 MongoConf.java，问题解决

### MongoConf.java

	package com.example.demo.dao;
	
	import com.mongodb.client.MongoDatabase;
	import com.mongodb.client.gridfs.GridFSBucket;
	import com.mongodb.client.gridfs.GridFSBuckets;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.context.annotation.Bean;
	import org.springframework.context.annotation.Configuration;
	import org.springframework.data.mongodb.MongoDbFactory;
	
	@Configuration
	public class MongoConf {
	
	    @Autowired
	    private MongoDbFactory mongoDbFactory;
	
	    @Bean
	    public GridFSBucket getGridFSBuckets() {
	        MongoDatabase db = mongoDbFactory.getDb();
	        return GridFSBuckets.create(db);
	    }
	}

## SpringMVC

### pom.xml

	<?xml version="1.0" encoding="UTF-8"?>
	
	<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
	    <modelVersion>4.0.0</modelVersion>
	
	    <groupId>my.test</groupId>
	    <artifactId>SpringMVCDemo</artifactId>
	    <version>1.0-SNAPSHOT</version>
	    <packaging>war</packaging>
	
	    <name>SpringMVCDemo</name>
	
	    <properties>
	        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>
	        <maven.compiler.source>1.8</maven.compiler.source>
	        <maven.compiler.target>1.8</maven.compiler.target>
	        <spring.version>4.2.6.RELEASE</spring.version>
	    </properties>
	
	    <dependencies>
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-web</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-webmvc</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework</groupId>
	            <artifactId>spring-test</artifactId>
	            <version>${spring.version}</version>
	        </dependency>
	        <dependency>
	            <groupId>org.springframework.data</groupId>
	            <artifactId>spring-data-mongodb</artifactId>
	            <version>2.0.8.RELEASE</version>
	        </dependency>
	
	        <dependency>
	            <groupId>com.google.code.gson</groupId>
	            <artifactId>gson</artifactId>
	            <version>2.8.0</version>
	        </dependency>
	
	        <dependency>
	            <groupId>junit</groupId>
	            <artifactId>junit</artifactId>
	            <version>4.11</version>
	            <scope>test</scope>
	        </dependency>
	        <dependency>
	            <groupId>javax.servlet</groupId>
	            <artifactId>javax.servlet-api</artifactId>
	            <version>3.1.0</version>
	            <scope>provided</scope>
	        </dependency>
	    </dependencies>
	
	    <build>
	        <resources>
	            <resource>
	                <directory>src/main/resources/</directory>
	                <includes>
	                    <include>*.xml</include>
	                    <include>*.properties</include>
	                    <include>*.yml</include>
	                </includes>
	                <filtering>true</filtering>
	            </resource>
	        </resources>
	        <plugins>
	            <plugin>
	                <groupId>org.apache.maven.plugins</groupId>
	                <artifactId>maven-compiler-plugin</artifactId>
	                <version>2.0.2</version>
	                <configuration>
	                    <source>1.8</source>
	                    <target>1.8</target>
	                </configuration>
	            </plugin>
	        </plugins>
	    </build>
	</project>

### mongodb.properties

	mongo.host=192.168.3.187
	mongo.port=27017
	mongo.credentials=lin:123456@admin
	mongo.db=filetest

### mongodb-context.xml

之前卡在了这里，大部分文章都是 1.0 的配置，没有找到 2.0 配置的文章  
上午，我抛开他人文章的包袱，对照着 spring-mongo-2.0.xsd 内的描述，报错一个对象配置一个对象，就配置成这样了

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xmlns:context="http://www.springframework.org/schema/context"
	       xmlns:mongo="http://www.springframework.org/schema/data/mongo"
	       xsi:schemaLocation="http://www.springframework.org/schema/context
	          http://www.springframework.org/schema/context/spring-context-3.0.xsd
	          http://www.springframework.org/schema/data/mongo
	          http://www.springframework.org/schema/data/mongo/spring-mongo-2.0.xsd
	          http://www.springframework.org/schema/beans
	          http://www.springframework.org/schema/beans/spring-beans-3.0.xsd">
	
	    <!-- 加载mongodb的属性配置文件 -->
	    <context:property-placeholder location="classpath:mongodb.properties"/>
	
	    <mongo:mongo-client id="mongoClient" host="${mongo.host}" port="${mongo.port}" credentials="${mongo.credentials}" replica-set=""/>
	    <mongo:db-factory id="dbFactory" dbname="${mongo.db}" mongo-ref="mongoClient"/>
	    <mongo:template id="mongoTemplate" db-factory-ref="dbFactory" write-concern="NORMAL"/>
	    <mongo:mapping-converter id="mappingMongoConverter" db-factory-ref="dbFactory"/>
	    <mongo:gridFsTemplate id="gridFsTemplate" bucket="" db-factory-ref="dbFactory" converter-ref="mappingMongoConverter"/>
	</beans>

### applicationContext.xml

	<?xml version="1.0" encoding="UTF-8"?>
	<beans xmlns="http://www.springframework.org/schema/beans"
	       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
	       xmlns:context="http://www.springframework.org/schema/context"
	       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
	                           http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd">
	    <context:component-scan base-package="my.test"/>
	
	    <!-- 导入mongodb的配置文件 -->
	    <import resource="mongodb-context.xml"/>
	</beans>

### my.test.dao 同 SpringBoot 的 com.example.demo.dao

### my.test.model 同 SpringBoot 的 com.example.demo.model

### MongoTest.java

	package my.test.dao;
	
	import org.junit.Test;
	import org.junit.runner.RunWith;
	import org.springframework.beans.factory.annotation.Autowired;
	import org.springframework.test.context.ContextConfiguration;
	import org.springframework.test.context.junit4.SpringJUnit4ClassRunner;
	
	@RunWith(SpringJUnit4ClassRunner.class)
	@ContextConfiguration("classpath:applicationContext.xml")
	public class MongoTest {
	
	    @Autowired
	    private BookMongoDbDao bookMongoDbDao;
	
	    @Test
	    public void findTestByName() {
	        String info = bookMongoDbDao.getFileInfo("1265432345");
	        System.out.println(info);
	    }
	}

### 错误

启动单元测试，报错如下：

	Caused by: java.lang.ClassNotFoundException: org.springframework.core.MethodClassKey

搜索上述错误，找到[这篇文章](https://yq.aliyun.com/articles/269468)，该文章说：把 spring-tx 的版本号去掉，即改成如下：
	
	<dependency>
	    <groupId>org.springframework</groupId>
	    <artifactId>spring-tx</artifactId>
	    <!--<version>4.3.2.RELEASE</version>-->
	</dependency>

我看我都没有引入这个包啊，就在 pom.xml 引了一下，问题解决