# MyBatis复杂查询

## 复杂查询

创建数据表

```mysql
CREATE TABLE `teacher` (
`id` INT(10) NOT NULL,
`name` VARCHAR(30) DEFAULT NULL,
PRIMARY KEY (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8;

CREATE TABLE `student` (
`id` INT(10) NOT NULL,
`name` VARCHAR(30) DEFAULT NULL,
`tid` INT(10) DEFAULT NULL,
PRIMARY KEY (`id`),
KEY `fktid` (`tid`),
CONSTRAINT `fktid` FOREIGN KEY (`tid`) REFERENCES `teacher` (`id`)
) ENGINE=INNODB DEFAULT CHARSET=utf8
```



插入数据

```mysql
INSERT INTO teacher(`id`,`name`) VALUES (1,'王老师');

INSERT INTO student(`id`,`name`,`tid`) VALUES (1,'小明','1');
INSERT INTO student(`id`,`name`,`tid`) VALUES (2,'小红','1');
INSERT INTO student(`id`,`name`,`tid`) VALUES (3,'小白','1');
INSERT INTO student(`id`,`name`,`tid`) VALUES (4,'小黑','1');
INSERT INTO student(`id`,`name`,`tid`) VALUES (5,'小王','1');
```



创建实体类、工具类、接口以及仓库，配置核心文件等。。。。

![img1]()



### 1对多查询



StudentMapper接口：

```java
package com.wang.dao;

import com.wang.pojo.Student;

import java.util.List;

public interface StudentMapper {
    //查询所有学生的信息，以及对应老师的信息
    public List<Student> getStudent();

    public List<Student> getStudent2();
}

```



StudentMapper.xml:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.wang.dao.StudentMapper">

<!--按结果嵌套处理-->
    <select id="getStudent2" resultMap="StudentTeacher2">
        select s.id sid,s.name sname,t.name tname
        from student s,teacher t
        where s.tid = t.id;
    </select>
    
    <resultMap id="StudentTeacher2" type="Student">
        <result property="id" column="sid"/>
        <result property="name" column="sname"/>
        <association property="teacher" javaType="Teacher">
            <result property="name" column="tname"/>
        </association>
    </resultMap>
    
    
<!--
    查询嵌套处理
    思路：
        1.查询所有的学生信息
        2.根据查询的学生的tid，寻找对应的老师！
-->

    <select id="getStudent" resultMap="StudentTeacher">
        select * from student
    </select>

    <resultMap id="StudentTeacher" type="Student">
        <result property="id" column="id"/>
        <result property="name" column="name"/>
    <!--复杂的属性需要单独处理
        对象：association
        集合：collocation
    -->
        <association property="teacher" column="tid" javaType="Teacher" select="getTeacher"/>
        
    </resultMap>
    
    <select id="getTeacher" resultType="Teacher">
        select * from teacher where id = #{id}
    </select>

</mapper>
```



在配置文件中设置控制台标准日志输出（mybatis-config.xml）：

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE configuration
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-config.dtd">
<!--configuration核心配置文件-->
<configuration>

    <properties resource="db.properties"/>

<!--设置-->
    <settings>
        <setting name="logImpl" value="STDOUT_LOGGING"/>
    </settings>

    <!-- 可以可以给实体类起别名 -->
    <!--
    <typeAliases>
        <typeAlias type="com.wang.pojo.User" alias="User"/>
    </typeAliases>
    -->
    <typeAliases>
        <package name="com.wang.pojo"/>
    </typeAliases>

    <environments default="development">
        <environment id="development">
            <transactionManager type="JDBC"/>
            <dataSource type="POOLED">
                <property name="driver" value="${driver}"/>
                <property name="url" value="${url}"/>
                <property name="username" value="${username}"/>
                <property name="password" value="${password}"/>
            </dataSource>
        </environment>
    </environments>

    <mappers>
        <mapper class="com.wang.dao.TeacherMapper"/>
        <mapper class="com.wang.dao.StudentMapper"/>
    </mappers>

</configuration>
```



MyTest测试类:

```java
import com.wang.dao.StudentMapper;
import com.wang.dao.TeacherMapper;
import com.wang.pojo.Student;
import com.wang.pojo.Teacher;
import com.wang.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class MyTest {
    public static void main(String[] args) {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        TeacherMapper mapper= sqlSession.getMapper(TeacherMapper.class);
        Teacher teacher = mapper.getTeacher(1);
        System.out.println(teacher);
        sqlSession.close();
    }


    @Test
    public void testStudent(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        StudentMapper mapper= sqlSession.getMapper(StudentMapper.class);
        List<Student> students = mapper.getStudent();
        for(Student student : students){
            System.out.println(student);
        }

        sqlSession.close();
    }

    @Test
    public void testStudent2() {
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        StudentMapper mapper = sqlSession.getMapper(StudentMapper.class);
        List<Student> students = mapper.getStudent2();
        for (Student student : students) {
            System.out.println(student);
        }

        sqlSession.close();
    }
}
```



### 多对1查询

TeacherMapper接口：

```java
package com.wang.dao;

import com.wang.pojo.Teacher;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;

import java.util.List;

public interface TeacherMapper {

    //获取老师
   // List<Teacher> getTeacher();

    //获取指定老师下的所有学生及老师的信息
    Teacher getTeacher(@Param("tid") int id);
    Teacher getTeacher2(@Param("tid")int id);
}
```



Teacher实体类：

```java
package com.wang.pojo;

import lombok.Data;

import java.util.List;

@Data
public class Teacher {
    private int id;
    private String name;

    //一个老师拥有多个学生
    private List<Student> students;
}
```



TeacherMapper.xml:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="com.wang.dao.TeacherMapper">

<!--    按结果嵌套查询-->
    <select id="getTeacher" resultMap="TeacherStudent">
        select s.id sid,s.name sname,t.name tname,t.id tid
        from student s,teacher t
        where s.tid = t.id and t.id = #{tid}
    </select>

    <resultMap id="TeacherStudent" type="Teacher">
        <result property="id" column="tid"/>
        <result property="name" column="tname"/>

        <collection property="students" ofType="Student">
            <result property="id" column="sid"/>
            <result property="name" column="sname"/>
            <result property="tid" column="tid"/>
        </collection>
    </resultMap>

<!--=======================================================-->
<!--    查询嵌套处理-->
<select id="getTeacher2" resultMap="TeacherStudent2">
    select * from mybatis.teacher where id = #{tid}
</select>
    <resultMap id="TeacherStudent2" type="Teacher">
        <collection property="student" javaType="ArrayList" ofType="Student" select="getStudentByTeacherId" column="id"/>
    </resultMap>

    <select id="getStudentByTeacherId" resultType="Student">
        select * from mybatis.student where tid = #{tid}
    </select>

</mapper>
```



测试：

```java
import com.wang.dao.TeacherMapper;
import com.wang.pojo.Teacher;
import com.wang.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.List;

public class MyTest {
    @Test
    public void Test(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        TeacherMapper mapper = sqlSession.getMapper(TeacherMapper.class);
        Teacher teacher = mapper.getTeacher(1);
        System.out.println(teacher);
        sqlSession.close();
    }

    @Test
    public void Test2(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        TeacherMapper mapper = sqlSession.getMapper(TeacherMapper.class);
        Teacher teacher = mapper.getTeacher2(1);
        System.out.println(teacher);
        sqlSession.close();
    }
}

```



## 动态SQL

创建数据表：

```mysql
CREATE TABLE `blog` (
`id` VARCHAR(50) NOT NULL COMMENT '博客id',
`title` VARCHAR(100) NOT NULL COMMENT '博客标题',
`author` VARCHAR(30) NOT NULL COMMENT '博客作者',
`create_time` DATETIME NOT NULL COMMENT '创建时间',
`views` INT(30) NOT NULL COMMENT '浏览量'
) ENGINE=INNODB DEFAULT CHARSET=utf8`blog`
```



目录结构：

![img2]()



Blog实体类:

```java
package com.wang.pojo;

import lombok.Data;

import java.util.Date;

@Data
public class Blog {
    private String id;
    private String title;
    private String author;
    private Date createTime;
    private int views;
}
```



idUtils:

```java
package com.wang.utils;

import org.junit.Test;

import java.util.UUID;

public class IdUtils {

    public static String getId(){
        return UUID.randomUUID().toString().replaceAll("-","");
    }

    @Test
    public void test(){
        System.out.println(IdUtils.getId());
    }
}

```



BlogMapper:

```java
package com.wang.dao;

import com.wang.pojo.Blog;

public interface BlogMapper {
    //插入数据
    int addBlog(Blog blog);

}
```



BlogMapper.xml:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.wang.dao.BlogMapper">

    <insert id="addBlog" parameterType="Blog">
        insert into mybatis.blog (id,title,author,create_time,views)
        values(#{id},#{title},#{author},#{createTime},#{views});
    </insert>
    
</mapper>
```



MyTest测试:

```java
import com.wang.dao.BlogMapper;
import com.wang.pojo.Blog;
import com.wang.utils.IdUtils;
import com.wang.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.Date;

public class MyTest {
    @Test
    public void addInitBlog(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);

        Blog blog = new Blog();
        blog.setId(IdUtils.getId());
        blog.setTitle("MyBatis测试");
        blog.setAuthor("baboo");
        blog.setCreateTime(new Date());
        blog.setViews(999);
        mapper.addBlog(blog);
        sqlSession.close();
    }

}
```



### IF语句

BlogMapper.xml:

```xml
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.wang.dao.BlogMapper">

    <insert id="addBlog" parameterType="Blog">
        insert into mybatis.blog (id,title,author,create_time,views)
        values(#{id},#{title},#{author},#{createTime},#{views});
    </insert>


    <select id="queryBlogIF" parameterType="map" resultType="blog">
        select * from mybatis.blog where 1=1
        <if test="title != null">
            and title = #{title}
        </if>
        <if test="author != null">
            and author = #{author}
        </if>
    </select>

</mapper>
```



BlogMapper.java:

```java
package com.wang.dao;

import com.wang.pojo.Blog;

import java.util.List;
import java.util.Map;

public interface BlogMapper {
    //插入数据
    int addBlog(Blog blog);

    //查询博客
    List<Blog> queryBlogIF(Map map);

}
```



MyTest.java:

```java
import com.wang.dao.BlogMapper;
import com.wang.pojo.Blog;
import com.wang.utils.IdUtils;
import com.wang.utils.MybatisUtils;
import org.apache.ibatis.session.SqlSession;
import org.junit.Test;

import java.util.Date;
import java.util.HashMap;
import java.util.List;

public class MyTest {
    @Test
    public void addInitBlog(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);

        Blog blog = new Blog();
        blog.setId(IdUtils.getId());
        blog.setTitle("MyBatis测试");
        blog.setAuthor("baboo");
        blog.setCreateTime(new Date());
        blog.setViews(999);
        mapper.addBlog(blog);
        sqlSession.close();
    }

    @Test
    public void queryBlogIF(){
        SqlSession sqlSession = MybatisUtils.getSqlSession();
        BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);

        HashMap map = new HashMap();
        map.put("title","Mybatis测试 ");
        List<Blog> blogs = mapper.queryBlogIF(map);

        for(Blog blog : blogs){
            System.out.println(blog);
        }

        sqlSession.close();
    }
}
```



其他不一一列举了，还有很多建议查看官方文档-> [mybatis – MyBatis 3 | 动态 SQL](https://mybatis.org/mybatis-3/zh/dynamic-sql.html)

if、choose、when、otherwise、trim、where、set、foreach、script、bind



**其中用choose和when可以去除多余的（and）,set可以去除多余的（,）**