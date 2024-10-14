## RESTful架构和DRF入门

把软件（Software）、平台（Platform）、基础设施（Infrastructure）做成服务（Service）是很多IT企业都一直在做的事情，这就是大家经常听到的SasS（软件即服务）、PasS（平台即服务）和IasS（基础设置即服务）。实现面向服务的架构（SOA）有诸多的方式，包括RPC（远程过程调用）、Web Service、REST等，在技术层面上，SOA是一种**抽象的、松散耦合的粗粒度软件架构**；在业务层面上，SOA的核心概念是“**重用**”和“**互操作**”，它将系统资源整合成可操作的、标准的服务，使得这些资源能够被重新组合和应用。在实现SOA的诸多方案中，REST被认为是最适合互联网应用的架构，符合REST规范的架构也经常被称作RESTful架构。



### REST概述



REST这个词，是**Roy Thomas Fielding**在他2000年的博士论文中提出的，Roy是HTTP协议（1.0和1.1版）的主要设计者、Apache服务器软件主要作者、Apache基金会第一任主席。在他的博士论文中，Roy把他对互联网软件的架构原则定名为REST，即**RE**presentational **S**tate **T**ransfer的缩写，中文通常翻译为“**表现层状态转移**”或“**表述状态转移**”。



这里的“表现层”其实指的是“资源”的“表现层”。所谓资源，就是网络上的一个实体，或者说是网络上的一个具体信息。它可以是一段文本、一张图片、一首歌曲或一种服务。我们可以用一个URI（统一资源定位符）指向资源，要获取到这个资源，访问它的URI即可，URI就是资源在互联网上的唯一标识。资源可以有多种外在表现形式。我们把资源具体呈现出来的形式，叫做它的“表现层”。比如，文本可以用`text/plain`格式表现，也可以用`text/html`格式、`text/xml`格式、`application/json`格式表现，甚至可以采用二进制格式；图片可以用`image/jpeg`格式表现，也可以用`image/png`格式表现。URI只代表资源的实体，不代表它的表现形式。严格地说，有些网址最后的`.html`后缀名是不必要的，因为这个后缀名表示格式，属于“表现层”范畴，而URI应该只代表“资源”的位置，它的具体表现形式，应该在HTTP请求的头信息中用`Accept`和`Content-Type`字段指定，这两个字段才是对“表现层”的描述。



访问一个网站，就代表了客户端和服务器的一个互动过程。在这个过程中，势必涉及到数据和状态的变化。Web应用通常使用HTTP作为其通信协议，客户端想要操作服务器，必须通过HTTP请求，让服务器端发生“状态转移”，而这种转移是建立在表现层之上的，所以就是“表现层状态转移”。客户端通过HTTP的动词GET、POST、PUT（或PATCH）、DELETE，分别对应对资源的四种基本操作，其中GET用来获取资源，POST用来新建资源（也可以用于更新资源），PUT（或PATCH）用来更新资源，DELETE用来删除资源。



简单的说RESTful架构就是：“每一个URI代表一种资源，客户端通过四个HTTP动词，对服务器端资源进行操作，实现资源的表现层状态转移”。

我们在设计Web应用时，如果需要向客户端提供资源，就可以使用REST风格的URI，这是实现RESTful架构的第一步。当然，真正的RESTful架构并不只是URI符合REST风格，更为重要的是“无状态”和“幂等性”两个词，我们在后面的课程中会为大家阐述这两点。下面的例子给出了一些符合REST风格的URI，供大家在设计URI时参考。

| 请求方法（HTTP动词） | URI                        | 解释                                         |
| -------------------- | -------------------------- | -------------------------------------------- |
| **GET**              | `/students/`               | 获取所有学生                                 |
| **POST**             | `/students/`               | 新建一个学生                                 |
| **GET**              | `/students/ID/`            | 获取指定ID的学生信息                         |
| **PUT**              | `/students/ID/`            | 更新指定ID的学生信息（提供该学生的全部信息） |
| **PATCH**            | `/students/ID/`            | 更新指定ID的学生信息（提供该学生的部分信息） |
| **DELETE**           | `/students/ID/`            | 删除指定ID的学生信息                         |
| **GET**              | `/students/ID/friends/`    | 列出指定ID的学生的所有朋友                   |
| **DELETE**           | `/students/ID/friends/ID/` | 删除指定ID的学生的指定ID的朋友               |

### DRF使用入门



在Django项目中，如果要实现REST架构，即将网站的资源发布成REST风格的API接口，可以使用著名的三方库`djangorestframework` ，我们通常将其简称为DRF。

#### 安装和配置DRF

安装DRF。

```
pip install djangorestframework
```

配置DRF。

```python
INSTALLED_APPS = [

    'rest_framework',
    
]

# 下面的配置根据项目需要进行设置
REST_FRAMEWORK = {
    # 配置默认页面大小
    # 'PAGE_SIZE': 10,
    # 配置默认的分页类
    # 'DEFAULT_PAGINATION_CLASS': '...',
    # 配置异常处理器
    # 'EXCEPTION_HANDLER': '...',
    # 配置默认解析器
    # 'DEFAULT_PARSER_CLASSES': (
    #     'rest_framework.parsers.JSONParser',
    #     'rest_framework.parsers.FormParser',
    #     'rest_framework.parsers.MultiPartParser',
    # ),
    # 配置默认限流类
    # 'DEFAULT_THROTTLE_CLASSES': (
    #     '...'
    # ),
    # 配置默认授权类
    # 'DEFAULT_PERMISSION_CLASSES': (
    #     '...',
    # ),
    # 配置默认认证类
    # 'DEFAULT_AUTHENTICATION_CLASSES': (
    #     '...',
    # ),
}
```

#### 编写序列化器



前后端分离的开发需要后端为前端、移动端提供API数据接口，而API接口通常情况下都是返回JSON格式的数据，这就需要对模型对象进行序列化处理。DRF中封装了`Serializer`类和`ModelSerializer`类用于实现序列化操作，通过继承`Serializer`类或`ModelSerializer`类，我们可以自定义序列化器，用于将对象处理成字典，代码如下所示。

```python
from rest_framework import serializers 

class SubjectSerializer(serializers.ModelSerializer):

    class Meta:
        model = Subject
        fields = '__all__'
```

上面的代码直接继承了`ModelSerializer`，通过`Meta`类的`model`属性指定要序列化的模型以及`fields`属性指定需要序列化的模型字段，稍后我们就可以在视图函数中使用该类来实现对`Subject`模型的序列化。



#### 编写视图函数



DRF框架支持两种实现数据接口的方式，一种是FBV（基于函数的视图），另一种是CBV（基于类的视图）。我们先看看FBV的方式如何实现数据接口，代码如下所示。

```python
from rest_framework.decorators import api_view
from rest_framework.response import Response


@api_view(('GET', ))
def show_subjects(request: HttpRequest) -> HttpResponse:
    subjects = Subject.objects.all().order_by('no')
    # 创建序列化器对象并指定要序列化的模型
    serializer = SubjectSerializer(subjects, many=True)
    # 通过序列化器的data属性获得模型对应的字典并通过创建Response对象返回JSON格式的数据
    return Response(serializer.data)
```

对比上一个章节的使用`bpmapper`实现模型序列化的代码，使用DRF的代码更加简单明了，而且DRF本身自带了一套页面，可以方便我们查看我们使用DRF定制的数据接口，如下图所示。

[![img](https://github.com/jackfrued/Python-100-Days/raw/master/Day46-60/res/drf-app.png)](https://github.com/jackfrued/Python-100-Days/blob/master/Day46-60/res/drf-app.png)

直接使用上一节写好的页面，就可以通过Vue.js把上面接口提供的学科数据渲染并展示出来，此处不再进行赘述。

#### 实现老师信息数据接口



编写序列化器。

```python
class SubjectSimpleSerializer(serializers.ModelSerializer):

    class Meta:
        model = Subject
        fields = ('no', 'name')


class TeacherSerializer(serializers.ModelSerializer):

    class Meta:
        model = Teacher
        exclude = ('subject', )
```



编写视图函数。

```python
@api_view(('GET', ))
def show_teachers(request: HttpRequest) -> HttpResponse:
    try:
        sno = int(request.GET.get('sno'))
        subject = Subject.objects.only('name').get(no=sno)
        teachers = Teacher.objects.filter(subject=subject).defer('subject').order_by('no')
        subject_seri = SubjectSimpleSerializer(subject)
        teacher_seri = TeacherSerializer(teachers, many=True)
        return Response({'subject': subject_seri.data, 'teachers': teacher_seri.data})
    except (TypeError, ValueError, Subject.DoesNotExist):
        return Response(status=404)
```



配置URL映射。

```python
urlpatterns = [
    
    path('api/teachers/', show_teachers),
    
]
```

通过Vue.js渲染页面。

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>老师信息</title>
    <style>
        /* 此处省略掉层叠样式表 */
    </style>
</head>
<body>
    <div id="container">
        <h1>{{ subject.name }}学科的老师信息</h1>
        <hr>
        <h2 v-if="loaded && teachers.length == 0">暂无该学科老师信息</h2>
        <div class="teacher" v-for="teacher in teachers">
            <div class="photo">
                <img :src="'/static/images/' + teacher.photo" height="140" alt="">
            </div>
            <div class="info">
                <div>
                    <span><strong>姓名：{{ teacher.name }}</strong></span>
                    <span>性别：{{ teacher.sex | maleOrFemale }}</span>
                    <span>出生日期：{{ teacher.birth }}</span>
                </div>
                <div class="intro">{{ teacher.intro }}</div>
                <div class="comment">
                    <a href="" @click.prevent="vote(teacher, true)">好评</a>&nbsp;&nbsp;
                    (<strong>{{ teacher.good_count }}</strong>)
                    &nbsp;&nbsp;&nbsp;&nbsp;
                    <a href="" @click.prevent="vote(teacher, false)">差评</a>&nbsp;&nbsp;
                    (<strong>{{ teacher.bad_count }}</strong>)
                </div>
            </div>
        </div>
        <a href="/static/html/subjects.html">返回首页</a>
    </div>
    <script src="https://cdn.bootcdn.net/ajax/libs/vue/2.6.11/vue.min.js"></script>
    <script>
        let app = new Vue({
            el: '#container',
            data: {
                subject: {},
                teachers: [],
                loaded: false
            },
            created() {
                fetch('/api/teachers/' + location.search)
                    .then(resp => resp.json())
                    .then(json => {
                        this.subject = json.subject
                        this.teachers = json.teachers
                    })
            },
            filters: {
                maleOrFemale(sex) {
                    return sex? '男': '女'
                }
            },
            methods: {
               vote(teacher, flag) {
                    let url = flag? '/praise/' : '/criticize/'
                    url += '?tno=' + teacher.no
                    fetch(url).then(resp => resp.json()).then(json => {
                        if (json.code === 10000) {
                            if (flag) {
                                teacher.good_count = json.count
                            } else {
                                teacher.bad_count = json.count
                            }
                        }
                    })
                }
            }
        })
    </script>
</body>
</html>
```



### 前后端分离下的用户登录



之前我们提到过， HTTP是无状态的，一次请求结束连接断开，下次服务器再收到请求，它就不知道这个请求是哪个用户发过来的。但是对于一个Web应用而言，它是需要有状态管理的，这样才能让服务器知道HTTP请求来自哪个用户，从而判断是否允许该用户请求以及为用户提供更好的服务，这个过程就是常说的**会话管理**。

之前我们做会话管理（用户跟踪）的方法是：

用户登录成功后，在服务器端通过一个session对象保存用户相关数据，然后把session对象的ID写入浏览器的cookie中；

下一次请求时，HTTP请求头中携带cookie的数据，服务器从HTTP请求头读取cookie中的sessionid，根据这个标识符找到对应的session对象，这样就能够获取到之前保存在session中的用户数据。



我们刚才说过，REST架构是最适合互联网应用的架构，它强调了HTTP的无状态性，这样才能保证应用的水平扩展能力（当并发访问量增加时，可以通过增加新的服务器节点来为系统扩容）。显然，基于session实现用户跟踪的方式需要服务器保存session对象，在做水平扩展增加新的服务器节点时，需要复制和同步session对象，这显然是非常麻烦的。



解决这个问题有两种方案，一种是架设缓存服务器（如Redis），让多个服务器节点共享缓存服务并将session对象直接置于缓存服务器中；另一种方式放弃基于session的用户跟踪，使用**基于token的用户跟踪**。



基于token的用户跟踪是在用户登录成功后，为用户生成身份标识并保存在浏览器本地存储（localStorage、sessionStorage、cookie等）中，这样的话服务器不需要保存用户状态，从而可以很容易的做到水平扩展。基于token的用户跟踪具体流程如下：



1. 用户登录时，如果登录成功就按照某种方式为用户生成一个令牌（token），该令牌中通常包含了用户标识、过期时间等信息而且需要加密并生成指纹（避免伪造或篡改令牌），服务器将令牌返回给前端；



2. 前端获取到服务器返回的token，保存在浏览器本地存储中（可以保存在`localStorage`或`sessionStorage`中，对于使用Vue.js的前端项目来说，还可以通过Vuex进行状态管理）



3. 对于使用了前端路由的项目来说，前端每次路由跳转，可以先判断`localStroage`中有无token，如果没有则跳转到登录页；



4. 每次请求后端数据接口，在HTTP请求头里携带token；后端接口判断请求头有无token，如果没有token以及token是无效的或过期的，服务器统一返回401；

   

5. 如果前端收到HTTP响应状态码401，则重定向到登录页面。



通过上面的描述，相信大家已经发现了，基于token的用户跟踪最为关键是在用户登录成功时，要为用户生成一个token作为用户的身份标识。生成token的方法很多，其中一种比较成熟的解决方案是使用JSON Web Token。



#### JWT概述



JSON Web Token通常简称为JWT，它是一种开放标准（RFC 7519）。随着RESTful架构的流行，越来越多的项目使用JWT作为用户身份认证的方式。JWT相当于是三个JSON对象经过编码后，用`.`分隔并组合到一起，这三个JSON对象分别是头部（header）、载荷（payload）和签名（signature），如下图所示。



[![img](https://github.com/jackfrued/Python-100-Days/raw/master/Day46-60/res/json-web-token.png)](https://github.com/jackfrued/Python-100-Days/blob/master/Day46-60/res/json-web-token.png)

1. 头部

   1. 头部

      ```
      {
        "alg": "HS256",
        "typ": "JWT"
      }
      ```

      

      其中，`alg`属性表示签名的算法，默认是HMAC SHA256（简写成`HS256`）；`typ`属性表示这个令牌的类型，JWT中都统一书写为`JWT`。

   2. 载荷

      载荷部分用来存放实际需要传递的数据。JWT官方文档中规定了7个可选的字段：

      - iss ：签发人
      - exp：过期时间
      - sub：主题
      - aud：受众
      - nbf：生效时间
      - iat：签发时间
      - jti：编号

      除了官方定义的字典，我们可以根据应用的需要添加自定义的字段，如下所示。

      ```
      {
        "sub": "1234567890",
        "nickname": "jackfrued",
        "role": "admin"
      }
      ```

      

   3. 签名

      签名部分是对前面两部分生成一个指纹，防止数据伪造和篡改。实现签名首先需要指定一个密钥。这个密钥只有服务器才知道，不能泄露给用户。然后，使用头部指定的签名算法（默认是`HS256`），按照下面的公式产生签名。

      ```python
      HS256(base64Encode(header) + '.' + base64Encode(payload), secret)
      ```

算出签名以后，把头部、载荷、签名三个部分拼接成一个字符串，每个部分用`.`进行分隔，这样一个JWT就生成好了。



#### JWT的优缺点



使用JWT的优点非常明显，包括：

1. 更容易实现水平扩展，因为令牌保存在浏览器中，服务器不需要做状态管理。
2. 更容易防范CSRF攻击，因为在请求头中添加`localStorage`或`sessionStorage`中的token必须靠JavaScript代码完成，而不是自动添加到请求头中的。
3. 可以防伪造和篡改，因为JWT有签名，伪造和篡改的令牌无法通过签名验证，会被认定是无效的令牌。



当然，任何技术不可能只有优点没有缺点，JWT也有诸多缺点，大家需要在使用的时候引起注意，具体包括：

1. 可能会遭受到XSS攻击（跨站脚本攻击），通过注入恶意脚本执行JavaScript代码获取到用户令牌。
2. 在令牌过期之前，无法作废已经颁发的令牌，要解决这个问题，还需要额外的中间层和代码来辅助。
3. JWT是用户的身份令牌，一旦泄露，任何人都可以获得该用户的所有权限。为了降低令牌被盗用后产生的风险，JWT的有效期应该设置得比较短。对于一些比较重要的权限，使用时应通过其他方式再次对用户进行认证，例如短信验证码等。



如果要使用 Redis 实现无感刷新令牌的机制，主要需要对刷新令牌的存储和验证方式进行修改。传统情况下，刷新令牌存储在数据库中，但使用 Redis 可以提升性能，并且提供更灵活的分布式管理方案。下面将讲解如何修改现有代码来实现通过 Redis 存储和验证刷新令牌的逻辑。

### 1. **核心逻辑变更点**：

- 将刷新令牌存储在 Redis 中，而不是在数据库中。
- 在生成刷新令牌时，将其存入 Redis，并设置过期时间。
- 验证刷新令牌时，直接从 Redis 中获取令牌进行验证。
- Redis 中的刷新令牌过期时间与刷新令牌的有效期同步。

### 2. **需要安装 Redis 相关依赖**

确保项目中已经安装了 `StackExchange.Redis` 库，用于连接和操作 Redis。

```bash
dotnet add package StackExchange.Redis
```

### 3. **TokenManagementService 的改动**

为了使用 Redis 实现无感刷新的刷新令牌管理，需要将现有的数据库操作改为 Redis 操作。

#### 新增 Redis 连接和配置：

在 `TokenManagementService` 中添加 `IConnectionMultiplexer` 来管理 Redis 的连接。

```csharp
using StackExchange.Redis;
```

并在构造函数中注入 Redis 连接对象：

```csharp
public class TokenManagementService : ResponseMessageService, ITokenManagementService
{
    private readonly IConnectionMultiplexer _redis;
    private readonly IDatabase _redisDb;
    
    public TokenManagementService(
        IAccountRepository accountRepository,
        ITokenRepository tokenRepository,
        IMapper mapper,
        IUnitOfWork unitOfWork,
        IOptionsMonitor<ResponseMessage> responseMessage,
        IConnectionMultiplexer redis) : base(responseMessage)
    {
        this._accountRepository = accountRepository;
        this._tokenRepository = tokenRepository;
        this._mapper = mapper;
        this._unitOfWork = unitOfWork;
        this._redis = redis;
        this._redisDb = redis.GetDatabase(); // 获取 Redis 的数据库实例
        this._secret = Encoding.ASCII.GetBytes(JwtConfig.Secret);
    }
```

### 4. **修改刷新令牌存储逻辑**

将刷新令牌存储到 Redis 中，而不是数据库。

#### 生成新刷新令牌并存储到 Redis：

在 `GenerateNewTokensAsync` 方法中，将刷新令牌存储到 Redis 而不是存储到数据库。假设你使用用户ID作为键，并将刷新令牌信息作为值存储到 Redis 中。

```csharp
public async Task<BaseResponse<TokenResource>> GenerateNewTokensAsync(RefreshTokenResource refreshTokenResource, DateTime now)
{
    // 验证 Redis 中的刷新令牌
    var redisRefreshTokenKey = $"refresh_token:{refreshTokenResource.AccountId}";
    var storedRefreshToken = await _redisDb.StringGetAsync(redisRefreshTokenKey);
    
    if (string.IsNullOrEmpty(storedRefreshToken) ||
        !storedRefreshToken.Equals(refreshTokenResource.RefreshToken))
    {
        return new BaseResponse<TokenResource>(ResponseMessage.Values["Token_Not_Permitted"]);
    }

    // 获取账户信息
    var tempAccount = await _accountRepository.GetByIdAsync(refreshTokenResource.AccountId);
    if (tempAccount == null)
    {
        return new BaseResponse<TokenResource>(ResponseMessage.Values["Account_NoData"]);
    }

    // 生成新访问令牌
    var accessToken = GenerateAccessToken(tempAccount, now);

    // 生成新刷新令牌
    var refreshToken = GenerateRefreshToken(now, refreshTokenResource.UserAgent);
    
    // 将新的刷新令牌存储到 Redis 中，并设置过期时间
    await _redisDb.StringSetAsync(redisRefreshTokenKey, refreshToken.RefreshToken, TimeSpan.FromMinutes(JwtConfig.RefreshTokenExpiration));

    // 更新账户最后活动时间
    tempAccount.LastActivity = DateTime.UtcNow;

    // 提交数据库更改
    await _unitOfWork.CompleteAsync();

    var result = new TokenResource()
    {
        Id = refreshToken.Id,
        AccessToken = accessToken,
        ExpireTime = refreshToken.ExpireTime,
        RefreshToken = refreshToken.RefreshToken,
        Role = tempAccount.Role
    };

    return new BaseResponse<TokenResource>(result);
}
```

#### GenerateRefreshToken 方法保持不变：

```csharp
private Token GenerateRefreshToken(DateTime now, string userAgent)
    => new()
    {
        RefreshToken = GenerateRefreshTokenString(),
        ExpireTime = now.AddMinutes(JwtConfig.RefreshTokenExpiration),
        UserAgent = userAgent,
        IsUsed = false
    };

private static string GenerateRefreshTokenString()
{
    var randomNumber = new byte[32];
    using var randomNumberGenerator = RandomNumberGenerator.Create();
    randomNumberGenerator.GetBytes(randomNumber);
    return Convert.ToBase64String(randomNumber);
}
```

### 5. **验证刷新令牌**

在验证刷新令牌时，从 Redis 中检索并验证它的有效性。

#### 修改 `LogoutAsync` 方法

在登出时，也需要将刷新令牌从 Redis 中删除。

```csharp
public async Task<BaseResponse<object>> LogoutAsync(LogoutResource logoutResource)
{
    var redisRefreshTokenKey = $"refresh_token:{logoutResource.AccountId}";
    var storedRefreshToken = await _redisDb.StringGetAsync(redisRefreshTokenKey);

    if (string.IsNullOrEmpty(storedRefreshToken) || 
        !storedRefreshToken.Equals(logoutResource.RefreshToken))
    {
        return new BaseResponse<object>(ResponseMessage.Values["Token_Invalid"]);
    }

    // 删除 Redis 中的刷新令牌
    await _redisDb.KeyDeleteAsync(redisRefreshTokenKey);

    return new BaseResponse<object>(true);
}
```

### 6. **处理登录时的刷新令牌生成**

在用户登录时，生成刷新令牌并将其存储到 Redis 中。

```csharp
public async Task<BaseResponse<AccessTokenResource>> GenerateTokensAsync(LoginResource loginResource, DateTime now, string userAgent)
{
    // 验证用户凭证
    var tempAccount = await _accountRepository.ValidateCredentialsAsync(loginResource);
    if (tempAccount == null)
    {
        return new BaseResponse<AccessTokenResource>(ResponseMessage.Values["Token_Invalid"]);
    }

    // 生成新的访问令牌和刷新令牌
    var accessToken = GenerateAccessToken(tempAccount, now);
    var refreshToken = GenerateRefreshToken(now, userAgent);

    // 将刷新令牌存入 Redis，并设置过期时间
    var redisRefreshTokenKey = $"refresh_token:{tempAccount.Id}";
    await _redisDb.StringSetAsync(redisRefreshTokenKey, refreshToken.RefreshToken, TimeSpan.FromMinutes(JwtConfig.RefreshTokenExpiration));

    // 更新账户最后活动时间
    tempAccount.LastActivity = DateTime.UtcNow;
    await _unitOfWork.CompleteAsync();

    return new BaseResponse<AccessTokenResource>(MappingTokenResoure(tempAccount, refreshToken, accessToken));
}
```

### 7. **总结**

通过使用 Redis 实现刷新令牌的存储和验证，整个令牌管理系统的性能将大大提高，特别是在高并发环境下。以下是主要改动的几个方面：

- 将刷新令牌存储到 Redis 中，取代数据库存储。
- 每次生成新的刷新令牌时，将其存入 Redis，并设置过期时间。
- 在验证和登出时，从 Redis 中获取或删除刷新令牌，确保刷新令牌的唯一性和有效性。

这种方式特别适合分布式架构中的场景，因为 Redis 能够提供高效的分布式存储和访问，加快认证操作的速度。



**Token在Redis中的无感刷新**（也称为“续租”或“滑动过期”）是为了保证用户的会话在活跃期间不会失效的一种机制。简单来说，在Token快过期时，如果用户在一定的时间范围内仍然活跃，系统会自动刷新Token的过期时间，使得用户不感受到需要重新登录。

在使用Redis进行Token管理时，通常会使用一种“滑动窗口”的策略，即每次用户请求时，系统都会检查Token的过期时间并进行刷新，而不会等到Token完全过期才刷新。具体的实现可以按照以下步骤进行：

### 1. **Redis Token存储模型**

使用Redis存储Token时，通常采用 `key-value` 模型，`key` 为Token值，`value` 为用户相关的信息或Token元数据，Redis会对每个key设置TTL（Time-to-Live，过期时间）



```python
import redis
import jwt
import time

# 假设已经连接上了 Redis
r = redis.StrictRedis(host='localhost', port=6379, db=0)

# 生成 JWT Token（作为示例，实际场景应配置密钥和更复杂的内容）
def generate_token(user_id, secret, expiration=1800):
    token = jwt.encode({'user_id': user_id, 'exp': time.time() + expiration}, secret, algorithm='HS256')
    # 存储到Redis，设置过期时间为30分钟
    r.setex(token, expiration, user_id)
    return token

# 每次用户请求时，检查并刷新过期时间
def check_and_refresh_token(token, secret, refresh_threshold=300, new_expiration=1800):
    try:
        # 解码Token
        decoded = jwt.decode(token, secret, algorithms=['HS256'])
        user_id = decoded['user_id']

        # 获取Token剩余时间
        ttl = r.ttl(token)
        
        # 如果Token剩余时间小于5分钟，则刷新Token过期时间
        if ttl < refresh_threshold:
            r.expire(token, new_expiration)
            print(f"Token for user {user_id} is refreshed, new expiration is {new_expiration} seconds.")
        else:
            print(f"Token is still valid for {ttl} seconds.")

    except jwt.ExpiredSignatureError:
        print("Token has expired.")
    except jwt.InvalidTokenError:
        print("Invalid token.")

# 示例用法
secret_key = "my_secret"
user_token = generate_token(user_id=123, secret=secret_key)

# 模拟用户请求
check_and_refresh_token(user_token, secret=secret_key)

```



### 大致流程：

1. 用户登录时，服务器返回一个 **access token** 和一个 **refresh token**。
2. **access token** 用于验证用户身份，并具有较短的有效期。
3. **refresh token** 具有较长的有效期，当 **access token** 过期时，可以用 **refresh token** 申请一个新的 **access token**。
4. 当用户通过 **refresh token** 获取新的 **access token** 时，服务器可以选择生成一个新的 **refresh token**，或保持 **refresh token** 不变。
5. **refresh token** 一旦被盗或泄露，必须提供一种机制来撤销其权限，比如通过数据库或Redis存储Token的状态。



### 使用Redis管理Token无感刷新的示例：

#### 1. **登录阶段**

- 用户登录后，生成一个短生命周期的 **access token** 和一个长生命周期的 **refresh token**，并将 **refresh token** 存储在Redis中。

```python
import redis
import jwt
import time

# 假设已经连接到Redis
r = redis.StrictRedis(host='localhost', port=6379, db=0)

# 生成JWT Token
def generate_tokens(user_id, access_secret, refresh_secret, access_expiration=900, refresh_expiration=604800):
    # 生成 access token 和 refresh token
    access_token = jwt.encode({'user_id': user_id, 'exp': time.time() + access_expiration}, access_secret, algorithm='HS256')
    refresh_token = jwt.encode({'user_id': user_id, 'exp': time.time() + refresh_expiration}, refresh_secret, algorithm='HS256')
    
    # 将 refresh token 存储到 Redis，并设置一个较长的TTL
    r.setex(refresh_token, refresh_expiration, user_id)
    
    return access_token, refresh_token

# 示例用法：生成access token 和 refresh token
access_secret = "my_access_secret"
refresh_secret = "my_refresh_secret"
access_token, refresh_token = generate_tokens(user_id=123, access_secret=access_secret, refresh_secret=refresh_secret)
```



#### 2. **刷新Access Token**

- 当 **access token** 过期后，前端发送 **refresh token** 来获取新的 **access token**。服务器会验证 **refresh token**，并生成新的 **access token**。

```python
# 刷新Access Token
def refresh_access_token(refresh_token, access_secret, refresh_secret, access_expiration=900):
    try:
        # 验证 refresh token 是否有效
        decoded_refresh_token = jwt.decode(refresh_token, refresh_secret, algorithms=['HS256'])
        user_id = decoded_refresh_token['user_id']
        
        # 检查 Redis 中是否存在该 refresh token
        if r.exists(refresh_token):
            # 生成一个新的 access token
            new_access_token = jwt.encode({'user_id': user_id, 'exp': time.time() + access_expiration}, access_secret, algorithm='HS256')
            return new_access_token
        else:
            raise Exception("Invalid refresh token.")
    except jwt.ExpiredSignatureError:
        raise Exception("Refresh token has expired.")
    except jwt.InvalidTokenError:
        raise Exception("Invalid token.")

# 示例用法：通过refresh token刷新access token
new_access_token = refresh_access_token(refresh_token=refresh_token, access_secret=access_secret, refresh_secret=refresh_secret)
print(f"New Access Token: {new_access_token}")

```

#### 3. **处理Refresh Token的撤销**

- 可以在用户注销或检测到潜在的安全问题时，通过从Redis中删除 **refresh token** 来撤销它的权限。

```python
# 撤销 Refresh Token
def revoke_refresh_token(refresh_token):
    r.delete(refresh_token)
    print(f"Refresh token {refresh_token} revoked.")

# 示例用法：撤销refresh token
revoke_refresh_token(refresh_token=refresh_token)
```

### 实现细节注意点：

1. **access token** 的短生命周期可以提高安全性，因为即便被窃取，攻击者也只能在有限的时间内使用。
2. **refresh token** 应该保存在服务端（如Redis或数据库中）并设置合理的过期时间。
3. **refresh token** 的无感刷新机制通过Redis来检测Token是否有效并更新 **access token**，用户不会察觉到后台的Token续租操作。
4. 如果要进一步增强安全性，可以为 **refresh token** 实现一个“单次使用”机制，即每次使用 **refresh token** 获取新 **access token** 后，生成新的 **refresh token**，并使旧的 **refresh token** 失效。
5. 对 **refresh token** 的管理要慎重，因为 **refresh token** 的生命周期较长，如果被窃取，可能会造成更大的安全风险。

这种双Token机制可以有效提升安全性，并在保持良好用户体验的同时确保会话管理的无感刷新。