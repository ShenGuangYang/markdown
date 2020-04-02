# mybatis 源码解读



## mybatis 使用步骤



参照如下代码：

```java
@Test
    public void testStatement() throws IOException {
        // 1. 读取配置文件
        InputStream inputStream = Resources.getResourceAsStream("mybatis-config.xml");
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);
		// 2. 创建回话
        SqlSession session = sqlSessionFactory.openSession();
        try {
            // 3. 获取mapper
            BlogMapper mapper = session.getMapper(BlogMapper.class);
            // 4. 执行sql返回结果
            Blog blog = mapper.selectBlogById(1);
            System.out.println(blog);
        } finally {
            session.close();
        }
    }
```





## 配置解析过程源码解读



![1585808264708](../../img/mybatis/1.png)



## 创建会话



![1585810184412](../../img/mybatis/2.png)





## 获取代理对象

![1585811367671](../../img/mybatis/3.png)





## 执行sql返回结果



![1585815325637](../../img/mybatis/4.png)