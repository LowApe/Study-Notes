> 场景: 在form 表单中使用了 `enctype="multipart/form-data"` 如果servlet 3.0 并且 tomcat 不是8.0以上会出现 无法使用 `getPart()` 和 `getParameter()`

> 解决: 在servlet 添加注解`@MultipartConfig`

参考:[enctype](https://stackoverflow.com/questions/2422468/how-to-upload-files-to-server-using-jsp-servlet)
