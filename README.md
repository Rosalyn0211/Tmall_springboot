# 天猫整站Springboot 

## 后台-分类管理

### 查询

#### Category.java

```java
package com.how2java.tmall.pojo;
 
import javax.persistence.Column;
import javax.persistence.Entity;
import javax.persistence.GeneratedValue;
import javax.persistence.GenerationType;
import javax.persistence.Id;
import javax.persistence.Table;
 
import com.fasterxml.jackson.annotation.JsonIgnoreProperties;
 
@Entity
@Table(name = "category")
@JsonIgnoreProperties({ "handler","hibernateLazyInitializer" })
 
public class Category {
    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    @Column(name = "id")   
    int id;
     
    String name;
     
    public int getId() {
        return id;
    }
    public void setId(int id) {
        this.id = id;
    }
     
    public String getName() {
        return name;
    }
    public void setName(String name) {
        this.name = name;
    }
}
```

#### @Entity作用

@Entity 表示这是一个实体类

Java实体类的理解：

- Java实体类其实就是俗称的POJO,这种类一般不实现特殊框架下的接口，在程序中仅作为数据容器用来持久化存储数据用的。
- 属性类，通常定义在model层里面。
- 一般的实体类对应一个数据表，其中的属性对应数据表中的字段。
  - 对对象实体的封装，体现OO思想。
  - 把相关信息用一个实体类封装后，我们在程序中可以把实体类作为参数传递，更加方便。
  - 属性可以对字段定义和状态进行判断和过滤。
- 把相关信息用一个实体类封装后，我们在程序中可以把实体类作为参数传递，更加方便。
- 简单说就是为了让程序员在对数据库操作的时候不用写SQL语句。
- 一个数据库表生成一个类。
- 实体类中都是实例对象,实例对象在jvm的堆区中开辟了一个该对象引用空间，并且让该引用指向某个实例,类声明只是在jvm的栈去中开辟了一个该对象引用,没有让该引用做任何指向。

##### @Table(name = "category") 作用

表示对应的表名是 category

##### @JsonIgnoreProperties({ "handler","hibernateLazyInitializer" })作用

因为是做前后端分离，而前后端数据交互用的是 json 格式。 那么 Category 对象就会被转换为 json 数据。 而本项目使用 jpa 来做实体类的持久化，jpa 默认会使用 hibernate, 在 jpa 工作过程中，就会创造代理类来继承 Category ，并添加 handler 和 hibernateLazyInitializer 这两个无须 json 化的属性，所以这里需要用 JsonIgnoreProperties 把这两个属性忽略掉。

#####  @GeneratedValue(strategy = GenerationType.IDENTITY)作用

JPA通用策略生成器 通过annotation来映射hibernate实体的，基于annotation的hibernate主键标识为@Id。

其生成规则是由@GeneratedValue设定的。这里的@id和@GeneratedValue都是JPA的标准用法。

其中GenerationType: 

JPA提供的四种标准用法为TABLE，SEQUENCE，IDENTITY，AUTO。

- TABLE：使用一个特定的数据库表格来保存主键。 
- SEQUENCE：根据底层数据库的序列来生成主键，条件是数据库支持序列。 
- IDENTITY：主键由数据库自动生成（主要是自动增长型）。
- AUTO：主键由程序控制。 

#### CategoryDAO.java

```java
package com.how2java.tmall.dao;
  
import org.springframework.data.jpa.repository.JpaRepository;
 
import com.how2java.tmall.pojo.Category;
 
public interface CategoryDAO extends JpaRepository<Category,Integer>{
 
}
```

CategoryDAO 类继承了 JpaRepository，就提供了CRUD和分页的各种常见功能。 这就是采用 JPA 方便的地方。

JpaRepository<Category,Integer>中的Integer是怎么来的？

查看JpaRepository的源文件后发现它是指ID类型，ID的类型是int，JpaRepository<>里对应就是哪一个实体类跟实体类ID对应的类型。

#### CategoryService.java

```java
package com.how2java.tmall.service;
 
import java.util.List;
 
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.domain.Sort;
import org.springframework.stereotype.Service;
 
import com.how2java.tmall.dao.CategoryDAO;
import com.how2java.tmall.pojo.Category;
 
@Service
public class CategoryService {
    @Autowired CategoryDAO categoryDAO;
 
    public List<Category> list() {
        Sort sort = new Sort(Sort.Direction.DESC, "id");
        return categoryDAO.findAll(sort);
    }
}
```

注： 这里抛弃了 CategoryService 接口 加上 CategoryService 实现类的这种累赘的写法，而是直接使用 CategoryService 作为实现类来做。

##### @Service的作用

标记这个类是 Service类

##### @Autowired CategoryDAO categoryDAO的作用

自动装配上个步骤的 CategoryDAO 对象

#### AdminPageController.java

后台管理页面跳转专用控制器

```java
package com.how2java.tmall.web;
 
import org.springframework.stereotype.Controller;
import org.springframework.web.bind.annotation.GetMapping;
 
@Controller
public class AdminPageController {
    @GetMapping(value="/admin")
    public String admin(){
        return "redirect:admin_category_list";
    }
    @GetMapping(value="/admin_category_list")
    public String listCategory(){
        return "admin/listCategory";
    }
}
```

##### @Controller的作用

表示这是一个控制器。

 ##### @GetMapping的作用

访问地址 admin,就会客户端跳转到 admin_category_list去。

访问地址 admin_category_list 就会访问 admin/listCategory.html 文件。

#### CategoryController.java

 RESTFUL 服务器控制器

```java
package com.how2java.tmall.web;
 
import com.how2java.tmall.pojo.Category;
import com.how2java.tmall.service.CategoryService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;
 
import java.util.List;
  
@RestController
public class CategoryController {
    @Autowired CategoryService categoryService;
     
    @GetMapping("/categories")
    public List<Category> list() throws Exception {
        return categoryService.list();
    }
}
```

##### @RestController的作用

表示这是一个控制器，并且对每个方法的返回值都会直接转换为 json 数据格式。

对于categories 访问，会获取所有的 Category对象集合，并返回这个集合。 因为是声明为 @RestController， 所以这个集合，又会被自动转换为 JSON数组抛给浏览器。

##### @Controller和@RestController的区别？

@RestController相当于@Controller+@ResponseBody合在一起的作用

使用@RestController这个注解，就不能返回jsp,html页面，视图解析器无法解析jsp,html页面

##### 什么是 REST

REST（REpresentational State Transfer）,它指的是一组架构约束条件和原则。满足这些约束条件和原则的应用程序或设计就是 RESTful 的。

1、每一个 URI 代表一种资源

2、客户端和服务端之间，传递这种资源的某种表现层

3、客户端通过四个 HTTP 动词（四个表示操作方式的动词：GET、POST、PUT、DELETE），对服务端资源进行操作，它们分别对应四种基本操作：GET 用来获取资源，POST 用来新建资源（也可以用于更新资源），PUT 用来更新资源，DELETE 用来删除资源。实现“表现层状态转化”

#### Application.java

启动类，代替自动生成的TmallSpringbootApplication.java

```java
package com.how2java.tmall;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
@SpringBootApplication
public class Application {
    public static void main(String[] args) {
        SpringApplication.run(Application.class, args);    
    }
}
```

#### CORSConfiguration.java

配置类，用于允许所有的请求都跨域。
因为是二次请求，第一次是获取 html 页面， 第二次通过 html 页面上的 js 代码异步获取数据，一旦部署到服务器就容易面临跨域请求问题，所以允许所有访问都跨域，就不会出现通过 ajax 获取数据获取不到的问题了。

```java
package com.how2java.tmall.config;
 
import org.springframework.context.annotation.Configuration;
import org.springframework.web.servlet.config.annotation.CorsRegistry;
import org.springframework.web.servlet.config.annotation.WebMvcConfigurerAdapter;
 
@Configuration
public class CORSConfiguration extends WebMvcConfigurerAdapter{
    @Override
    public void addCorsMappings(CorsRegistry registry) {
        //所有请求都允许跨域
        registry.addMapping("/**")
                .allowedOrigins("*")
                .allowedMethods("*")
                .allowedHeaders("*");
    }
}
```

##### 为什么会出现跨域

跨域问题来源于JavaScript的同源策略，即只有 协议+主机名+端口号 (如存在)相同，则允许相互访问。也就是说JavaScript只能访问和操作自己域下的资源，不能访问和操作其他域下的资源。跨域问题是针对JS和ajax的，html本身没有跨域问题，比如a标签、script标签、甚至form标签（可以直接跨域发送数据并接收数据）等。

#### GloabalExceptionHandler.java

异常处理，主要是在处理删除父类信息的时候，因为外键约束的存在，而导致违反约束。

```java
package com.how2java.tmall.exception;
 
import javax.servlet.http.HttpServletRequest;
 
import org.springframework.web.bind.annotation.ControllerAdvice;
import org.springframework.web.bind.annotation.ExceptionHandler;
import org.springframework.web.bind.annotation.RestController;
 
@RestController
@ControllerAdvice
public class GloabalExceptionHandler {
    @ExceptionHandler(value = Exception.class)
    public String defaultErrorHandler(HttpServletRequest req, Exception e) throws Exception {
        e.printStackTrace();
        Class constraintViolationException = Class.forName("org.hibernate.exception.ConstraintViolationException");
        if(null!=e.getCause()  && constraintViolationException==e.getCause().getClass()) {
            return "违反了约束，多半是外键约束";
        }
        return e.getMessage();
    }
 
}
```

##### @ControllerAdvice的作用

@ControllerAdvice是一个增强的 Controller。使用这个 Controller ，可以实现三个方面的功能：

1. 全局异常处理

   @ExceptionHandler 注解用来指明异常的处理类型，即如果这里指定为 NullpointerException，则数组越界异常就不会进到这个方法中来。

2. 全局数据绑定

3. 全局数据预处理

#### application.properties

springboot 配置文件

```x
spring.datasource.url=jdbc:mysql://127.0.0.1:3306/tmall_springboot?characterEncoding=UTF-8
spring.datasource.username=root
spring.datasource.password=admin
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.jpa.hibernate.ddl-auto = none
```

分别是数据库访问地址，账号密码，驱动以及表结构自动生成策略(none)。

使用springdata jpa时，如果希望实体类发生更改而数据库表做出相应的更改且不破坏数据库现有的数据，要将spring.jpa.hibernate.ddl-auto属性值设置为update，同时需要在实体类上的@Table注解中加上catalog属性，值为数据库名。

```xml
spring.thymeleaf.mode=LEGACYHTML5
spring.thymeleaf.encoding=UTF-8
spring.thymeleaf.content-type=text/html
spring.thymeleaf.cache=false
```

使用 thymeleaf 作为视图，这个是springboot 官方推荐视图，它的好处是可以是纯 html 。
其中LEGACYHTML5表示经典html5模式，即允许非严格的html出现，元素少点什么也可以编译通过。
cache=false 表示不要缓存，以免在开发过程中因为停留在缓存而给开发人员带来困扰。

```xml
server.context-path=/tmall_springboot
```

上下文地址为 tmall_springboot, 所以访问的时候，都要加上这个，比如：

```xml
http://127.0.0.1:8080/tmall_springboot/admin
```

```xml
spring.http.multipart.maxFileSize=100Mb
spring.http.multipart.maxRequestSize=100Mb
```

设置上传文件大小，默认只有1M。

```xml
spring.jpa.hibernate.naming.physical-strategy=org.hibernate.boot.model.naming.PhysicalNamingStrategyStandardImpl
```

jpa对实体类的默认字段会把驼峰命名的属性，转换为字段名的时候自动加上下划线。 这个配置的作用就是去掉下划线
比如属性名称是 createDate, jpa 默认转换为字段名 create_Date。 有了这个配置之后，就会转换为同名字段 createDate

```xml
spring.jpa.show-sql=true
```

显示 hibernate 执行的sql语句。 这个在上线之后，应该是关掉的，因为大量的控制台输出会严重影响系统性能。 但是，因为本项目会和 redis 和 es 整合，打印 sql 语句的目的是为了观察缓存是否起效果。

#### 静态资源

静态资源为什么不放在 static 目录下？ 一般说来，在约定里，springboot 的静态资源会在 static 目录下，但是我们是放在 webapp 目录下，为什么会这样呢？ 因为我们还要做上传图片的功能，如果是放在 static 下，上传后的图片就无法被访问，还是放在 webapp 下，上传后，能够立即被访问。

#### listCategory.html

获取数据和展示数据

##### 获取数据

```html
$(function(){
}
```

jquery代码，表示当整个html加载好了之后执行。

```html
var data4Vue = {
    uri:'categories',
    beans: []
};
```

vue用到的数据， uri表示访问哪个地址去获取数据，这里的值是 categories，和 CategoryController.java 相呼应

```htm
var vue = new Vue({
    el: '#workingArea',
    data: data4Vue,
```

创建Vue对象，el 表示和本页面的 <div id="workingArea" > 元素绑定，data 表示vue 使用上面的data4Vue对象。

```html
mounted:function(){
    this.list();
},
```

加载Vue对象成功之后会调用，成功的时候去调用list() 函数。

```html
methods: {
    list:function(){
        var url =  this.uri;
        axios.get(url).then(function(response) {
            vue.beans = response.data;
        });
    }
}
```

list 函数使用 data4Vue里的 uri作为地址，然后调用 axios.js 这个 ajax库，进行异步调用。 调用成功之后，把服务端返回的数据，保存在 vue.beans 上。

##### 展示数据

```html
<tr v-for="bean in beans ">
```

使用 v-for进行遍历， 这个 beans 就表示data4Vue里面的beans属性。

```html
 <td>{{bean.id}}</td>
```

bean就是遍历出来的每个id, 这里就是输出每个分类的id.

```html
<a :href="'admin_property_list?cid=' + bean.id "><span class="glyphicon glyphicon-th-list"></span></a>
```

在超链里的href里拼接分类id.

#### 热部署

spring为开发者提供了一个名为spring-boot-devtools的模块来使springboot应用支持热部署，提高开发的效率，修改代码后无需重启应用。

在 idea2017 里， springboot thymeleaf 修改 html 之后不能立即看到效果，要重新启动 Application 才可以看到效果。 这样做开发效率肯定是大受影响的。



