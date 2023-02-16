# 多字段IN
```sql
-- 1
SELECT * FROM news_info WHERE (class_id,tag_id) IN ((10,6));
-- 2
SELECT * FROM news_info WHERE class_id = 10 AND tag_id = 6;
```


```sql
-- 1
SELECT * FROM news_info WHERE (class_id,tag_id) IN ((10,6),(13,5));
-- 2
SELECT * FROM news_info WHERE (class_id = 10 and tag_id = 6) OR (class_id = 13 and tag_id = 5);
```

1方式的语法查出来的结果与2方式的语法查出来的结果是相等的.

mybatis多字段in写法
```java
List<UserDto> selectByUserNameAndAge(List<User> list);
```

```xml
<select id="selectByUserNameAndAge" resultMap="UserMap" parameterType="java.util.List">
    select *
    from user
    where (user_name,user_age) in (
    <foreach collection="list" item="item" separator=",">
      (#{item.user_name},#{item.user_age})
    </foreach>
    )
  </select>

```

