[TOC]

# Consuming a Restful Web Service

### Consuming a restful web service

之前都是用postman调用接口，测试后端程序，现在师父让我用另外一个来代替。也就是用springboot建立一个服务端来代替postman实现它之前能实现的操作。

注意，这里不是用socket实现的。

老规矩，遇到springboot，先上官网教程。

https://spring.io/guides/gs/consuming-rest/

有些不是很懂，不太清楚具体怎么操作，瞎搞了一通又报错。于是，又找了一些资料。

https://blog.csdn.net/weixin_34392435/article/details/93950790?utm_source=app&app_version=4.12.0&code=app_1562916241&uLinkId=usr1mkqgl919blen

上面的和官方给的有些不一样，我准备先试一试上面的这一个版本。



## 之前的文件：

https://www.jianshu.com/p/4820e5f7729c



## 官网地址：

https://spring.io/guides/gs/consuming-rest/



## 初始代码分析：

- 用RestTemplate获取指定url返回的数据，并以绑定的数据类型转化。



```java
package com.example.consumingrest;

import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.CommandLineRunner;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.client.RestTemplateBuilder;
import org.springframework.context.annotation.Bean;
import org.springframework.web.client.RestTemplate;

@SpringBootApplication
public class ConsumingRestApplication {
private static final Logger log = LoggerFactory.getLogger(ConsumingRestApplication.class);

public static void main(String[] args) {
	SpringApplication.run(ConsumingRestApplication.class, args);
}

@Bean
public RestTemplate restTemplate(RestTemplateBuilder builder) {
	return builder.build();
}

@Bean
public CommandLineRunner run(RestTemplate restTemplate) throws Exception {
	return args -> {
		Quote quote = restTemplate.getForObject(
				"https://quoters.apps.pcfone.io/api/random", Quote.class);
		log.info(quote.toString());
	};
}
```
[^@SpringBootApplication]: 开启自动配置
[^LoggerFactory.getLogger(ConsumingRestApplication.class);]: 在控制台打印ConsumingRestApplication.class类的日志。
[^builder.build()]: 使用 RestTemplateBuilder.build() 代替 new RestTemplate()





- 写一个存数据的类（官网上分了两层）

第一层：

````java
package com.example.consumingrest;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Quote {

  private String type;
  private Value value;

  public Quote() {
  }

  public String getType() {
    return type;
  }

  public void setType(String type) {
    this.type = type;
  }

  public Value getValue() {
    return value;
  }

  public void setValue(Value value) {
    this.value = value;
  }

  @Override
  public String toString() {
    return "Quote{" +
        "type='" + type + '\'' +
        ", value=" + value +
        '}';
  }
}
````

[^@JsonIgnoreProperties(ignoreUnknown = true)]: class中忽视所有没写的属性（properties）



第二层：

````java
package com.example.consumingrest;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Value {

  private Long id;
  private String quote;

  public Value() {
  }

  public Long getId() {
    return this.id;
  }

  public String getQuote() {
    return this.quote;
  }

  public void setId(Long id) {
    this.id = id;
  }

  public void setQuote(String quote) {
    this.quote = quote;
  }

  @Override
  public String toString() {
    return "Value{" +
        "id=" + id +
        ", quote='" + quote + '\'' +
        '}';
  }
}
````



## 要改的部分：

- url

url改成我要调用的，而不是官网给的default版本。

- 返回的数据格式

由于这里的schema，输出的数据格式时json。json文件中的键值对改成我要调用的url的输出的格式。



## 我的核心代码：

### 目标：实现 accessing-data-mysql-0.0.1-SNAPSHOT.jar中增删改查的postman所做的事。

参考：https://blog.csdn.net/csdn86868686888/article/details/103791210



### 以get方法显示所有用户信息的功能。

由于显示所有用户信息，所以现实的是一个List，也就是一个数组。

```java
@Bean
public CommandLineRunner run(RestTemplate restTemplate) throws Exception {
   return args -> {
      Quote[] quotes = restTemplate.getForObject(
            "http://127.0.0.1:18001/user/all", Quote[].class);
      assert quotes != null;
      log.info(Arrays.toString(quotes));
   };
}
```



写一个存数据的类（官网上给了两层来存数据，我这边只用一层）

由于显示所有用户信息，所以显示的是一个List，也就是一个数组。

```java
package com.example.consumingrest;

import com.fasterxml.jackson.annotation.JsonIgnoreProperties;

import java.util.ArrayList;
import java.util.List;

@JsonIgnoreProperties(ignoreUnknown = true)
public class Quote {
    private Integer id;
    private String name;
    private String email;

    public Quote() {
    }


    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getEmail() {
        return email;
    }

    public void setEmail(String email) {
        this.email = email;
    }


    @Override
    public String toString() {
        return "Quote{" +
                "id=" + id +
                "name=" + name +
                "email=" + email + '\'' +
                '}';
    }
}
```

[^@JsonIgnoreProperties(ignoreUnknown = true)]: class中忽视所有没写的属性（properties）





### 以post方法添加用户信息

````java
//	private static final Logger log = LoggerFactory.getLogger(ConsumingRestApplication.class);
````

```java
@Bean
public String run(RestTemplate restTemplate) throws Exception {
   String url = "http://127.0.0.1:18001/user/add";

   //headers
   HttpHeaders requestHeaders = new HttpHeaders();
   requestHeaders.setContentType(MediaType.APPLICATION_JSON);
   //body
   //MultiValueMap<String, String> requestBody = new LinkedMultiValueMap<>();
   Map<String, String> requestBody = new HashMap();

   requestBody.put("id", "2");
   requestBody.put("name", "2");
   requestBody.put("email", "2");


   //HttpEntity
   HttpEntity<Map<String, String>> requestEntity = new HttpEntity<>(requestBody, requestHeaders);
   //HttpEntity<MultiValueMap> requestEntity = new HttpEntity<MultiValueMap>(requestBody, requestHeaders);


   String s = restTemplate.postForObject(url, requestEntity, String.class);
   return s;
   };
```

[^requestHeaders.setContentType(MediaType.APPLICATION_JSON);]: 设置请求的数据类型
[^Map&lt;String, String&gt; requestBody = new HashMap();]: 以hashmap来存输入的Body
[^requestBody.put(&quot;id&quot;, &quot;2&quot;);]: 加入键值对





## 今日小tip:

一个端口上只能最多存在一个进程，所以要在服务器上用 consuming-rest-0.0.1-SNAPSHOT.jar来测试 accessing-data-mysql-0.0.1-SNAPSHOT.jar时，要用不同的端口。













