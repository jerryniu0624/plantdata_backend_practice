# Springboot-File-Upload-Download



## 文件结构

![image-20210827140306727](Springboot文件上传下载.assets/image-20210827140306727.png)



## StorageService

```java
package uploadingfiles.storage;

import org.springframework.core.io.Resource;
import org.springframework.web.multipart.MultipartFile;

import java.nio.file.Path;
import java.util.stream.Stream;

public interface StorageService {

   void init();

   void store(MultipartFile file);

   Stream<Path> loadAll();

   Path load(String filename);

   Resource loadAsResource(String filename);

   void deleteAll();

}
```

Resource:

Interface for a resource descriptor that abstracts from the actual type of underlying resource, such as a file or class path resource.
An InputStream can be opened for every resource if it exists in physical form, but a URL or File handle can just be returned for certain resources. The actual behavior is implementation-specific.



## StorageException

```java
package uploadingfiles.storage;

public class StorageException extends RuntimeException {

   public StorageException(String message) {
      super(message);
   }

   public StorageException(String message, Throwable cause) {
      super(message, cause);
   }
}
```

Throwable:

The Throwable class is the superclass of all errors and exceptions in the Java language. Only objects that are instances of this class (or one of its subclasses) are thrown by the Java Virtual Machine or can be thrown by the Java throw statement. Similarly, only this class or one of its subclasses can be the argument type in a catch clause. For the purposes of compile-time checking of exceptions, Throwable and any subclass of Throwable that is not also a subclass of either RuntimeException or Error are regarded as checked exceptions.



## StorageFileNotFoundException

```java
package uploadingfiles.storage;

public class StorageFileNotFoundException extends StorageException {

   public StorageFileNotFoundException(String message) {
      super(message);
   }

   public StorageFileNotFoundException(String message, Throwable cause) {
      super(message, cause);
   }
}
```



## StorageProperties

```java
package uploadingfiles.storage;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties("storage")
public class StorageProperties {

   /**
    * Folder location for storing files
    */
   private String location = "upload-dir";

   public String getLocation() {
      return location;
   }

   public void setLocation(String location) {
      this.location = location;
   }

}
```

ConfigurationProperties:

Annotation for externalized configuration. Add this to a class definition or a @Bean method in a @Configuration class if you want to bind and validate some external Properties (e.g. from a .properties file).



## FileUploadController

```java
package uploadingfiles;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.core.io.Resource;
import org.springframework.http.HttpHeaders;
import org.springframework.http.MediaType;
import org.springframework.http.ResponseEntity;
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.*;
import org.springframework.web.multipart.MultipartFile;
import uploadingfiles.storage.StorageFileNotFoundException;
import uploadingfiles.storage.StorageService;

import java.io.File;
import java.io.IOException;

@Controller
public class FileUploadController {

    private final StorageService storageService;

    // 注入 storageService bean
    @Autowired
    public FileUploadController(StorageService storageService) {

        this.storageService = storageService;
    }

    // 文件下载
    // 正则表达式匹配, 语法: {varName:regex} 前面式变量名，后面式表达式
    // 匹配出现过一次或多次.的字符串 如: "xyz.png"
    @GetMapping(path = "/files/{filename:.+}")
    @ResponseBody
    public ResponseEntity<Resource> serveFile(@PathVariable String filename) {
        // 根据文件名读取文件
        Resource file = storageService.loadAsResource(filename);
        // @ResponseBody 用于直接返回结果(自动装配)
        // ResponseEntity 可以定义返回的 HttpHeaders 和 HttpStatus (手动装配)
        // ResponseEntity.ok 相当于设置 HttpStatus.OK (200)
        // CONTENT_DISPOSITION 该 标志将通知浏览器启动下载
        System.out.print(filename);
        return ResponseEntity.ok().header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + file.getFilename() + "\"").body(file);
    }


    @PostMapping(value = "/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE,
            produces = MediaType.APPLICATION_JSON_VALUE)
    @ResponseBody
    public String upload(MultipartFile file1) {
        if (file1 == null) {
            return null;
        }
        String fileName = file1.getOriginalFilename();
        System.out.println("文件名：" + fileName);
        String filePath = "/opt/niuyuanzhuo";
        File dest = new File(filePath, fileName);
        try {
            file1.transferTo(dest);
            System.out.println("上传成功！");
        } catch (IOException e) {
            e.printStackTrace();
            return "error";
        }
        return "success";
    }

    @GetMapping(value = "/download", consumes = MediaType.MULTIPART_FORM_DATA_VALUE,
            produces = MediaType.APPLICATION_JSON_VALUE)
    @ResponseBody
    public String download(MultipartFile file1, String filePath) {

        if (file1 == null) {
            return "下载失败，未选择文件";
        }
        String fileName = file1.getOriginalFilename();
        System.out.println("文件名：" + fileName);
        File dest = new File(filePath, fileName);
        try {
            file1.transferTo(dest);
            System.out.println("下载成功！");
        } catch (IOException e) {
            System.out.println("下载异常！" + e);
            return "error";
        }
        return "success";
    }

    // 统一处理该 controller 异常
    @ExceptionHandler(StorageFileNotFoundException.class)
    public ResponseEntity<?> handleStorageFileNotFound(StorageFileNotFoundException exc) {
        return ResponseEntity.notFound().build();
    }

}
```

ResponseEntity:

Extension of HttpEntity that adds an HttpStatus status code. Used in RestTemplate as well as in @Controller methods.

PathVariable:

Annotation which indicates that a method parameter should be bound to a URI template variable. Supported for RequestMapping annotated handler methods.
If the method parameter is Map<String, String> then the map is populated with all path variable names and values.



## UploadingFilesApplication

```java
package uploadingfiles;

import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import uploadingfiles.storage.StorageProperties;
import uploadingfiles.storage.StorageService;

@SpringBootApplication
@EnableConfigurationProperties(StorageProperties.class)
public class UploadingFilesApplication {

    public static void main(String[] args) {
        SpringApplication.run(UploadingFilesApplication.class, args);
    }

    @Bean
    CommandLineRunner init(StorageService storageService) {
        return (args) -> {
            storageService.deleteAll();
            storageService.init();
        };
    }
}
```



### 下载

先上代码

```java
@GetMapping(path = "/files/{filename:.+}")
@ResponseBody
public ResponseEntity<Resource> serveFile(@PathVariable String filename) {
    // 根据文件名读取文件
    Resource file = storageService.loadAsResource(filename);
    // @ResponseBody 用于直接返回结果(自动装配)
    // ResponseEntity 可以定义返回的 HttpHeaders 和 HttpStatus (手动装配)
    // ResponseEntity.ok 相当于设置 HttpStatus.OK (200)
    // CONTENT_DISPOSITION 该 标志将通知浏览器启动下载
    System.out.print(filename);
    return ResponseEntity.ok().header(HttpHeaders.CONTENT_DISPOSITION, "attachment; filename=\"" + file.getFilename() + "\"").body(file);
}
```

PathVariable：

Annotation which indicates that a method parameter should be bound to a URI template variable. Supported for RequestMapping annotated handler methods.
If the method parameter is Map<String, String> then the map is populated with all path variable names and values.



由于惯性思维，以为在url调用接口时**{filename:.+}**这部分也要原封不动的抄上去。经过师傅的提醒，发现这个位置写的是文件名。

然而，报错：404    not found ，而且这个错误没通过json或text表现出来，而是在postman上很小的一块地方标注，迷惑了我好久。

要解决这个问题，师父让我先在拿文件的代码处**断点**。也就是这一条。

​    Resource file = storageService.loadAsResource(filename);

再通过“**ctrl+左键**”一层层看是从哪里下载的文件，（虽然我还是没太理解），最后发现是在“dir_upload"中下载（好像是）。别以为发现这个就算完事了。不能先放文件再运行程序，否则程序运行时有一步会清空该文件夹。所以**要在IDEA显示运行完后**，已经有进程时，再放文件，然后在Postman中Get。

总之，这个degub的思路很重要。





### 上传



```java
@PostMapping(value = "/upload", consumes = MediaType.MULTIPART_FORM_DATA_VALUE,
        produces = MediaType.APPLICATION_JSON_VALUE)
public String upload(MultipartFile file1) {
    if (file1 == null) {
        return null;
    }
    String fileName = file1.getOriginalFilename();
    System.out.println("文件名：" + fileName);
    String filePath = "/opt/niuyuanzhuo";
    File dest = new File(filePath, fileName);
    try {
        file1.transferTo(dest);
        System.out.println("上传成功！");
    } catch (IOException e) {
        e.printStackTrace();
        return "error";
    }
    return "succeess";
}
```

这串代码的雏形是csdn上某网站抄的，但是有几处不够严谨。

- 没写consumes = MediaType.MULTIPART_FORM_DATA_VALUE, produces = MediaType.APPLICATION_JSON_VALUE，导致postan上输入输出格式不好。
-  File dest = new File(filePath, fileName);原来的“，”是加号，“ctrl+左键”点进去后才发现要逗号。
- 异常处理时，要e.printStackTrace();，否则不知道哪里错了。
- 上传地址要这样写：String filePath = "/opt/niuyuanzhuo";。

补充：

上传地址之所以能这么写，是因为我的程序时放在了我们要上传文件的目的服务器上运行了。在我的另一篇文章中，就不会采用这种权宜之计。



## 今天学的小技巧

- 在构建jar包时，输入jar包名，可以通过“tab"键一步输出。
- 若是java -jar jar包名线上运行jar包，可以通过“ctrl+C"退出进程。
- 要看代码哪里出问题，例外报错“error"屁用没有，要打印日志，方式下面链接有。



更加详细的点击链接。

https://blog.csdn.net/wx1528159409/article/details/90608525?ops_request_misc=%257B%2522request%255Fid%2522%253A%252216269347%207716780265477554%2522%252C%2522scm%2522%253A%252220140713.130102334..%2522%257D&request_id=162693477716780265477554&biz_id=0&utm_medium=distribute.pc_search_result.none-task-blog-2~all~sobaiduend~default-1-90608525.first_rank_v2_pc_rank_v29&utm_term=nohup%E8%BF%90%E8%A1%8Cjar%E5%8C%85&spm=1018.2226.3001.4187





