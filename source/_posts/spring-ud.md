---
title: Spring文件上传下载
tags: Spring
categories: Spring
keywords: Spring
comments: false
---

**注：本文原链接**：[https://www.cnblogs.com/chloneda/p/spring-ud.html](https://www.cnblogs.com/chloneda/p/spring-ud.html)



# 前言

开发人员多少都会遇到文件的上传下载，特别是Java的Spring框架。那么我们能不能实现一些通用工具类，实现文件上传和下载的功能，可以有效地避免重复开发代码。

因此，我在网上查找了相关资料，并在此基础上加以改进,支持文件上传及下载功能，而且代码非常精简。如有问题，欢迎大家指正！



# 文件上传

文件上传及下载需要两个依赖：

```bash
<dependency>
    <groupId>org.springframework</groupId>
    <artifactId>spring-web</artifactId>
    <version>5.1.6.RELEASE</version>
</dependency>
<!--tomcat中也有servlet-api包，避免冲突添加<scope>provided</scope>-->
<dependency>
    <groupId>javax.servlet</groupId>
    <artifactId>servlet-api</artifactId>
    <version>2.5</version>
    <scope>provided</scope>
</dependency>
```



Spring文件上传一般从请求中获取文件对象，或者直接上传文件获取文件对象。这里提供两个方法，可以满足以上两种情形！当然，如果想自定义文件上传路径，也可以在这两个方法的基础上再增加一个文件路径参数，具体还是看大家的需求情况吧！

```bash
package com.chloneda.utils;

import org.springframework.web.multipart.MultipartFile;
import org.springframework.web.multipart.MultipartHttpServletRequest;
import org.springframework.web.multipart.commons.CommonsMultipartResolver;

import javax.servlet.http.HttpServletRequest;
import java.io.File;
import java.io.IOException;
import java.util.*;

/**
 * @author chloneda
 * @description: 文件上传工具类
 */
public class RequestUtils {

    /**
     * 获取请求中的文件并上传
     *
     * @param request    请求对象
     * @param useNewName 是否使用原文件名称
     */
    public static void uploadFileFromRequest(HttpServletRequest request, boolean useNewName)
            throws IOException {
        CommonsMultipartResolver multipartResolver =
                new CommonsMultipartResolver(request.getSession().getServletContext());
        /** 判断 request 是否有文件上传 */
        if (multipartResolver.isMultipart(request)) {
            /** 向上转型获取更多功能 */
            MultipartHttpServletRequest multiRequest = (MultipartHttpServletRequest) request;
            Iterator<String> iter = multiRequest.getFileNames();
            while (iter.hasNext()) {
                MultipartFile multipartFile = multiRequest.getFile(iter.next());
                if (multipartFile.isEmpty()) {
                    System.out.println("文件未上传!");
                    continue;
                }
                uploadFileFromMultipartFile(multipartFile, request, useNewName);
            }
        }
    }

    /**
     * 获取文件并上传至指定目录
     *
     * @param file       上传的文件
     * @param useNewName 是否使用原文件名称
     * @throws IOException
     */
    public static void uploadFileFromMultipartFile(
            MultipartFile file, HttpServletRequest request, boolean useNewName) throws IOException {
        /** 获取服务器项目发布运行所在地址 */
        String uploadDir = request.getSession().getServletContext().getRealPath("/") + "upload";
        String originalName = file.getOriginalFilename();
        /** 使用时间缀解决文件重名问题 */
        if (useNewName) {
            long timestamp = System.currentTimeMillis();
            originalName = timestamp + "-" + originalName;
        }
        String destFilePath = uploadDir + File.separator + originalName;
        file.transferTo(new File(destFilePath));
    }

}

```



# 文件下载

文件下载需要知道文件所在路径，并通过文件流的响应方式返回给前端下载，具体代码如下：

```bash
package com.chloneda.utils;

import javax.servlet.http.HttpServletResponse;
import java.io.*;
import java.net.URLEncoder;
import java.nio.charset.StandardCharsets;

/**
 * @author chloneda
 * @description: 返回文件流至浏览器
 */
public class ResponseUtils {

    /**
     * 返回文件流至前端进行展示
     *
     * @param srcFilePath
     * @param contentType 展示contentType一般是image/jpeg、jpg、gif、png等
     * @param response
     */
    public static void responseFileForShow(
            HttpServletResponse response, String srcFilePath, String contentType) {
        File file = checkFile(srcFilePath, false);
        responseFile(response, file, contentType, false);
    }

    /**
     * 返回文件流至前端进行下载
     *
     * @param srcFilePath 返回文件的具体路径
     * @param response
     */
    public static void responseFileForLoad(HttpServletResponse response, String srcFilePath) {
        File file = checkFile(srcFilePath, false);
        responseFile(response, file, "application/octet-stream", true);
    }

    /**
     * 使用response返回文件流
     *
     * @param response    response进行流的返回
     * @param file        要返回的具体文件
     * @param contentType contentType类型
     * @param isLoad      true时下载，false时展示
     */
    private static void responseFile(
            HttpServletResponse response, File file, String contentType, boolean isLoad) {
        try (BufferedInputStream in = new BufferedInputStream(new FileInputStream(file));
             BufferedOutputStream out = new BufferedOutputStream(response.getOutputStream())) {
            response.setContentType(contentType);
            /** 如果是下载,指定文件名称 */
            if (isLoad) {
                String fileName = file.getName();
                /** 指定字符集解决下载文件名乱码 */
                response.setHeader("Content-disposition", "attachment;filename="
                        + URLEncoder.encode(fileName, StandardCharsets.UTF_8.name()));
            }
            byte[] buffer = new byte[8192];
            int count = 0;
            while ((count = in.read(buffer, 0, 8192)) != -1) {
                out.write(buffer, 0, count);
            }
            out.flush();
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private static File checkFile(String filePath, boolean create) {
        File file = new File(filePath);
        if (!file.exists()) {
            if (create) {
                file.mkdirs();
            } else {
                throw new IllegalArgumentException("文件不存在：{}！" + filePath);
            }
        }
        return file;
    }

}

```



# 小结

文件的上传下载是比较普遍的功能需求，这里通过请求响应的方式实现，可以满足日常的开发需求！



---
