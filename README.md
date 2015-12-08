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
