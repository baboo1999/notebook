# MyBatis优化程序

## 1.属性

添加属性文件

![img1](http://img.baboo.fun/21.07/mybatis_yh_1.PNG)

```properties
driver=com.mysql.jdbc.Driver
url=jdbc:mysql://localhost:3306?serverTimezone=UTC&useSSL=false&useUnicode=true&characterEncoding-UTF-8
username=root
password=
```

在配置表中添加上路径(注意添加<properties等>有顺序)，以及更改属性值的获取方式（${}）

![img2](http://img.baboo.fun/21.07/mybatis_yh_2.PNG)



## 2.别名

先在配置文件中添加别名

![img3](http://img.baboo.fun/21.07/mybatis_yh_3.PNG)

然后在dao下面的mapper.xml把别名加上

![img4](http://img.baboo.fun/21.07/mybatis_yh_4.PNG)

***

**or**

***

可以指定包名，MyBatis会在包名下搜索需要的java Bean.例如

扫描实体类的包，它的默认别名就是这个类的类名，**首字母小写**！

![img5](http://img.baboo.fun/21.07/mybatis_yh_5.PNG)

![img6](http://img.baboo.fun/21.07/mybatis_yh_6.PNG)



### 总结：

实体类比较少时，建议使用第一种方式。

实体类十分多时，建议使用第二种方式。

区别：第一种可以DIY,第二种其实也可以，利用注解的方式

![img7](http://img.baboo.fun/21.07/mybatis_yh_7.PNG)

![img8](http://img.baboo.fun/21.07/mybatis_yh_8.PNG)



**下面是一些为常见的 Java 类型内建的类型别名。它们都是不区分大小写的，注意，为了应对原始类型的命名重复，采取了特殊的命名风格。**

| byte       | byte       |
| ---------- | ---------- |
| _long      | long       |
| _short     | short      |
| _int       | int        |
| _integer   | int        |
| _double    | double     |
| _float     | float      |
| _boolean   | boolean    |
| string     | String     |
| byte       | Byte       |
| long       | Long       |
| short      | Short      |
| int        | Integer    |
| integer    | Integer    |
| double     | Double     |
| float      | Float      |
| boolean    | Boolean    |
| date       | Date       |
| decimal    | BigDecimal |
| bigdecimal | BigDecimal |
| object     | Object     |
| map        | Map        |
| hashmap    | HashMap    |
| list       | List       |
| arraylist  | ArrayList  |
| collection | Collection |
| iterator   | Iterator   |



## 3.设置

MyBatis中极为重要的调整设置，它们会改变MyBatis的运行行为。

日志实现：

![img9](http://img.baboo.fun/21.07/mybatis_yh_9.PNG)

开启缓存和懒加载：

![img10](http://img.baboo.fun/21.07/mybatis_yh_10.PNG)



## 4.映射器

MapperRegistry:注册绑定我们的Mapper文件；

方式一：【推荐】

```xml
    <!--每一个Mapper.XML都需要在MyBatis配置文件中注册-->
    <mappers>
        <mapper resource="com/wang/dao/UserMapper.xml"/>
    </mappers>
```

***

方式二：使用class文件绑定注册

```xml
    <!--每一个Mapper.XML都需要在MyBatis配置文件中注册-->
    <mappers>
        <mapper class="com.wang.dao.UserMapper"/>
    </mappers>
```

注意点：

- 接口和他的Mapper配置文件必须同名
- 接口和他的Mapper配置文件必须在同一个包下

***

方式三：使用扫描包进行注入绑定

```xml
    <mappers>
        <package name="com.wang.dao"/>
    </mappers>
```

注意点：

- 接口和他的Mapper配置文件必须同名
- 接口和他的Mapper配置文件必须在同一个包下



