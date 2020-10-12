# SpringBoot-Mybatis(XML版)

## 一、Maven依赖

- 主要依赖：**mybatis-spring-boot-starter**

```xml
<dependencies>
	<dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-web</artifactId>
    </dependency>
	<dependency>
		<groupId>org.mybatis.spring.boot</groupId>
		<artifactId>mybatis-spring-boot-starter</artifactId>
		<version>2.0.0</version>
	</dependency>
     <dependency>
        <groupId>mysql</groupId>
        <artifactId>mysql-connector-java</artifactId>
    </dependency>
</dependencies>
```

## 二、application.properties配置

```properties
mybatis.config-location=classpath:mybatis/mybatis-config.xml
mybatis.mapper-locations=classpath:mybatis/mapper/*.xml
mybatis.type-aliases-package=com.mybatisxml.mapper

spring.datasource.url=jdbc:mysql://localhost:3306/test?serverTimezone=UTC&useUnicode=true&characterEncoding=utf-8&useSSL=true
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.cj.jdbc.Driver
```

## 三、mybatis-config.xml

```xml
<configuration>
    <!--使用自动扫描包来定义别名-->
    <typeAliases>
        <package name="com.mybatisxml.model"/>
    </typeAliases>
</configuration>
```

## 四、UserMapper.xml映射文件

```xml
<mapper namespace="com.mybatisxml.mapper.UserMapper" >
    <resultMap id="BaseResultMap" type="User" >
        <id column="id" property="id" jdbcType="BIGINT" />
        <result column="userName" property="userName" jdbcType="VARCHAR" />
        <result column="passWord" property="passWord" jdbcType="VARCHAR" />
        <result column="user_sex" property="userSex" javaType="com.mybatisxml.enums.UserSexEnum"/>
        <result column="nick_name" property="nickName" jdbcType="VARCHAR" />
    </resultMap>
    <sql id="Base_Column_List" >
        id, userName, passWord, user_sex, nick_name
    </sql>

    <select id="getAll" resultMap="BaseResultMap"  >
        SELECT
        <include refid="Base_Column_List" />
        FROM users
    </select>
    <select id="getOne" parameterType="java.lang.Long" resultMap="BaseResultMap" >
        SELECT
        <include refid="Base_Column_List" />
        FROM users
        WHERE id = #{id}
    </select>
    <insert id="insert" parameterType="User" >
       INSERT INTO
       		users
       		(userName,passWord,user_sex)
       	VALUES
       		(#{userName}, #{passWord}, #{userSex})
    </insert>
    <update id="update" parameterType="User" >
        UPDATE
        users
        SET
        <if test="userName != null">userName = #{userName},</if>
        <if test="passWord != null">passWord = #{passWord},</if>
        nick_name = #{nickName}
        WHERE
        id = #{id}
    </update>
    <delete id="delete" parameterType="java.lang.Long" >
       DELETE FROM
       		 users
       WHERE
       		 id =#{id}
    </delete>
</mapper>
```

## 五、Mapper接口文件

```java
@Repository
public interface UserMapper {
    List<User> getAll();
    User getOne(Long id);
    void insert(User user);
    void update(User user);
    void delete(Long id);
}
```

## 六、启动类

在启动类中添加对 mapper 包扫描`@MapperScan`

```java
@SpringBootApplication
@MapperScan("com.mybatisxml.mapper")
public class SpringBootMybatisXmlApplication {
    public static void main(String[] args) {
        SpringApplication.run(SpringBootMybatisXmlApplication.class, args);
    }
}
```

## 七、测试类

```java
@RunWith(SpringRunner.class)
@SpringBootTest
class UserMapperTest {

    @Autowired
    private UserMapper userMapper;

    @Test
    public void testInsert() throws Exception {
        userMapper.insert(new User("Jax", "123", UserSexEnum.MAN));
        userMapper.insert(new User("Luna", "111", UserSexEnum.WOMAN));
        userMapper.insert(new User("Amy", "222", UserSexEnum.WOMAN));

        Assert.assertEquals(3, userMapper.getAll().size());
    }

    @Test
    public void testQuery() throws Exception {
        List<User> users = userMapper.getAll();
        if(users==null || users.size()==0){
            System.out.println("is null");
        }else{
            System.out.println(users.toString());
        }
    }


    @Test
    public void testUpdate() throws Exception {
        User user = userMapper.getOne(1l);
        System.out.println(user.toString());
        user.setNickName("汤姆");
        userMapper.update(user);
        Assert.assertTrue(("汤姆".equals(userMapper.getOne(1l).getNickName())));
    }
}
```

- Assert.assertEquals()及其重载方法: 1. 如果两者一致, 程序继续往下运行. 2. 如果两者不一致, 中断测试方法, 抛出异常信息 AssertionFailedError .