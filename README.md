# flask-study 常见问题

### 1、改变默认的本地路径
```
可以在创建应用对象时使用关键字参数static_folder改变 默认的静态文件夹。
例如，你的静态文件都存放在应用下的assets目录下， 那么可以按如下的方式创建应用对象：

app = Flask(__name__,static_folder='assets')
也可以使用一个绝对路径：

app = Flask(__name__,static_folder='/var/www/static')
改变默认的本地路径并不会对路由表产生影响。

改变默认的URL规则 ： 如果不喜欢静态目录URL/static，也可以在创建应用 对象时使用关键字参数static_url_path换一个别的名字。

下面的示例中，将应用下的assets文件夹注册为静态目录/assets：

app = Flask(__name__,static_folder='assets',static_url_path='/assets')
当应用运行后，通过URL/assets/main.css就可以访问assets文件夹下的 main.css文件了。
```

### 2、设置端口号，打出debug
```
if __name__ == '__main__':
    app.run(host='0.0.0.0', port=5001, debug=True)
```

### 3、添加URL变量
```
如果指定访问点对应的视图函数接收参数，那么关键字参数将生成对应的参数URL。
下面的 示例将生成/contact/Julia?format=html：

@app.route('/')
def v_index():
    print url_for('v_contact',name='Julia',format='html')  #  /contact/<name>
    return ''
@app.route('/contact/<name>')
def v_contact(name):pass
</name>
```

### 4、添加锚点
```
使用_anchor关键字可以为生成的URL添加锚点。下面的示例将生成URL /contact#part2

@app.route('/')
def v_index():
    print url_for('v_contacts',_anchor='part2')
@app.route('/contact')
def v_contacts():pass
```
### 5、外部URL
```
默认情况下，url_for()生成站内URL，可以设置关键字参数_external 为True，生成包含站点地址的外部URL。
下面的示例将生成URLhttp://<x.y.z>/contacts:

@app.route('/')
def v_index():
    print url_for('v_contacts',_external=True)
@app.route('/contact')
def v_contacts():pass
```

### 6、Request对象
```
Request实例对象中包含 了关于一次HTTP请求的一切信息，常用的属性包括：

form - 记录请求中的表单数据。类型：MultiDict
args - 记录请求中的查询参数。类型：MultiDict
cookies - 记录请求中的cookie。类型：Dict
headers - 记录请求中的报文头。类型：EnvironHeaders
method - 记录请求使用的HTTP方法：GET/POST/PUT....。类型：string
environ - 记录WSGI服务器转发的环境变量。类型：Dict
url - 记录请求的URL地址。类型：string
关于Request类的详细信息，可以查阅 Flask官网


在视图函数中，可以直接使用全局对象request访问当次请求对应的Response对象。 下面的示例打印当前请求的全部报文头信息：

@app.route('/')
def index(): 
    print request.headers
    return 'see console output'

比如，如果想获取cookie，可以用 request.headers['Cookie'] 来获取
```

### 7、读取表单数据(POST方式提交)
```
Flask框架将用户使用POST方法提交的表单数据，存储在所创建Request对象的 form属性中。

form是一个MultiDict类型的对象，和Dict类似，我们可以使用[]操作符读取 指定的键值：

@app.route('/')
def v_index():
    uid = request.form['uid']
    pwd = request.form['pwd']
    return  'uid : %s pwd : %s' % (uid,pwd)
除了form属性，也可以使用Request对象的values属性来读取表单值，用法相同。
```

### 8、读取查询参数(GET方式提交)
```
对于浏览器以GET方法提交的表单数据，Flask框架将其存储在Request实例对象的args 属性中。

和form属性一样，args属性也是一个MultiDict类型的对象，因此我们可以是用[]操作 符读取指定键值：

@app.route('/search')
def v_search():
    q = request.args['q']
    return 'you are searching %s' % q
除了args属性，也可以使用Request对象的values属性来读取查询参数。
```

### 9、读取JSON数据
```
在REST架构中，前端经常使用AJAX提交JSON数据：

//javascript code in browser
$.ajax({
    url : "/user",
    method : "POST",
    data : JSON.stringify(jsondata),
    contentType : "application/json;charset=UTF-8",
    success : function(){},
    error : function(){}
});
如果前端在提交JSON数据时，正确设置了mimetype为application/json， 那么Flask框架会将其存储在Request实例对象的json属性中。

根据所提交的JSON数据的具体情况，Request对象的json属性值为自动转换后的 的字典或数组：

@app.route('/')
def v_index():
    print request.json #字典或数组
    return 'see console output'
```

### 10、响应 ：Response
```
与Request类相对应，Flask框架使用Response类表征对HTTP请求的响应。

根据视图函数的返回结果，Flask确保向后续处理环节传递一个正确的Response实例对象。 这有几种可能性：

视图函数返回字符串 ：当视图函数返回的是一个字符串时，Flask自动使用 这个字符串作为正文内容，
以200作为状态码，以text/html作为mimetype，构造 一个Response对象传递给后续处理环节。

下面的示例中，视图函数v_ping()返回一个字符串：

@app.route('/ping')
def ping():
    return 'pong'
Flask框架将基于这个返回结果构造如下的Response对象：

response : ['pong']
status_code : 200
mimetype : 'text/html'
视图函数返回元组 ：当视图函数返回的是一个形式如(response,status,headers)的元组时， Flask自动根据这几个值构造一个Response对象。

下面的示例中，视图函数v_ping()返回一个包含响应正文、状态码和包头的元组：

@app.route('/ping')
def v_ping():
    return 'pong',200,{'x-tag':'sth. magic'}
Flask框架将基于这个返回结构构造如下的Response对象：

response: ['pong']
status_code : 200
mimetype : 'text/html'
headers : [('x-tag','sth. magic')]
视图函数返回Response对象 ：当视图函数返回的是一个Response对象时，Flask 框架直接将这个对象向后续处理环节传递：

from flask import Flask,make_response
@app.route('/ping')
def v_ping():
    rsp = make_response('pong')
    rsp.mimetype = 'text/plain'
    rsp.headers['x-tag'] = 'sth. magic'
    return rsp
make_response()函数用来构造一个Response对象，第一个参数为响应的正文。

```

### 11、设置cookie
```
使用Response类的set_cookie()方法可以设置客户端cookie：

Response.set_cookie(
    key,            //键
    value='',       //值
    max_age=None,   //秒为单位的cookie寿命，None表示http-only
    expires=None,   //失效时间，datetime对象或unix时间戳
    path='/',       //cookie的有效路径
    domain=None,    //cookie的有效域
    secure=None, 
    httponly=False)
显然，如果要设置cookie，我们必须自行构造Response对象，而不是交给Flask框架去 完成这件事情。

下面的示例在访问首页/时设置cookie，并在访问/page2时读取cookie：

@app.route('/')
def index():
    rsp = make_response('go <a href="%s">page2</a>' % '/page2')
    rsp.set_cookie('user','JJJJJohnny')
    return rsp
@app.route('/page2')
def page2():
    user = request.cookies['user']
    return 'you are %s' % user
    
```


### 12、构造JSON响应
```
在Flask中，可以使用json模块的dumps()方法将数组或字典对象转换为JSON字符串：

from flask import json
a = [1,2,3]
print json.dumps(a) # '[1,2,3]'
b = {'x':1,'y':2}
print json.dumps(b) # '{"x":1,"y":2}'
这在开发REST API时相当有用。比如我们定义获得用户列表的API为GET /user：

@app.route('/user')
def v_users():
    users = ['Linda','Marion5','Race8']
    return json.dumps(users),200,[('Content-Type','application/json;charset=utf-8')]
为了向客户端正确标示响应的类型，我们在视图函数返回时使用了元组，在响应报文头/headers 中添加了Content-Type字段，并设置响应正文类型为application/json。

```

### 13、重定向响应
```
使用flask框架的redirect()方法，可以要求客户端进行重定向：

flask.redirect(location, code=302, Response=None)
redirect()方法其实是构造了一个具有重定向状态码的Response对象。重定向状态码 默认为302，这表示一个临时性重定向。redirect()方法还支持以下重定向状态码：

301 - 请求的网页已被永久移动到新位置
302 - 服务器目前正从不同位置的网页响应请求，但请求者应继续使用原有位置来进行以后的请求。
303 - 对于POST请求，它表示请求已经被处理，客户端可以接着使用GET方法去请求Location里的URI
305 - 请求者只能使用代理访问请求的网页。
307 - 对于POST请求，表示请求还没有被处理，客户端应该向Location里的URI重新发起POST请求
下面的示例中，当用户访问首页时，将自动重定向到新手页/newbies：

@app.route('/')
def v_index():
    return redirect('/newbies')
 
@app.route('/newbies')    
def v_newbies():
    return 'this page is for newbies only!'
    
```

### 14、终止响应
```
可以使用flask框架的abort()方法通知框架终止处理当前响应：

flask.abort(code)
abort()方法的code参数用来指定返回给客户端的HTTP状态码。由于abort()方法 将抛出HttpException异常，因此它之后的代码不会被执行。

下面的示例中，要求访问/admin时必须附加查询参数token，否则返回HTTP 状态码401，提醒用户没有权限：

@app.route('/admin')
def v_admin():
    if 'token' in request.args:
        return 'you are a good boy.'
    else:
        abort(401)

```

### 15、session(会话) : 访问者上下文
```
会话/Session是为Web服务器建立状态的一个成熟模式。会话主要解决两个问题：

访问者的标识问题

服务器需要识别来自同一访问者的请求。这主要是通过浏览器的cookie实现的。 
访问者在第一次访问服务器时，服务器在其cookie中设置一个唯一的ID号——会话ID。 
这样，访问者后续对服务器的访问头中将自动包含该信息，服务器通过这个ID号，即可区 隔不同的访问者。

Flask框架中，每当一个请求进来时会自动根据请求中cookie的会话ID创建 一个Session类的实例对象。
你可以查看当前请求的cookie验证这一点（会话ID的键 默认为session）：

@app.route('/')
def v_index():
    return request.cookies['session']
访问者信息的记录问题

服务器可以记录、提取指定访问者的历史信息。对每一个会话ID，服务端维护一个 数据上下文，这个数据运行在内存中，通常在变化时持久化到文件系统中或数据库中。

在视图函数内，Flask提供了一个全局对象session，它始终等效于当前请求所对应的 Session类实例对象。
Session类定义了get_item()方法和set_item()方法， 因此我们可以像使用Dict对象一样，通过[]操作符读取或设置会话变量：

@app.route('/')
    if !session['user']:
        return redirect('/login')
    return 'some restricted for authorized users only'
由于默认情况下，Flask将会话对象加密后存储在客户端的cookie里，因此必须要 为应用实例的secret_key属性配置一个加密种子才能使用session：

app.secret_key = 'sth. random as a encrypt key.'

```

