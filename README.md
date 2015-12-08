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
视图函数返回元组 ：当视图函数返回的是一个形式如(response,status,headers)的元组时，
Flask自动根据这几个值构造一个Response对象。

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
为了向客户端正确标示响应的类型，我们在视图函数返回时使用了元组，在响应报文头/headers中添加了Content-Type字段，
并设置响应正文类型为application/json。

```

### 13、重定向响应
```
使用flask框架的redirect()方法，可以要求客户端进行重定向：

flask.redirect(location, code=302, Response=None)
redirect()方法其实是构造了一个具有重定向状态码的Response对象。重定向状态码 默认为302，这表示一个临时性重定向。
redirect()方法还支持以下重定向状态码：

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
abort()方法的code参数用来指定返回给客户端的HTTP状态码。由于abort()方法 将抛出HttpException异常，
因此它之后的代码不会被执行。

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

服务器可以记录、提取指定访问者的历史信息。对每一个会话ID，服务端维护一个 数据上下文，
这个数据运行在内存中，通常在变化时持久化到文件系统中或数据库中。

在视图函数内，Flask提供了一个全局对象session，它始终等效于当前请求所对应的 Session类实例对象。
Session类定义了get_item()方法和set_item()方法， 因此我们可以像使用Dict对象一样，
通过[]操作符读取或设置会话变量：

@app.route('/')
    if !session['user']:
        return redirect('/login')
    return 'some restricted for authorized users only'
由于默认情况下，Flask将会话对象加密后存储在客户端的cookie里，
因此必须要为应用实例的secret_key属性配置一个加密种子才能使用session：

app.secret_key = 'sth. random as a encrypt key.'

```

### 16、伪造请求上下文
```
有一种场景特别需要伪造请求上下文 —— 自动测试：

@app.route('/genius')
def genius():
    return 'nothing special'
with app.test_request_context('/genius',method='GET'):
    print app.dispatch_request() 
```

### 17、生命周期
```
Flask框架在一次请求应答的生命周期中，提供了一些方便的装饰器以便开发者可以 在合理的时间点注入一些定制化的代码：

before_first_request()

被装饰的函数将在应用启动后，处理第一个请求之前被调用

before_request()

被装饰的函数将在处理请求之前被调用

after_request(response)

被装饰的函数将在处理请求之后被调用，调用时前序环节生成的Response 对象将作为参数传入。
被装饰的函数需要返回一个Response对象以作为后续环节的处理对象。
需要指出的是，如果在前序环节dispatch_request()处理请求时发生异常，after_request装饰 器装饰的函数可能被跳过。

teardown_request(exception)

被装饰的函数将在请求上下文对象出栈之前被调用。
和after_request不同，即使前序环节发生异常，Flask保证这个函数总能被调用。因此，清理 性质的代码应当放置在此处。

teardown_appcontext(exception)

被装饰的函数将在应用上下文对象出栈之前被调用。应用上下文总是和请求上下文一起出入栈。
```

### 18、模板渲染(jinjia2)
```
Flask基于Jinja2模板引擎，提供了两个渲染函数，分别使用字符串或单独的文件保存模板内容：

render_template_string(sourcestr,**context) - 使用sourcestr作为模板字符串
render_template(filename,**context) - 使用filename指定的文件内容作为模板字符串
render_template_string ：下面的示例使用一个相当简单的模板，向不同的用户回 送个性化的欢迎信息：

@app.route('/user/<uname>')
def show_user_profile(uname):    
    return render_template_string('<h1>Welcome,{{ uname }}</h1>',uname=uname)
</uname>
在Jinja2的语法中，{{varibale}}表示一个输出命令，每当渲染引擎发现一个输出命令，
它就在渲染结果中，使用模板数据上下文中变量variable的值替换原始的输出命令。

render_template ：在生产环境中，在代码里写模板字符串不是什么好主意。
正经的方法是将模板写在单独的模板文件里，使用render_template()函数进行渲染：

@app.route('/user/<username>')
def v_user(username):    
    return render_template('user.html',username=username)
</username>
默认情况下，Flask使用当前模块文件夹下的templates子目录作为模板目录，user.html文件 应当放置在这个文件夹下：

/app
    /web.py
    /templates
        /user.html
user.html的内容如下：

<body>
    <h1>Welcome, {{ username }}</h1>
</body>
```

### 19、变量与表达式
```
模板变量在模板被渲染时通过上下文字典传递给模板。下面的示例中，在模板中使用了变量name和age，
当调用render_template_string()渲染模板时，通过关键字参数 将两个变量的值传递进来：

tpl = 'name : {{ name }} age : {{age}} '
print render_template_string(tpl,name='Marion5',age=12)
变量成员 ：如果传入模板的变量是不是Python简单类型，而是比如字典或对象类型， 那么在模板中可以向Python中一样的方式访问其成员属性或方法。

稍有不同的是，对于字典变量，除了可以使用[]方式访问其成员，还可以使用.：

tpl = 'name: {{u["name"]}} name again:{{u.name}}'
print render_template_string(tpl,u={'name':'Marion5','age':12})
同样的，对于对象变量，除了使用.访问属性值，还可以使用[]：

class User:
    def __init__(self,name,age):
        self.name = name
        self.age = age
tpl = 'name : {{u.name}} name again:{{u["name"]}}'
print render_template_string(tpl,u=User('Mary',20))
表达式 ：变量还可以应用表达式，比如进行数学运算，那些常用的数学 操作符（ + - * / // % ** ）都是有效的：

data = {'x':12,'y':13}
tpl = '{{x}} + {{y}} = {{ x+y }}'
print render_template_string(tpl,**data)
或者进行比较或逻辑运算（ == != > >= < <= and or not）:

data = {'x':12,'y':13,'z':11}
tpl = '{{x}} > {{y}} : {{ x>y }}'
print render_template_string(tpl,**data)
函数调用 ：在输出命令中，可以对变量或常量进行函数调用：

tpl = '{{ range(10) }} '
print render_template_string(tpl)
需要注意的是，模板有自己的全局域/globals，因此这里的range()函数并不是Python 应用中的函数。

```

### 20、全局对象
```
Jinja2内置的全局对象包括：

range([start, ]stop[, step])
lipsum(n=5, html=True, min=20, max=100)
dict(**items)
class cycler(*items)
class joiner(sep=', ')
Flask向Jinja2模板注入了以下全局对象，可以在模板中直接访问：

config - 当前Flask应用实例的配置对象
request - 当前HTTP请求对象
session - 当前HTTP请求的会话对象
g - 当前HTTP请求周期内的全局对象
url_for() - URL生成函数
get_flashed_messages() - 闪信函数
下面的示例中，从session中提取当前用户名：

@app.route('/')
def v_index():
    tpl = 'welcome back, {{ session.username }}, your user agent is <b>{{ request.headers['User-Agent']}}</b>'
    return render_template_string(tpl)
    
```

### 21、自定义全局对象
```
可以使用应用对象的context_processor装饰器向引擎注入额外的全局对象。 下面的示例向模板全局域中注入vendor变量，其值为hubwiz：

@app.context_processor
def vendor_processor():
    return dict(vendor='hubwiz')
这时我们可以在模板中直接使用vendor变量了：

@app.route('/')
def v_index():
    tpl = 'powered by {{vendor}}'
    return render_template_string(tpl)
当然，同样的方法可以用于注入全局函数。下面的示例向模板全局域中注入format_price 函数：

@app.context_processor
def utility_processor():
    def format_price(amount, currency=u'€'):
        return u'{0:.2f}{1}'.format(amount, currency)
    return dict(format_price=format_price)

```

### 21、过滤器
```
模板中可以使用过滤器|来修改变量的值。下面的示例使用内置的title过滤器 将name变量中每个单词的首字母转换为大写：

tpl = '{{ name|title  }}'
print render_template_string(tpl,name='jimi hendrix') #Jimi Hendrix
过滤器级联 ：可以将多个过滤器串联起来，构成过滤流水线。下面的示例对name 变量依次使用了两个过滤器，scriptags过滤器用来除去name变量中的HTML标签， title过滤器将传入的字符串中每个单词的首字母转换为大写：

tpl = '{{ name|striptags|title  }}'
print render_template_string(tpl,name='<h1>jimi hendrix</h1>') #Jimi Hendrix
过滤器参数 ：可以使用小括号为过滤器传入额外的参数。下面的示例将列表型变量 的多个成员使用join过滤器连接起来：

tpl = '{{ seq | join("-") }}'  
print render_template_string(tpl,seq=[1,2,3]) # 1-2-3
在Jinja2中，一个过滤器其实就是一个函数，第一个参数用来接收前序环节传入的值，而 返回值则作为后续环节过滤器函数的第一个参数：

Jinja2内置了很多过滤器，在其官网文档页 可以了解详细情况。
```

### 22、定制过滤器
```
我们已经知道，过滤器其实就是一个函数。在Flask中，可以使用Flask.template_filter 装饰器创建自己的过滤器。下面的示例创建了一个名为reverse的串反转过滤器，它总是 将输入的字符串逆向重排：

@app.template_filter('reverse')
def reverse_filter(s):
    return s[::-1]
下面的示例演示了如何调用我们自制的过滤器：

@app.route('/')
def index():
    return render_template_string('{{ greeting | reverse }}',greeting='Hello, Jinja2' )
另一种等价地创建定制过滤器的方法是将过滤器函数添加到Flask应用实例的jinja_env字典中：

def reverse_filter(s):
    return s[::-1]
app.jinja_env.filters['reverse'] = reverse_filter

```

## 棉花冰杯
```js
var webmianhua = {
	author: '棉花冰杯(勺哥)',
	qq: 136733282,
	weixin: 'mianhuabingbei',
	website: 'http://web.pxuexiao.com'
}
```