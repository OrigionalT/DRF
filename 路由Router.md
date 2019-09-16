# 路由Router

**本小节目标**：

- 掌握路由Router的作用和基本使用
- 理解视图集类属性lookup_value_regex的作用
- 常握视图集中额外处理方法配置项的生成
- 了解SimpleRouter的路由生成规则
- 了解SimpleRouter和DefaultRouter的区别



### 1. 基本使用

1）创建Router对象。

```python
from rest_framework.routers import SimpleRouter, DefaultRouter
router = SimpleRouter() 或 DefaultRouter()
```

2）注册视图集

```python
router.register(prefix, viewset, base_name)
```

> 参数：
> prefix：该视图集所有处理函数url地址的统一前缀
> viewset：视图集
> base_name：该视图集所有处理函数路由name的统一前缀

3）添加路由数据

```python
urlpatterns = [
    ...
]

# 将生成的路由配置项添加到urlpatterns列表中
urlpatterns += router.urls
```

> 通过router.urls可以获取路由Router生成的url配置项列表。

如：

```python
from booktest import views
# 1. 创建Router对象
from rest_framework.routers import SimpleRouter
router = SimpleRouter()
# 2. 注册视图集
router.register('books', views.BookInfoViewSet, base_name='books')
# 3. 打印生成的url配置项
for url in router.urls:
    print(url)
```

效果如下： ![路由Router生成配置项](./images/路由Router生成配置项.png)

> <FONT COLOR=RED>注：上面生成了两个配置项，但是提取参数的正则表达式不是\d+，可以在视图集中设置lookup_value_regex指定。</fONT>
>
### 2. 视图集中额外处理方法配置项生成
>
> 在视图集中，如果想要让路由Router自动生成视图集中额外添加的处理方法的url配置项，需要给额外的处理方法添加`rest_framework.decorators.action`装饰器。
>
> action装饰器可以接收两个参数：
>
> - **methods**：表明该处理方法对应的请求方式，列表传递。
>
> - detail
>
>   ：表明生成url配置项时，是否需要从路径中提取pk数据。
>
>   - True：生成url配置项时，需要从url地址中提取pk参数
>   - False：生成url配置项时，不需要从url地址中提取pk参数

```python
 @action(methods=['get'], detail=False)
    def latest(self, request):
 @action(methods=['put'], detail=True)
    def read(self, request, pk):
```

执行上面生成url配置项的代码，效果如下： ![路由Router生成配置项3](./images/路由Router生成配置项3.png)



路由器将匹配包含除斜杠和句点字符之外的任何字符的查找值。对于更严格（或宽松）的查找模式，请`lookup_value_regex`在视图集上设置该属性。例如，您可以将查找限制为有效的UUID：

```
class MyModelViewSet(mixins.RetrieveModelMixin, viewsets.GenericViewSet):
    lookup_field = 'my_model_id'
    lookup_value_regex = '[0-9a-f]{32}'
```

### 3. 路由router路由生成规则

1） SimpleRouter

![SimpleRouter](./images/simple_router.png)

2）DefaultRouter

![DefaultRouter](./images/default_router.png)

> 注：DefaultRouter与SimpleRouter的区别：
>
> 1. DefaultRouter会生成一个根路径/的配置项
> 2. DefaultRouter生成的每个配置项后都可以跟上`.json`，直接返回json数据。





#### 结论：

在使用router的时候该视图必须是个视图集，如果有特殊的查询就要定义属性lookup_value_regex，lookup_field(<font color=red>查找的字段可以不是(pk)</font>)







官方对lookup_field和lookup_url_kwarg 的解释:
        `lookup_field` - *The model field that should be used to for performing object lookup of individual model instances. Defaults to ‘pk’. Note that when using hyperlinked APIs you’ll need to ensure that both the API views and the serializer classes set the lookup fields if you need to use a custom value.*
        `lookup_url_kwarg` - *The URL keyword argument that should be used for object lookup. The URL conf should include a keyword argument corresponding to this value. If unset this defaults to using the same value as lookup_field.*

```python
# 定义model
class BookInfo(models.Model):
    btitle = models.CharField(max_length=20, verbose_name='名称')
    bpub_date = models.DateField(verbose_name='发布日期')
    bread = models.IntegerField(default=0, verbose_name='阅读量')
    bcomment = models.IntegerField(default=0, verbose_name='评论量')
    is_delete = models.BooleanField(default=False, verbose_name='逻辑删除')
    bimage=models.ImageField(verbose_name='图书图片',upload_to='booktest',null=True)

# 定义模型序列化器
class BookInfoSerializerModel(serializers.ModelSerializer):
    class Meta:
        model = BookInfo
        fields = '__all__'

# 定义视图,继承GenericAPIView基类
class BookInfo_detailGenericAPIView(GenericAPIView):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializerModel

    def get(self, request, pk):
        book = self.get_object()
        serializer = self.get_serializer(book)
        return Response(data=serializer.data)

# 路由的配置:
urlpatterns = [
    url(r'^books/(?P<pk>\d+)/$', views.BookInfo_detailGenericAPIView.as_view()),]

```

这样可以通过http://127.0.0.1:8000/books/2/ 来访问id=2 的数据详情
如何来自定义lookup_field的值实现按照任意字段去查询呢?
下面是代码:

```python
# 定义视图,继承GenericAPIView基类
class BookInfo_detailGenericAPIView(GenericAPIView):
    queryset = BookInfo.objects.all()
    serializer_class = BookInfoSerializerModel
    lookup_field = 'btitle'

    def get(self, request, btitle):
        book = self.get_object()
        serializer = self.get_serializer(book)
        return Response(data=serializer.data)

# 路由的配置:
urlpatterns = [
    url(r'^books/(?P<btitle>\w+)/$', views.BookInfo_detailGenericAPIView.as_view()),]

```

重写lookup_field,查询图书的title,这样get_object()就会依据’btitle’字段去数据库查询了.
这样可以通过http://127.0.0.1:8000/books/‘图书名称’/ 来访问btitle=’图书名称’ 的数据详情了