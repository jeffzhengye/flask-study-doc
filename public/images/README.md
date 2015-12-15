# 使用 Flask 搭建静态博客

<div class="entry">
    			<p>现在流行的静态博客/网站生成工具有很多，比如 Jekyll, Pelican, Middleman, Hyde 等等，<a href="https://www.staticgen.com">StaticGen</a> 列出了目前最流行的一些静态网站生成工具。</p>
<p>我们的内部工具由 Python/Flask/MongoDB 搭建，现在需要加上文档功能，写作格式是 Markdown，不想把文档放到数据库里，也不想再弄一套静态博客工具来管理文档，于是找到了 <a href="https://pythonhosted.org/Flask-FlatPages/">Flask-FlatPages</a> 这个好用的 Flask 模块。熟悉 Flask 的同学花几分钟的时间就可以用搭建一个简单博客，加上 Bootstrap 的帮助，不到一小时内就可以用 Flask-Flatpages 弄个像模像样的网站出来。</p>
<h2>创建开发环境</h2>
<p>首先我们需要 pip，在 Mac 上最简单的安装办法是：</p>
<pre class="code">$ sudo easy_install pip
$ sudo easy_install virtualenv
</pre>
<p>如果你在 Mac 上用 Homebrew 包管理工具的话的话，也可以用 brew 升级 Python 和安装 pip：</p>
<pre class="code">$ brew update
$ brew install python
</pre>
<p>创建一个 blog 目录、生成 Python 独立虚拟环境并在这个环境里安装需要的 Flask, Flask-FlatPages 模块：</p>
<pre class="code">$ mkdir blog
$ cd blog

$ virtualenv flask
New python executable in flask/bin/python
Installing setuptools, pip...done.

$ flask/bin/pip install flask
$ flask/bin/pip install flask-flatpages
</pre>
<p>在 blog 目录下我们分别新建几个目录：static 用来存放 css/js 等文件，templates 用来存放 flask 要用的 Jinja2 模版，pages 用来存放我们静态博客（Markdown 格式）：</p>
<pre class="code">$ mkdir -p app/static app/templates app/pages
</pre>
<h2>程序</h2>
<p>主程序 blog.py 的功能是，导入必要的模块、配置 Flask-FlatPages 模块需要的参数、创建 Flask 应用、写几个 URL 路由函数，最后运行这个应用：</p>
<pre class="code">$ vi app/blog.py
#!flask/bin/python
from flask import Flask, render_template
from flask_flatpages import FlatPages

DEBUG = True
FLATPAGES_AUTO_RELOAD = DEBUG
FLATPAGES_EXTENSION = '.md'

app = Flask(__name__)
app.config.from_object(__name__)
flatpages = FlatPages(app)

@app.route('/')
def index():
    pages = (p for p in flatpages if 'date' in p.meta)
    return render_template('index.html', pages=pages)

@app.route('/pages/&lt;path:path&gt;/')
def page(path):
    page = flatpages.get_or_404(path)
    return render_template('page.html', page=page)

if __name__ == '__main__':
    app.run(port=8000)
</pre>
<h2>模版</h2>
<p>在 Python 中直接生成 HTML 很繁琐并不好玩（那是上个世纪90年代的 PHP 搞的事情），在现代社会，我们使用模版引擎，Flask 已经自动配置好了 Jinja2 模版，使用方法 render_template() 来渲染模版就可以了。Flask 会默认在 templates 目录里中寻找模版，我们只需要创建几个模版文件就可以了，这里我们创建 base.html, index.html 和 page.html.</p>
<pre class="code">$ vi app/templates/base.html
&lt;!doctype html&gt;
&lt;html&gt;
&lt;head&gt;
    &lt;meta charset="utf-8"&gt;
    &lt;title&gt;vpsee.com static blog&lt;/title&gt;
&lt;/head&gt;

&lt;body&gt;
    &lt;h1&gt;&lt;a href="{{ url_for("index") }}"&gt;vpsee.com blog&lt;/a&gt;&lt;/h1&gt;
    {% block content %}
    {% endblock content %}
&lt;/body&gt;
&lt;/html&gt;
</pre>
<p>代码里 extends “base.html” 的意思是从 base.html 里继承基本的 “骨架”。</p>
<pre class="code">$ vi app/templates/index.html
{% extends "base.html" %}

{% block content %}
    &lt;h2&gt;List of pages
    &lt;ul&gt;
        {% for page in pages %}
        &lt;li&gt;
            &lt;a href="{{ url_for("page", path=page.path) }}"&gt;{{ page.title }}&lt;/a&gt;
        &lt;/li&gt;
        {% else %}
        &lt;li&gt;No post.&lt;/li&gt;
        {% endfor %}
    &lt;/ul&gt;
{% endblock content %}
</pre>
<pre class="code">$ vi app/templates/page.html
{% extends "base.html" %}

{% block content %}
    &lt;h2&gt;{{ page.title }}&lt;/h2&gt;
    {{ page.html|safe }}
{% endblock content %}
</pre>
<p>Flask-FlatPages 模块会默认从 pages 目录里寻找 .md 结尾的 Markdown 文档，所以我们把静态博客的内容都放在这个目录里：</p>
<pre class="code">$ vi app/pages/hello-world.md
title: Hello World
date: 2014-10-14
tags: [general, blog]

**Hello World**!

$ vi app/pages/test-flatpages.md
title: Test Flask FlatPages
date: 2014-10-15
tags: [python, flask]

Test [Flask-FlatPages](https://pythonhosted.org/Flask-FlatPages/)
</pre>
<h2>运行</h2>
<p>基本搞定，运行看看效果吧：</p>
<pre class="code">$ flask/bin/python app/blog.py
 * Running on http://127.0.0.1:8000/
 * Restarting with reloader
</pre>
<p style="overflow:hidden;"><img style="clear: both; float: left; width: 550px; display: block;" src="http://www.vpsee.com/wp-content/uploads/2014/10/static-blog-flask-flatpages.png" alt="build a static blog with flask"></p>
<p class="clr">
</p><h2>静态化</h2>
<p>到目前为止，上面的博客运行良好，但是有个问题，这个博客还不是 “静态” 的，没有生成任何 html 文件，不能直接放到 nginx/apache 这样的 web 服务器下用。所以我们需要另一个 Flask 模块 <a href="http://pythonhosted.org/Frozen-Flask/">Frozen-Flask</a> 的帮助。</p>
<p>安装 Frozen-Flask：</p>
<pre class="code">$ flask/bin/pip install frozen-flask
</pre>
<p>修改 blog.py，导入 Flask-Frozen 模块，初始化 Freezer，使用 freezer.freeze() 生成静态 HTML：</p>
<pre class="code">$ vi app/blog.py
...
from flask_flatpages import FlatPages
from flask_frozen import Freezer
import sys
...
flatpages = FlatPages(app)
freezer = Freezer(app)
...
if __name__ == '__main__':
    if len(sys.argv) &gt; 1 and sys.argv[1] == "build":
        freezer.freeze()
    else:
        app.run(port=8000)
</pre>
<p>运行 blog.py build 后就在 app 目录下生成 build 目录，build 目录里面就是我们要的 HTML 静态文件：</p>
<pre class="code">$ flask/bin/python app/blog.py build

$ ls app/
blog.py   build     pages     static    templates
</pre>
<p>更清晰的目录结构如下：</p>
<pre class="code">$ tree app
app
├── blog.py
├── build
│&nbsp;&nbsp; ├── index.html
│&nbsp;&nbsp; └── pages
│&nbsp;&nbsp;     ├── hello-world
│&nbsp;&nbsp;     │&nbsp;&nbsp; └── index.html
│&nbsp;&nbsp;     └── test-flatpages
│&nbsp;&nbsp;         └── index.html
├── pages
│&nbsp;&nbsp; ├── hello-world.md
│&nbsp;&nbsp; └── test-flatpages.md
├── static
└── templates
    ├── base.html
    ├── index.html
    └── page.html
</pre>
			</div>
