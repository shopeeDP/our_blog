---
title: dp 项目部署说明
date: 2018-07-11 16:14:42
tags:
- nginx
categories:
- 项目文档
description: 举例阐述 dp 组前端项目的部署流程
---

> shopee 前端项目部署涉及的点比较多，且部署过程中有些环节对于我们开发人员来说是黑盒，很容易掉坑，为了降低大家的采坑概率，将自己摸索的经验总结如下

# 生产环境目录说明
shopee 中每个项目对应到生产环境中都会有如下所示的一个目录：

```
/data/shopee/{{ env }}/{{ project_name }}-{{ cid }}/
```

其中 `project_name` 是项目名称，对应到我们组就是 dp，`env` 是环境，可选项为 test, live 等，`cid` 是国家代号，可选项为 id(印尼), sg(新加坡), th(泰国) 等。由于 dp 项目下有很多前端子项目，如 sniper, windrunner, hub-admin 等，所以我们需要通过文件目录来区分各个项目，如下所示：

```
/data/shopee/test/dp-id# tree -L 2
...........
├── hub
│   ├── favicon.ico
│   ├── index.html
│   └── static
├── sniper
│   ├── asset-manifest.json
│   ├── index.html
│   ├── manifest.json
│   ├── service-worker.js
│   └── static
├── windrunner
│   ├── favicon.png
│   ├── index.20e07574.js
│   ├── index.e20816cc.css
│   ├── index.html
│   └── static
..........
```

所以，我们前端项目打包的路径就必须要进行调整，下面以 [hub-admin](https://git.garena.com/shopee/digital-purchase/hub-admin) 为例来说说单个项目如何打包：

# 项目打包设置
首先每个前端项目下必须要有一个 deploy 目录，下面包含项目部署配置文件，文件名必须同 jenkin 上的 module 名称一致。以 hub-admin 项目 test 环境为例，其在 jenken 上的名称为 [shopee-dp-hubstatic-test](http://jenkins.shopeemobile.com/view/dp/job/shopee-dp-hubstatic-test/)，所以 deploy 下的配置文件名应该为 hubstatic.json，下面看看该文件：

```json
{
	"project_dir_depth": 2,
	"project_name": "dp",
	"module_name": "hubstatic",
	"install": {
		"commands": [
		]
	},

	"build": {
		"docker_image": {
			"base_image": "harbor.shopeemobile.com/shopee/nodejs-base",
			"squash": false,
			"dependent_libraries_files": [
				"package.json"
			],
			"run_commands": [
				"npm install"
			]
    },
    "commands": [
			"npm run build"
		],
		"upload_static": {
			"static_dir": "dist",
			"enable_cdn": false
		}
	}
}
```

文件中配置的含义请参考[这里](https://docs.google.com/document/d/1Z9sqex8kgx0VLJwFe2VRWTsk7LZbTbhVlSWIgqjuhvQ/edit)，另外，如何申请一个新的 jenkin 部署项目请看[这里](https://docs.google.com/document/d/1Z64ERrLuVxXUsWB_JU8pETnLYwvlqUWewjX8enE_WCI/edit)。

其中， upload_static 下面的 static_dir 指定的路径下面的文件会在部署的时候全部拷贝到前一节所说的目录之下，具体来说就是 `/data/shopee/test/dp-id`，所以我们打包出来的结果应该如下所示：

```
└── hub
    ├── index.html
    └── static
        ├── css
        ├── fonts
        ├── img
        └── js
```

*index.html*
```html
<!DOCTYPE html>
<html>

<head>
  <meta charset=utf-8>
  <meta name=viewport content="width=device-width,initial-scale=1">
  <title>SHOPEE Hub Admin</title>
  <link rel="shortcut icon" href=/hub/favicon.ico>
  <!-- 这里的 basename "hub" 不一定要和上面的目录名 hub 一致，sniper 项目就是一个例子-->
  <link href=/hub/static/css/app.0edc75456c91cd763270c8d0c769e143.css rel=stylesheet>
</head>

<body>
  <div id=app></div>
  <script type=text/javascript src=/hub/static/js/manifest.ce3d40e477b8ed629793.js></script>
  <script type=text/javascript src=/hub/static/js/vendor.494ab495d29c7629ae55.js></script>
  <script type=text/javascript src=/hub/static/js/app.ba67cad897139a164db1.js></script>
</body>

</html>
```

具体如何设置请参考 webpack 官网或 dp 现有项目。

# nginx 配置
dp 组现有 sniper, windrunner, hub-admin 前端项目，下面详细说明下三个项目的 nginx 配置：

项目 | 访问地址 | nginx配置地址
---- | ---- | ----
sniper | https://shopee.co.id/produk-digital/m | [mall.conf.j2](https://git.garena.com/shopee/shopee-nginx-lb/blob/master/lb_templates/http/mall.conf.j2) 可以搜索 produk-digital 快速定位
windrunner | https://dp-admin.test.shopee.io/windrunner | [dp.conf.j2](https://git.garena.com/shopee/shopee-nginx-lb/blob/master/lb_templates/http/dp.conf.j2)
hub-admin | https://dp-admin.test.shopee.io/hub | [dp.conf.j2](https://git.garena.com/shopee/shopee-nginx-lb/blob/master/lb_templates/http/dp.conf.j2)

其中 sniper 是放在 shopee.co.id 域名下的，所以它的配置写在 mall.conf.j2 里面，而 windrunner 和 hub-admin 两者是在同一个域名下通过 pathname 来区分，所以他们的配置都写在 dp.conf.j2 里面。下面详细介绍一下他们的配置：


*sniper*

```
    ## begin of digital purchase ##
    {% if cid in ["id"] %} # 印尼
    location /produk-digital/m { # 形如 /produk-digital/m/*** 的路由都会被导流到 /produk-digital/m/index.html 下
        try_files     $uri      /produk-digital/m/index.html;
    }

    location = /produk-digital/m/index.html { # 上面的 location 都导流到了这里
        alias                   /data/shopee/{{ env }}/dp-{{ cid }}/sniper/index.html; # 所有非静态文件的请求都返回 index.html 文件

        access_log              off;
        add_header              Cache-Control "no-cache, no-store";
        expires                 -1;
        etag                    on;

        include                 gzip_params;
    }

    location /digital-product/static/ {
        # 请求 http://test.com/produk-digital/static/css/main.css
        # 会返回
        # /data/shopee/{{ env }}/dp-{{ cid }}/sniper/static/static/css/main.css
        alias                   /data/shopee/{{ env }}/dp-{{ cid }}/sniper/static/;

        access_log              off;
        expires                 30d;
        etag                    on;

        include                 gzip_params;
    }

    {% endif %}
    ## end of digital purchase ##

```

由于产品对访问 dp 项目的 url 有地区性的要求，所以印尼版的 url 路径为 `/produk-digital/m`，以后泰国可能为 `/ผลิตภัณฑ์ดิจิตอล/m`，而静态文件则没有这种要求，可以保持 `/digital-product/static/` 不变。

*dp.conf.j2*

```
    ## Start of hub_admin ##

    location /hub/static/ {
        alias                   /data/shopee/{{ env }}/{{ project_name }}-{{ cid }}/hub/static/;
        expires                 30d;
        access_log              off;

        include                 gzip_params;
    }

    location /hub {
        try_files     $uri      /hub/index.html;
    }

    location = /hub/index.html {
        alias                 /data/shopee/{{ env }}/{{ project_name }}-{{ cid }}/hub/index.html;
        expires               -1;
        etag                  on;
        include               gzip_params;
    }

    ## End of hub_admin ##

    ## Start of dp_admin_portal ##

    location /windrunner/static/ {
        alias                   /data/shopee/{{ env }}/{{ project_name }}-{{ cid }}/windrunner/static/;
        expires                 30d;
        access_log              off;

        include                 gzip_params;
    }

    location /windrunner {
        try_files     $uri      /windrunner/index.html;
    }

    location = /windrunner/index.html {
        alias                 /data/shopee/{{ env }}/{{ project_name }}-{{ cid }}/windrunner/index.html;
        expires               -1;
        etag                  on;
        include               gzip_params;
    }

    ## End of dp_admin_portal ##

```

各配置的含义同 sniper 类似，就不赘述了。

下面说明一下修改 nginx 配置的流程：

1. 从 master 分支上 checkout 一个分支。
2. 在 jira 上新建一个问题，格式如下所示：
![](/images/shopee-deploy/1.png)
3. 在新建的分支上进行修改，然后发起合并请求到 test 或者 master 分支，将 jira 单号写到合并请求标题中，如：`[SPTOS-882] live chang dp admin nginx configuration`。

# 总结
本文只是对现有的项目进行了一个介绍，以后遇到其他需求都可以参考现阶段项目的配置。在进行添加或修改之前最好在本地搭建一个 nginx 环境模拟一下，然后再更新到线上环境中。