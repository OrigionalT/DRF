# 管理员登录API

**本小节目标**：

- 理解管理员登录API接口的设计
- 理解管理员登录API接口的业务逻辑
- 理解AdminAuthSerializer序列化器类的定义
- 理解管理员登录前端vue的请求逻辑
- 了解前端sessionStorage和localStorage保存数据

### 1. 接口设计

**请求方式**： POST `meiduo_admin/authorizations/`

**请求参数**： JSON 或 表单

| 参数名   | 类型 | 是否必须 | 说明   |
| -------- | ---- | -------- | ------ |
| username | str  | 是       | 用户名 |
| password | str  | 是       | 密码   |

**返回数据**： JSON

```json
{
    "id": "<用户id>",
    "username": "<用户名>",
    "token": "<jwt token>"
}
```

| 返回值   | 类型 | 是否必须 | 说明          |
| -------- | ---- | -------- | ------------- |
| username | str  | 是       | 用户名        |
| id       | int  | 是       | 用户id        |
| token    | str  | 是       | jwt token数据 |

### 2. API实现

1）在`meiduo_admin/views`目录下新建`users.py`文件并实现登录API接口。

```python
from rest_framework.views import APIView
from rest_framework.response import Response
from rest_framework import status

from meiduo_admin.serializers.users import AdminAuthSerializer

# POST /meiduo_admin/authorizations/
class AdminAuthorizeView(APIView):
    def post(self, request):
        """
        管理员登录:
        1. 获取参数并进行校验
        2. 服务器签发jwt token数据
        3. 返回应答
        """
        # 1. 获取参数并进行校验
        serializer = AdminAuthSerializer(data=request.data)
        serializer.is_valid(raise_exception=True)

        # 2. 服务器签发jwt token数据(create)
        serializer.save()

        # 3. 返回应答
        return Response(serializer.data, status=status.HTTP_201_CREATED)
```

2）在`meiduo_admin/serializer`目录下新建`users.py`文件并创建序列化器类。

```python
class AdminAuthSerializer(serializers.ModelSerializer):
    """管理员序列化器类"""
    username = serializers.CharField(label='用户名')
    token = serializers.CharField(label='JWT Token', read_only=True)

    class Meta:
        model = User
        fields = ('id', 'username', 'password', 'token')

        extra_kwargs = {
            'password': {
                'write_only': True
            }
        }

    def validate(self, attrs):
        # 获取username和password
        username = attrs['username']
        password = attrs['password']

        # 进行用户名和密码校验
        try:
            user = User.objects.get(username=username, is_staff=True)
        except User.DoesNotExist:
            raise serializers.ValidationError('用户名或密码错误')
        else:
            # 校验密码
            if not user.check_password(password):
                raise serializers.ValidationError('用户名或密码错误')

            # 给attrs中添加user属性，保存登录用户
            attrs['user'] = user

        return attrs

    def create(self, validated_data):
        # 获取登录用户user
        user = validated_data['user']

        # 设置最新登录时间
        user.last_login = timezone.now()
        user.save()

        # 服务器生成jwt token, 保存当前用户的身份信息
        from rest_framework_jwt.settings import api_settings

        # 组织payload数据的方法
        jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
        # 生成jwt token数据的方法
        jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER

        # 组织payload数据
        payload = jwt_payload_handler(user)
        # 生成jwt token
        token = jwt_encode_handler(payload)

        # 给user对象增加属性，保存jwt token的数据
        user.token = token

        return user
```

3）在`meiduo_admin/urls.py`文件中添加配置项

```python
from django.conf.urls import url
from meiduo_admin.views import users

urlpatterns = [
    url(r'^authorizations/$', users.AdminAuthorizeView.as_view()),
]
```

4）代码优化，登录API接口视图可以直接继承CreateAPIView。

```python
from rest_framework.generics import CreateAPIView

from meiduo_admin.serializers.users import AdminAuthSerializer

# POST /meiduo_admin/authorizations/
class AdminAuthorizeView(CreateAPIView):
    # 指定当前视图所使用的序列化器类
    serializer_class = AdminAuthSerializer
```

### 3. 前端保存jwt token(补充)

可以将jwt token保存在cookie中，也可以保存在浏览器的本地存储里，此处保存在浏览器本地存储中。

浏览器的本地存储提供了sessionStorage 和 localStorage 两种：

- **sessionStorage** 浏览器关闭即失效
- **localStorage** 长期有效

使用方法：

```js
sessionStorage.变量名 = 变量值   // 保存数据
sessionStorage.变量名  // 读取数据
sessionStorage.clear()  // 清除所有sessionStorage保存的数据

localStorage.变量名 = 变量值   // 保存数据
localStorage.变量名  // 读取数据
localStorage.clear()  // 清除所有localStorage保存的数据
```

登录前端：

```js
fnLogin(){
   ...
   this.axios.post(cons.apis + '/authorizations/',
    {
      username:this.username,
      password:this.password,

    },
    {}
    )
    .then(response=>{
      // console.log(response);
      // 存储用户登录信息
      sessionStorage.clear();
      localStorage.clear();
      localStorage.token = response.data.token;
      localStorage.username = response.data.username;
      localStorage.user_id = response.data.id;

      this.$router.push({path:'/home'});
    })
    .catch(error=>{
      this.errmsg = '用户名或密码错误';
      this.errshow = true;
    });
}
```

![1568220114406](.\images\1568220114406.png)



**总结**：

- 登录API接口业务逻辑：
  - 获取参数并进行校验(参数完整性，用户名和密码是否正确)
  - 创建jwt token保存登录用户的信息
  - 返回响应数据
- 序列化器类的定义
  - 确定序列化器的字段以及字段的read_only和write_only特性
  - 考虑调用序列化器的is_valid进行数据校验时是否需补充验证
  - 考虑调用序列化器的save进行数据保存时考虑ModelSerializer中的create和update是否需重写
- 客户端浏览器保存数据
  - sessionStorage：浏览器关闭数据即失效
  - localStorage：浏览器关闭数据页长期存在