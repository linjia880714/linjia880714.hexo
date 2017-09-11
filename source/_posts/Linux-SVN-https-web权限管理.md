---
title: Linux SVN(https+web权限管理)
date: 2017-09-11 18:10:25
tags: linux,svn,http,https,web,权限管理
categories:
- IT
- Linux
---

<!-- toc -->

----

# 创建仓库
```bash
svnadmin create /data/svndata/test -- test

#修改权限, 由于使用apache访问，所以修改成apache的启动用户
chown -R apache:apache /data/svndata/test
```

# apache配置SVN模块
1. 修改 /etc/httpd/conf/httpd.conf文件 增加
```bash
Include conf/svn.conf
```

2. 创建/etc/httpd/conf/svn.conf
```bash
LoadModule dav_svn_module modules/mod_dav_svn.so
LoadModule authz_svn_module modules/mod_authz_svn.so

<Location /svn/test>
        DAV svn
        SVNPath /data/svndata/test
        SVNListParentPath on

        AuthName "welcome to Medica svn"
        AuthType Basic
        AuthUserFile /data/svndata/svn-auth.htpasswd
        AuthzSVNAccessFile /data/svndata/svn-access
        Require valid-user
</Location>
```

| name | description | remark |
| ----  | ---- | ---- |
| SVNPath | 仓库地址 | |
| AuthUserFile | 用户名和密码 | 使用web管理界面配置 |
| AuthzSVNAccessFile | 权限管理 | 使用web管理界面配置 |

# iF.SVNAdmin - Web管理界面（用户和权限管理）

iF.SVNAdmin是PHP写的，放到apache运行，且配置好如下配置

{% asset_img 01.png %}

