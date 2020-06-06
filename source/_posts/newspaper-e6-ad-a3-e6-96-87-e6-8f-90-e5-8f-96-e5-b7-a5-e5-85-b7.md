---
title: Newspaper 正文提取工具
url: 1581.html
id: 1581
categories:
  - DEV
date: 2020-02-02 23:46:15
tags:
---

最近想做一个文章的统一检索，但是发现对正文很难进行统一的提取，所以就在 github 上去找轮子。

找了很久，终于找到一个合适的，可以进行页面的正文提取，这样就可以抓取里面的文章的内容，并且进行入库。

* * *

项目地址：[https://github.com/codelucas/newspaper](https://github.com/codelucas/newspaper)

项目文档：[https://newspaper.readthedocs.io/en/latest/](https://newspaper.readthedocs.io/en/latest/)

* * *

其使用的依赖项目在git的说明里面已经给出了，基本到了上手直接使用的级别。

自己本来是打算最终打包成一个 docker 镜像的，但是由于家里的网实在不好，在 `pip install`的过程中老是因为链接超时失败。然后去 docker hub 上去搜了一下，发现已经有了 `newspaper-api` 这个项目，提供了文件解析的API级别的服务

*   [https://hub.docker.com/r/smarp/newspaper-api](https://hub.docker.com/r/smarp/newspaper-api)

到时候直接使用就OK，但是看了一下代码，好像是没有返回正文抓取的内容不开心，后面加上吧。

*   [https://github.com/Smarp/newspaper-api/blob/cb6f2d21028579d4f084361f3b7879a38be904d2/src/server.py#L102](https://github.com/Smarp/newspaper-api/blob/cb6f2d21028579d4f084361f3b7879a38be904d2/src/server.py#L102)

* * *

具体的思路也是使用 API的方法来提供网页的解析服务，这里贴一下核心代码：

### Server服务部分

    route_map = {
        '/q':cgifunc.content_extract,
        '/q_s': cgifunc.page_extract,
    }
    
    def application(environ, start_response):
        environ['PATH_INFO'] = '/' + environ['PATH_INFO'].split('/')[-1]
        print(environ['PATH_INFO'])
        if environ['PATH_INFO'] in route_map:
            start_response('200 OK', [('Content-Type', 'text/html')])
            resp=route_map[environ['PATH_INFO']](environ)
        else:
            start_response('404 NOT FOUND', [('Content-Type', 'text/html')])
            return ""
        if not resp:
            return [b'Hello!']
        else:
            if type(resp) == type(''):
                resp = [resp.encode()]
            return resp
    
    if __name__ == '__main__':
        httpd = make_server('', 18000, application)
        print('Serving HTTP on port 18000...')
        httpd.serve_forever()

### 解析Handle部分

    def content_extract(environ):
        if environ['REQUEST_METHOD'] == "GET":
            logging.info('post update')
            try:
                request_body_size = int(environ.get('CONTENT_LENGTH', 0))
            except (ValueError):
                request_body_size = 0
            request_body = environ['wsgi.input'].read(request_body_size)
            params = dict(urllib.parse.parse_qsl(environ['QUERY_STRING']))
            print(params)
            try:
                page_url = params['url']
            except KeyError as e:
                print(repr(e))
                return repr(e) 
            try:
                article = Article(page_url,keep_article_html=True)
                article.download()
                article.parse()
                article.nlp()
                resp = {
                    "title": article.title,
                    "authors": article.authors,
                    # "keywords": [i for i in article.keywords if len(i)<16],
                    "keywords": article.keywords,
                    "summary": article.summary,
                    # "html": article.html,
                    "text": article.text,
                    "article_html": article.article_html,
                }
                try:
                    resp["publish_date"] = article.publish_date.strftime('%Y-%m-%d %H:%M:%S')
                except Exception as e:
                    resp["publish_date"] = datetime.now().strftime('%Y-%m-%d %H:%M:%S')
                print(resp['title'],resp['authors'],len(resp['article_html']))
                resp = json.dumps(resp)
            except Exception as e:
                print(repr(e))
                return repr(e) 
            return resp

### Dockerfile

    FROM python:3.7.6-alpine3.10
    ENV LANG C.UTF-8
    COPY ./requirements.txt /tmp/./requirements.txt
    
    RUN pip3 install -r /tmp/./requirements.txt
    EXPOSE 18000
    COPY ./webhook.py /home/app/
    CMD ["python", "/home/app/webhook.py"]