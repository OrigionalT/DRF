# 分页器的设置

#### 1. Pagination

1. 简介
REST框架支持自定义分页风格，你可以修改每页显示数据集合的最大长度。
分页链接支持以下两种方式提供给用户：
- 分页链接是作为<font color=red>响应内容</font>提供给用户
- <font color=red>分页链接被包含在响应头中（Content-Range或者Link）</font>
内建风格使用作为响应内容提供给用户。这种风格更容易被使用可浏览API的用户所接受
如果使用通用视图<font color=red>`generic.GenericAPIView`</font>或者视图集合 <font color=red>`mixins.ListModelMixin`</font>。系统会自动帮你进行分页。如果使用的是<font color=red>APIView</font>,你就需要自己调用分页API，确保返回一个分页后的响应。
- 可以将pagination_class设置为None关闭分页功能。


### 2. 设置分页风格

可以通过 `DEFAULT_PAGINATION_CLASS` 和`PAGE_SIZE` 全局设置样式，比如设置我们分页器的极限

```python
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination',
    'PAGE_SIZE': 100
}
```

注意你需要同时设置*特定的分页器和页数最大值*



### 3.[PageNumberPagination](https://www.django-rest-framework.org/api-guide/pagination/#pagenumberpagination)

PNP分页样式可以查询参数中接受单个数字页码。

**request**：

```
GET https://api.example.org/accounts/?page=4
```

**response**：

```
HTTP 200 OK
{
    "count": 1023
    "next": "https://api.example.org/accounts/?page=5",
    "previous": "https://api.example.org/accounts/?page=3",
    "results": [
       …
    ]
}
```

#### [设定](https://www.django-rest-framework.org/api-guide/pagination/#setup)

要`PageNumberPagination`全局启用样式，请使用以下配置，并`PAGE_SIZE`根据需要进行设置：

```
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 100
}
```

在`GenericAPIView`子类上，您还可以根据不同的子类视图上加上`pagination_class`属性设置为`PageNumberPagination`。

