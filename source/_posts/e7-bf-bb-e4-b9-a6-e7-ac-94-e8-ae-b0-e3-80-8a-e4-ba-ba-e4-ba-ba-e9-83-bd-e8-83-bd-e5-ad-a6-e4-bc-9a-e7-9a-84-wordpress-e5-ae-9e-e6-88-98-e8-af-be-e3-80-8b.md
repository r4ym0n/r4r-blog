---
title: 翻书笔记 《 人人都能学会的 WordPress 实战课》
tags:
  - wordpress
url: 1586.html
id: 1586
categories:
  - 读本好书
date: 2020-02-05 21:19:49
---

好久没有好好的看一本书了，说明这段日子过的太浮躁。 现在是2020年初，特殊时期，导致了足不出户的现状，人呐闲久了总会想找点事情做。 这片博文算是一篇读书笔记，写的是自己在看 《 人人都能学会的 WordPress 实战课》这本书的过程的一些笔记。东西写的不是十分的高阶，所以就尽快读完写完了。

*   文档地址：https://www.easywpbook.com/
*   [提问的智慧](http://www.dianbo.org/9238/stone/tiwendezhihui.htm)
*   [X-Y Problem](https://coolshell.cn/articles/10804.html)

WP基本使用技巧
--------

### 站点优化

*   动静分离，静态文件与冬天文件进行分离，比如静态文件放在 COS 上进行托管
    
*   图片压缩来进行加载提速`Compress JPEG &PNG image`作为一个图片统一压缩的插件
    
*   启用站点预加载功能来实现，用户点击的预加载[WordPress Instant Articles](https://wordpress.org/plugins/instant-articles/)
    

* * *

*   使用 nginx 代替 apache 来作为 webserver
*   PHP 开启OPCache，使用操作码缓存提升运行速度
*   mysql 打开Query Cache，Mysql 查询缓存来提高速度

* * *

*   对象缓存 使用 Memcached 和 Batcache 进行缓存
*   WP Super Cache，html页面缓存
*   Super Static Cache 插件来实现页面静态化

* * *

*   关闭谷歌字体，来提升页面加载速度
*   Gravatar头像使用国内cdn 代替
*   图片使用LazyLoad
*   Smart WordPress 插件来开启 ETag

### 安全加固

一个系统的安全与否与它本身的价值相关

*   使⽤邮箱登陆， Email Login
*   关闭 XML-RPC
*   关闭 JSON Rest API

WP主题使用以及管理
----------

### 快速说明

前面主题安装使用的部分就直接跳过了。

*   主题分为 header/footer/siderbar/single 这三大部分
    
*   在indexphp 的文件中分为 `get_header/get_sidebar/get_footer` 这三个函数来分辨渲染这三个部分。
    
*   wp的文章主循环：
    
        <?php if ( have_posts() ) : ?>
        <?php while ( have_posts() ) : the_post(); ?>
        ... Display post content
        <?php endwhile; ?>
        <?php endif; ?>
          
        // 标题 替换为 <?php the_title();?>;
        // 摘要 替换为 <?php the_excerpt();?>;
        // 作者 替换为 <?php the_author();?>;
        // 作者标签的 href 的值替换为 <?php the_author_link();?> 
        // ⽇期 替换为 <?php the_date();?> 
        // 文章的永久链接： <?php the_permalink();?>
        
    

### 文件结构

WordPress 主题可能会有很多⽂件，不过总体来说可以将它们分为三类：

*   CSS 样式⽂件和 JS 脚本⽂件：style.css ；
    
*   函数⽂件：function.php ；
    
*   模板⽂件：index.php/home.php/single.php/etc.
    

* * *

Wordpress 的每个模块的调用关系在下面的

*   [wphierachy](https://wphierarchy.com/)

### 主题开发

关于页面的功能不仅仅是自己编写的，里面的内容可以使用自己预先使用 php 编写的模板。来实现归档页面，或者友链词云之类的功能。这里我直接 COpy一下书里的一个例子来，对开发的内容来进行说明：

    <?php
    /*
    Template Name: 归档
    */
    function _PostList($atts = array())
    {
      global $wpdb;
      $rawposts = $wpdb->get_results("SELECT ID, year(post_date) as post_year, post_date, post_title FROM $wpdb->
      posts WHERE post_status = 'publish' AND post_type = 'post' AND post_password = '' order by post_date desc");
    	foreach ($rawposts as $post) {
      	$posts[$post->post_year][] = $post;
      }
      
      $rawposts = null;
      $html = '<div class="archives-container"><ul class="archives-list">';
      
      foreach ($posts as $year => $posts_yearly) {
        $html .= '<li><div class="archives-year">' . $year . '年</div><ul class="archives-sublist">';
        foreach ($posts_yearly as $post) {
          $html .= '<li>';
          $html .= '<time datetime="' . $post->post_date . '">' . mysql2date('m⽉d⽇ D', $post->post_date, true) . '</time>';
          $html .= '<a href="' . get_permalink($post->ID) . '">' . $post->post_title . '</a>';
          $html .= "</li>";
        }
        $html .= "</ul></li>";
      }
    	  $html .= "</ul></div>";
      	return $html;
    }
      
      function _PostCount()
      {
      	$num_posts = wp_count_posts('post');
    	  return number_format_i18n($num_posts->publish);
    	}
    	get_header();
    ?>
    <div class="row">
    <div class="col-sm-12 blog-main">
    <?php echo _PostList();
    ?>
    </div>
    </div>
    </div>
    <?php get_footer();?>
    

可以在代码的内容看到，这里的页面模板实际上是一个进行数据库查询，之后把结果在页面上进行渲染的功能，这里的输出内容是直接使用的html 来进行拼接生成。

* * *

另外一个简单的例子是实现一个标签云，来实现文章的tag统一展示。

    <?php
    /*
    Template Name: 标签云
    */
    get_header();?>
    <div class="row">
    	<div class="col-sm-12 blog-main">
    	 <?php wp_tag_cloud("smallest=20&largest=50&unix=px&number=200");?>
     	</div>
     </div>
     </div>
    <?php get_footer();?>
    

这里是直接使用 的Wordpress的内置函数，来实现一个标签云。

* * *

[GenerateWP](https://generatewp.com/generator/) 可以直接生成一些常用的 wordpress的代码，

WP插件开发
------

### WP运行机制

从 `index.php` 文件开始 加载 `wp-blog-header.php` ,再加载 `wp-load.php` h和 `template-loader.php` , 继续加载 `wp-config.php` 之后是 `wp-settings.php` ，

在 setting 里面对已启用的插件来进行加载。

* * *

插件是通过Wordpress提供的API函数来实现继承的，之间的耦合挺弱的，相关的函数可以在 `wp-includes/plugin.php` 中来看到API函数，类似于HOOK的方法，来hook到每个动作上去。

其中的hook分 `action/filter` ，前者勾动作消息后执行，后者是接到消息处理。

*   do\_action/add\_action [action 列表](https://codex.wordpress.org/Plugin_API/Action_Reference)
*   apply\_filters/add\_filter [filter列表](https://codex.wordpress.org/Plugin_API/Filter_Reference)

* * *

### 配置项目入库

    /**
    * 初始化设置项
    */
    function gitchat_copyright_activate() {
    	add_option('gitchat_copyright_code','<hr><p>这是⼀个来⾃ GitChat 达⼈课的插件</p><hr>');
    }
    register_activation_hook(__FILE__,'gitchat_copyright_activate' );
    
    // 在插件应用数据库的值的时候
    $content .= get_option('gitchat_copyright_code');
    

### 短代码插件

shortCode 这里可以理解为一个洪，允许用户插入在页面中的地方，之后会在渲染页面的时候会自动的进行替换。

    // 不带参数与内容
    // [git]
    function gitchat_git_shortcode() {
    	return "Hello World";
    }
    add_shortcode( 'git', 'gitchat_git_shortcode' );
    
    // 带参数与内容
    [git id='222' title='jah']this is a test[/git]
    function gitchat_git_shortcode( $atts , $content = null ) {
      $atts = shortcode_atts(array(
        'id' => '1',
        'title' => 'hahaha',
        ),$atts,'git');
      return $atts['id']."__".$atts['title']."---".$content;
    }
    
    // 使用 [git] 来使用短代码