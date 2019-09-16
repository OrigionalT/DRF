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

### 配置

所述`PageNumberPagination`类包括多个可重写修改分页样式属性。

要设置这些属性，您应该重写`PageNumberPagination`该类，然后启用上面的自定义分页类。

>- <font color=red>`django_paginator_class`</font> - Django 配置 Paginator的配置键。默认值是<font color=red>`django.core.paginator.Paginator`</font>，对于大多数用例来说应该没问题。
>
>- <font color=red>`page_size`</font> - 表示页容量。如果设置，则会覆盖该`PAGE_SIZE`设置。默认值与<font color=red>`PAGE_SIZE`</font>设置键的值相同。
>
>- <font color=red>`page_query_param`</font> - 一个字符串值，指示用于分页控件的查询参数的名称。
>
>- <font color=red>`page_size_query_param`</font> - 如果设置，这是一个字符串值，指示允许客户端基于每个请求设置页面大小的查询参数的名称。默认为`None`，表示客户端可能无法控制请求的页面大小。
>
>- ```url
>  -比如page_pagination.page_size_query_param='max' 
>  -http://127.0.0.1:8000/books/?xx=2&max=6   
>  表示查询第二页的数据,每页显示6条
>  ```
>```
>
>```
>
>- <font color=red>`max_page_size` </font>- 如果设置，这是一个数值，表示允许的最大请求页面大小。**此属性仅在`page_size_query_param`设置时有效。**
>
>- `last_page_strings`- 字符串值的列表或元组，指示可用于`page_query_param`请求集合中最后一页的值。默认为`('last',)`
>
>- `template` - 在可浏览API中呈现分页控件时要使用的模板的名称。可以重写以修改呈现样式，或设置为`None`完全禁用HTML分页控件。默认为`"rest_framework/pagination/numbers.html"`。
>```
>
>```



# 4.[LimitOffsetPagination](https://www.django-rest-framework.org/api-guide/pagination/#limitoffsetpagination)

```
（在第n个位置，向后查看n条数据）
```

>这种分页样式反映了查找多个数据库记录时使用的语法。客户端包括“限制”和“偏移”查询参数。限制表示要返回的最大项目数，并且与`page_size`其他样式相同。偏移量表示查询相对于整套未标记项目的起始位置。
>
>**要求**：
>
>```
>GET https://api.example.org/accounts/?limit=100&offset=400
>```
>
>**回应**：
>
>```
>HTTP 200 OK
>{
>"count": 1023
>"next": "https://api.example.org/accounts/?limit=100&offset=500",
>"previous": "https://api.example.org/accounts/?limit=100&offset=300",
>"results": [
>  …
>]
>}
>```
>
>
>
#### [设定](https://www.django-rest-framework.org/api-guide/pagination/#setup_1)

>要`LimitOffsetPagination`全局启用该样式，请使用以下配置：
>
>```
>REST_FRAMEWORK = {
>'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.LimitOffsetPagination'
>}
>```
>
>您也可以选择设置`PAGE_SIZE`密钥。如果`PAGE_SIZE`还使用该参数，则`limit`查询参数将是可选的，并且客户端可以省略该参数。
>
>在`GenericAPIView`子类上，您还可以将`pagination_class`属性设置为`LimitOffsetPagination`基于每个视图进行选择。
>
>```
>-重点的参数         
>-default_limit:默认每页显示的条数,默认偏移的数量            
>比如:default_limit=5            
>-http://127.0.0.1:8000/books/就会显示5条数据         
>-limit_query_param:往后偏移多少条就用默认值:limit         
>-offset_query_param:标杆值用默认值:offset          
>limit_query_param+offset_query_param联合起来用:               
>-访问:http://127.0.0.1:8000/books/?limit=1&offset=5  表示:以数据的第5条作为标杆,往后偏移1条         
>-max_limit:最大偏移的条数(最大取出的条数)
>```

## 5[CursorPagination](https://www.django-rest-framework.org/api-guide/pagination/#cursorpagination)

```
（加密分页，只能看上一页和下一页，速度快）
```

#### [设定](https://www.django-rest-framework.org/api-guide/pagination/#setup_2)

要`CursorPagination`全局启用该样式，请使用以下配置，`PAGE_SIZE`根据需要进行修改：

```
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.CursorPagination',
    'PAGE_SIZE': 100
}
```

在`GenericAPIView`子类上，您还可以将`pagination_class`属性设置为`CursorPagination`基于每个视图进行选择。

#### [配置](https://www.django-rest-framework.org/api-guide/pagination/#configuration_2)

所述`CursorPagination`类包括多个可重写修改分页样式属性。

要设置这些属性，您应该覆盖`CursorPagination`该类，然后启用上面的自定义分页类。

- `page_size`=表示页面大小的数值。如果设置，则会覆盖该`PAGE_SIZE`设置。默认值与`PAGE_SIZE`设置键的值相同。
- `cursor_query_param`=表示“cursor”查询参数名称的字符串值。默认为`'cursor'`。
- `ordering`=这应该是一个字符串或字符串列表，指示将应用基于光标的分页的字段。例如：`ordering = 'slug'`。默认为`-created`。也可以通过`OrderingFilter`在视图上使用来覆盖此值。
- `template`=在可浏览API中呈现分页控件时要使用的模板的名称。可以重写以修改呈现样式，或设置为`None`完全禁用HTML分页控件。默认为`"rest_framework/pagination/previous_and_next.html"`。

```
-重点参数:
-page_size:每页显示的条数
-cursor_query_param:不需要动
-ordering:按什么排序
-通过get_paginated_response返回结果中带上一页和下一页的链接地址   page_pagination.get_paginated_response(book_ser.data) 方法的用法
```

# 6.[自定义分页样式](https://www.django-rest-framework.org/api-guide/pagination/#custom-pagination-styles)

要创建自定义分页序列化程序类，您应该子类化`pagination.BasePagination`并覆盖`paginate_queryset(self, queryset, request, view=None)`和`get_paginated_response(self, data)`方法：

- 该`paginate_queryset`方法传递给初始查询集，并且应该返回一个只包含所请求页面中的数据的可迭代对象。
- 该`get_paginated_response`方法传递序列化页面数据并应返回`Response`实例。

注意，该`paginate_queryset`方法可以在分页实例上设置状态，该状态稍后可以由该`get_paginated_response`方法使用。

## [例](https://www.django-rest-framework.org/api-guide/pagination/#example)

假设我们想要使用包含嵌套“链接”键下的下一个和上一个链接的修改格式替换默认的分页输出样式。我们可以像这样指定一个自定义分页类：

```
class CustomPagination(pagination.PageNumberPagination):
    def get_paginated_response(self, data):
        return Response({
            'links': {
                'next': self.get_next_link(),
                'previous': self.get_previous_link()
            },
            'count': self.page.paginator.count,
            'results': data
        })
```

然后我们需要在配置中设置自定义类：

```
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'my_project.apps.core.pagination.CustomPagination',
    'PAGE_SIZE': 100
}
```

请注意，如果您关心如何在可浏览的API中的响应中显示键的顺序，您可以选择`OrderedDict`在构造分页响应的主体时使用，但这是可选的。

## [使用您的自定义分页类](https://www.django-rest-framework.org/api-guide/pagination/#using-your-custom-pagination-class)

要默认使用自定义分页类，请使用以下`DEFAULT_PAGINATION_CLASS`设置：

```
REST_FRAMEWORK = {
    'DEFAULT_PAGINATION_CLASS': 'my_project.apps.core.pagination.LinkHeaderPagination',
    'PAGE_SIZE': 100
}
```

列表端点的API响应现在将包含`Link`标头，而不是将分页链接作为响应主体的一部分包括在内，例如：