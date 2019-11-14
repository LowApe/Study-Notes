# Mybatis 多种分页方式

## 数组分页

利用 List 的 subList(index1.index2) 进行截取，需要通过计算。

```java
public List<Student> queryStudents(int currPage,int pageSize){
    // 查询所有数据
    List<Student> students = studentMapper.queryAll();

    // 开始数据计算
    int firstIndex = (currPage - 1) * pageSize;
    // 结束数据计算
    int lastIndex = currPage * pageSize;
    return students.subList(firstIndex,lastIndex);
}
```



## SQL分页



```java
List<Student> queryStudents(Map<String,Object> data);
```

xml 文件

```xml
<select id="queryStudents" parameterType="map" resultMap="studentmapper">
	select * from student limit #{currIndex},#{pageSize} 
</select>
```

dao 层

```java
public List<Student> queryStudents(int currPage,int pageSize){
	Map<String,Object> data = new HashMap<>();
    data.put("currIndex",(currPage-1)*pageSize);
    data.put("pageSize",pageSize);
	return studentMapper.queryStudents(data);
}
```



## 拦截器分页

暂时待定



## RowBounds 分页

数据量小用 RowBounds ？ 但是 数据量大时，实现拦截器？

mybatis 接口加入 RowBounds 参数

```java
public List<UserBean> queryUserByPage(String userName，RowBounds rowBounds)；
```

dao层

```java
public List<RoleBean> queryRolesByPage(String roleName,int start,int limit){
    return roleDao.queryRolesByPage(roleName,new RowBounds(start,limit));
}
```



> 后期总结