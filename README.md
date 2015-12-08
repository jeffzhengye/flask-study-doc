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
