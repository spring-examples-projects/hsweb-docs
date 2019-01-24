# 增删改查

## 说明

`hsweb`实现了`Dao`,`Service`,`Controller`层的通用增删改查，以及灵活的`动态条件`以及`API`,可在不修改mybatis mapper.xml的情况下实现动态添加sql条件。所有参数都使用了参数化预编译，没有sql注入风险。

{% hint style="info" %}
约定: 

1. 列必须明确定义，可通过mapper.xml 中的resultMap或者JPA注解进行定义。
2. 查询条件的值为空\(null或者空字符串\)时，条件将被忽略。
3. 新增或者修改的值为null时，列将被忽略。
4. 不能执行无条件的 修改，删除操作。
{% endhint %}

## Dao层

{% code-tabs %}
{% code-tabs-item title="UserMapper.xml" %}
```markup
<?xml version="1.0" encoding="UTF-8" ?>
<!DOCTYPE mapper
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"
        "http://www.mybatis.org/dtd/mybatis-3-mapper.dtd">

<mapper namespace="org.hswebframework.web.dao.authorization.UserDao">
    <resultMap id="UserResultMap" type="org.hswebframework.web.entity.authorization.UserEntity">
       <!--如果没有使用JPA注解,必须在此明确定义字段映射关系-->
        <id property="id" column="u_id" javaType="string" jdbcType="VARCHAR"/>
        <result property="name" column="name" javaType="string" jdbcType="VARCHAR"/>
        <result property="username" column="username" javaType="string" jdbcType="VARCHAR"/>
        <result property="password" column="password" javaType="String" jdbcType="VARCHAR"/>
        <result property="salt" column="salt" javaType="String" jdbcType="VARCHAR"/>
        <result property="status" column="status" javaType="Byte" jdbcType="NUMERIC"/>
        <result property="createTime" column="create_time" javaType="Long" jdbcType="NUMERIC"/>
        <result property="creatorId" column="creator_id" javaType="String" jdbcType="VARCHAR"/>
    </resultMap>

    <!--用于动态生成sql所需的配置-->
    <sql id="config">
        <bind name="resultMapId" value="'UserResultMap'"/> <!--注意这里的单引号: 'UserResultMap'-->
        <bind name="tableName" value="'s_user'"/> <!--注意这里的单引号: 's_user' -->
    </sql>
    
    <select id="query" parameterType="org.hswebframework.web.commons.entity.Entity" resultMap="UserResultMap">
        <include refid="config"/>
        <include refid="BasicMapper.buildSelectSql"/>
    </select>

</mapper>

```
{% endcode-tabs-item %}
{% endcode-tabs %}

## Service层

```java
//select id,name from table where name = ? limit 0,1
createQuery()
    .select("id","name")
    .where("name",name)
    .single();
```

```java
//update table set name=? where user = ?
createUpdate()
    .set(user::getName)
    .where(user::getId)
    .exec();
```

## Controller层

```java
@GetMapping
public ResponseMessage<PagerResult<Order>> queryOrder(QueryParamEntity param){

  String userId = ....;//当前登录用户ID
  
  //where creatorId = ? and ( 前端传入的参数 )
  return param.toNestQuery(query->query.and("creatorId",userId))
              .execute(service::selectPager);

}
```

## 开始使用

{% page-ref page="tong-yong-crud.md" %}


