---
title: jQuery 中 ajax 使用 FormData 上传多个Base64或文件(File)
date: 2017.06.16 14:49:22
tags:
    - java
    - ajax
    - formdata
categories: java
---

-----
## 1. base64(字符串)作为数据源（在前端生成的文件而非选择的文件）。

### 前端调用上传接口示例

<!-- more -->

```javascript
// 图片分组上传，images，subjectTypeImages 为图片转的base64数组，titles是字符串数组。
var formData = new FormData();
images.forEach(function (image) {
    formData.append("picture[]", image);
});
subjectTypeImages.forEach(function (image) {
    formData.append("subjectTypeImages[]", image);
});
subjectStatusImages.forEach(function (image) {
    formData.append("subjectStatusImages[]", image);
});
titles.forEach(function (image) {
    formData.append("titles[]", image);
});
// 而外的参数数据
formData.append("titleKey", titleKey);
formData.append("searchType", searchType);

$.ajax({
    url:'upload.do',
    method:'POST',
    data:formData,
    // 默认为true，设为false后直到ajax请求结束(调完回掉函数)后才会执行$.ajax(...)后面的代码
    async: false, 
    // 下面三个，因为直接使用FormData作为数据，contentType会自动设置，也不需要jquery做进一步的数据处理(序列化)。
    cache: false,
    contentType: false,
    processData: false,
    success:function(data){
        console.log(data);
    },
    error:function(error){
        console.log(error.message);
    }
});
```
### java(springMVC) 后端接收示例
```java
@ResponseBody
@RequestMapping(value = "upload.do", method = RequestMethod.POST)
public Object upload(HttpServletRequest request) throws Exception {
    CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver(request.getSession().getServletContext());

    // 先判断request中是否包涵multipart类型的数据，
    if (multipartResolver.isMultipart(request)) {
        MultipartHttpServletRequest multiRequest = (MultipartHttpServletRequest) request;
        String[] picture = multiRequest.getParameterValues("picture[]");
        String[] subjectTypeImages = multiRequest.getParameterValues("subjectTypeImages[]");
        String[] subjectStatusImages = multiRequest.getParameterValues("subjectStatusImages[]");
        String[] titles = multiRequest.getParameterValues("titles[]");
        String searchType = multiRequest.getParameter("searchType");
        String titleKey = multiRequest.getParameter("titleKey");
        
        // todo: any-thing
    }
    return "success";
}
```

## 2. form 表单中使用input 选择文件上传， File类型

### form 表单

```html
<form id="form111" enctype="multipart/form-data" method="post" action="#">
    <input type="file" name="files">
    <input type="file" name="files">
    <input type="file" name="files">
    <!-- 这个可以选择多个文件，而且只可以选图片 -->
    <input type="file" name="files" accept="image/*" multiple>
    <input type="text" name="username">
    <input type="button" id="btn">
</form>
```

### javascript 上传接口调用示例

```javascript
$("#btn").click(function () {
    // 获取所有文件
    // 1. 使用直接使用form表单获取，会获取所有input值
        var fomdata = new FormData(document.getElementById("form111"));
    // 2. 只获取文件
        var fomdata = new FormData();
        $("#form111").children("input[name='files']").each(function (index, input) {
            for(var i=0; i<input.files.length; i++) {
                fomdata.append("files", input.files[i]);
            }
        });
    $.ajax({
        method:'POST',
        data:formData,
        url:'upload.do',
        // async: false, // 同步上传，默认(true)异步
        cache: false,
        contentType: false,
        processData: false,
        success:function(data){
            console.log(data);
        },
        error:function(){
            console.log('上传失败');
        }
    });
});
```

### java(springMVC) 后端代码示例

```java
@ResponseBody
@RequestMapping(value = "upload.do", method = RequestMethod.POST)
public Object upload(MultipartFile[] files/* files 为input的name，这样就可以获取所有文件的数组了。*/, String filename, HttpServletRequest request) throws Exception {
    // 遍历数组保存文件，并把文件相对路径存到数据库。。。等。
    return "success";
}

```

## 推荐一个前端上传组件
* [webuploader](http://fex.baidu.com/webuploader/getting-started.html)

* java 后端代码示例

```java
    @ResponseBody
    @RequestMapping(value = "upload.do", method = RequestMethod.POST)
    public Object uploadFile(MultipartFile file, String name1, HttpServletRequest request) throws Exception {
        UploadDto uploadDto = new UploadDto(); // 就是一个格式化返回信息的类。
        CommonsMultipartResolver multipartResolver = new CommonsMultipartResolver(request.getSession().getServletContext());
        // 先判断request中是否包涵multipart类型的数据，
        if (multipartResolver.isMultipart(request)) {
            try {
                // 
                String fileId = service.saveFile(file);
                uploadDto.setMessage("上传成功!");
                uploadDto.setCode(0);
                uploadDto.setFileId(fileId);
            } catch (Exception e) {
                e.printStackTrace();
                uploadDto.setMessage("上传失败!");
                uploadDto.setCode(-1);
            }
        }
        return uploadDto;
    }
```


## 总结
#### 不同的上传方式需要配合不同的接收方式。同一个项目尽量使用同一套规范进行封装。这里使用ajax作为客户端上传，其它平台遵循http协议的情况下可以使用同一个上传接口。
### 参考
1. [廖雪峰javascript教程](http://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/00143449993875172bbfac4b9764e2d9e2b5a17c706b3db000)
2. [Sending multipart/formdata with jQuery.ajax](https://stackoverflow.com/questions/5392344/sending-multipart-formdata-with-jquery-ajax)
