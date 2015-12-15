# php redis安装遇到的坑

<div class="wiki-content">

    <h3 id="phpredis安装遇到的坑-今天安装了php的redis，安装过程："><span style="color: rgb(51,102,255);">今天安装了php的redis，安装过程：</span></h3>
    <p>mac 安装PHPredis扩展<br>redis的PHP的扩展在pecl网站上居然没有。查了下redis官方的clients页面，发现PHP的客户端居然有这么多，囧了呀。仔细看了一遍，发现官方推荐的客户端倒也不多，就只有两个，其中只有一个是用C语言作为PHP的扩展编写的。于是就装这个啦。<br>先下源码编译：<br>git
        clone <a href="git://github.com/nicolasff/phpredis.git"
                 rel="nofollow">git://github.com/nicolasff/phpredis.git</a><br>cd ./phpredis<br>phpize<br>make<br>make
        install<br>然后编写ini文件：<br>vim /opt/local/etc/php5/conf.d/php.ini<br>内容：<br>extension=<a href="http://redis.so"
                                                                                               class="external-link"
                                                                                               rel="nofollow">redis.so</a><br>重启php-cgi，然后看看phpinfo()吧，应该有redis了。<br>
        <br><span style="color: rgb(255,0,0);">安装完成以后，访问本机页面，还是一直报错（找不到redis）</span><br> <br>php -i | grep redis <br>一直显示redis
        被占用<br> <br>后来发现 /usr/local/etc/php/5.5/conf.d/目录下有一个 ext-redis.ini <br>然后就把
        /usr/local/etc/php/5.5/php.ini里面的extension=<a href="http://redis.so" class="external-link" rel="nofollow">redis.so</a>这句给去掉，就可以启动了。<br>
        <br><span style="color: rgb(255,0,0);">但是，启动之后又报了个错，还是没找到redis，如下图：</span><br><span
                class="confluence-embedded-file-wrapper"><img class="confluence-embedded-image"
                                                              src="/download/attachments/3375643/image2015-12-11%2020%3A10%3A11.png?version=1&amp;modificationDate=1449835807000&amp;api=v2"
                                                              data-image-src="/download/attachments/3375643/image2015-12-11%2020%3A10%3A11.png?version=1&amp;modificationDate=1449835807000&amp;api=v2"
                                                              data-unresolved-comment-count="0"
                                                              data-linked-resource-id="3375646"
                                                              data-linked-resource-version="1"
                                                              data-linked-resource-type="attachment"
                                                              data-linked-resource-default-alias="image2015-12-11 20:10:11.png"
                                                              data-base-url="http://wiki.5ixc.com"
                                                              data-linked-resource-content-type="image/png"
                                                              data-linked-resource-container-id="3375643"
                                                              data-linked-resource-container-version="4"></span><br>
        <br>然后在本机执行了 where php-fpm<br>如下：<br>/usr/local/sbin/php-fpm<br>/usr/sbin/php-fpm<br> <br>说明本机安装了两个版本的php（MAC默认安装了一个，自己又用brew重新安装了一个），可能是启动了另一个php-fpm，这下就尝试<br>sudo
        /usr/local/sbin/php-fpm 启动，打开页面访问成功<br> <br><span style="color: rgb(51,102,255);">最后，不想一直这样启动，就在PATH里面添加了/usr/local/sbin: 添加在了最前面就可以直接用 sudo php-fpm默认启动了</span><br>
        <br>可能会用到的命令：<br>php55-fpm -h # 查看php55-fpm常用命令<br>Usage: /usr/local/sbin/php55-fpm
        {start|stop|force-quit|restart|reload|status}<br> <br>sudo pkill -o USR2 php-fpm # MAC下直接杀掉指定程序</p>


</div>
