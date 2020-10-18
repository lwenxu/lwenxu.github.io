---
title: JDBC
date: 2017-08-09 15:49:28
tags: JDBC
categories:
 -JavaWeb
---
### 1.普通的 JDBC 链接
```Java
public class JdbcConnect {
    @Test
   public void connect() throws ClassNotFoundException, SQLException {
        /**
         * 准备四大参数：  driver  url  username  password
         * 加载驱动类
         * 得到连接
         * 创建statement
         * 调用statement的查询或者更新方法
         * 关闭资源（必须关闭）倒关
         */


       Class.forName("com.mysql.jdbc.Driver");
        java.sql.Connection con= DriverManager.getConnection("jdbc:mysql://localhost:3306/Java","root","");
        Statement statement=con.createStatement();
        int effectRow=statement.executeUpdate("INSERT INTO `user` VALUES (1,'lwen',20,'lwen')");
        System.out.println(effectRow);
        ResultSet resultSet=statement.executeQuery("SELECT * FROM `user`");
        while (resultSet.next()){
            System.out.println(resultSet.getInt(1)); //或者使用列名称来获取
            System.out.println(resultSet.getString(2));
            System.out.println(resultSet.getInt(3));
            System.out.println(resultSet.getString(4));
        }
        //倒关
        resultSet.close();
        statement.close();
        con.close();
   }
}
```
<!--more-->
### 2. 数据库连接池：
```Java
/**
 * 数据库连接池
 * 就是为了连接课重用，由于创建销毁比较麻烦，所以放在一个map里面也就是一个池，下次要用的时候直接从里面取
 * 而不用平凡的销毁和创建
 *
 * 常用的就是dbcp连接池  这个也说commons里面的东西
 */
public class JdbcConnections {
    /**
     * 1.池参数
     * 2.四大连接参数
     * 3.实现javax.sql.DataSource接口
     */
    @Test
    public void dbcp() throws SQLException {
        BasicDataSource dataSource=new BasicDataSource();

        dataSource.setDriverClassName("com.mysql.jdbc.Driver");
        dataSource.setUrl("jdbc:mysql://localhost/Java");
        dataSource.setUsername("root");
        dataSource.setPassword("");

        dataSource.setMaxTotal(20);//设置最大的连接数
        dataSource.setMaxWaitMillis(1000);//设置最大等待时间

        Connection connection=dataSource.getConnection();//此时获得的connection不是以前的那个了而是在以前的mysql的连接上的增强
        //也就是说他是装饰者模式，装饰的部分就是close方法，因为此时的close方法不是关闭连接而是将连接归还的操作
//        下面的操作就和一般的sql连接一模一样了

//        然后注意一般来说上面的哪些创建连接池的操作我们只进行一次。然后整个项目就是用这一个连接池

    }

    /**
     * 现在dbcp用的更少了  主要用的就是c3p0
     */
    @Test
    public void c3p0() throws PropertyVetoException, SQLException {
        ComboPooledDataSource dataSource=new ComboPooledDataSource();

        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost/Java");
        dataSource.setUser("root");
        dataSource.setPassword("");

        dataSource.setMaxPoolSize(20);
        dataSource.setMinPoolSize(2);
        dataSource.setInitialPoolSize(20);
        dataSource.setAcquireIncrement(5);

        Connection connection=dataSource.getConnection();
        System.out.println(connection);  //返回的是代理连接  而不是装饰者模式  他比装饰者模式更加的灵活
        connection.close();

    }
}
```
### 3. c 组件中的数据库操作类
```Java
public class JdbcDbUtils {
    /**
     * 测试beanHandler  对于单行的javaBean
     * @throws SQLException
     */
    @Test
    public void fun1() throws SQLException, PropertyVetoException {
        ComboPooledDataSource dataSource=new ComboPooledDataSource();
        dataSource.setMaxPoolSize(20);
        dataSource.setAcquireIncrement(4);
        dataSource.setMinPoolSize(5);
        dataSource.setInitialPoolSize(10);

        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost/java");
        dataSource.setUser("root");
        dataSource.setPassword("");

        QueryRunner qr=new QueryRunner(dataSource);
        String sql="select * from user where id = ?";
        Object[] params={1};
        final Object query = qr.query(sql, new BeanHandler<UserBean>(UserBean.class), params);
        System.out.println(query);
    }

    @Test
    public void fun2() throws SQLException, PropertyVetoException {
        ComboPooledDataSource dataSource=new ComboPooledDataSource();
        dataSource.setMaxPoolSize(20);
        dataSource.setAcquireIncrement(4);
        dataSource.setMinPoolSize(5);
        dataSource.setInitialPoolSize(10);

        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost/java");
        dataSource.setUser("root");
        dataSource.setPassword("");

        QueryRunner qr=new QueryRunner(dataSource);
        String sql="select * from user";
        final Object query = qr.query(sql, new BeanListHandler<UserBean>(UserBean.class));
        System.out.println(query);
    }

    @Test
    public void fun3() throws SQLException, PropertyVetoException {
        ComboPooledDataSource dataSource=new ComboPooledDataSource();
        dataSource.setMaxPoolSize(20);
        dataSource.setAcquireIncrement(4);
        dataSource.setMinPoolSize(5);
        dataSource.setInitialPoolSize(10);

        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost/java");
        dataSource.setUser("root");
        dataSource.setPassword("");

        QueryRunner qr=new QueryRunner(dataSource);
        String sql="select * from user where id = ?";
        Object[] params={1};
        final Object query = qr.query(sql, new MapHandler(), params);
        System.out.println(query);
    }
    @Test
    public void fun4() throws SQLException, PropertyVetoException {
        ComboPooledDataSource dataSource=new ComboPooledDataSource();
        dataSource.setMaxPoolSize(20);
        dataSource.setAcquireIncrement(4);
        dataSource.setMinPoolSize(5);
        dataSource.setInitialPoolSize(10);

        dataSource.setDriverClass("com.mysql.jdbc.Driver");
        dataSource.setJdbcUrl("jdbc:mysql://localhost/java");
        dataSource.setUser("root");
        dataSource.setPassword("");

        QueryRunner qr=new QueryRunner(dataSource);
        String sql="select * from user ";
        Object[] params={1};
        final Object query = qr.query(sql, new MapListHandler());
        System.out.println(query);
    }
}
```
