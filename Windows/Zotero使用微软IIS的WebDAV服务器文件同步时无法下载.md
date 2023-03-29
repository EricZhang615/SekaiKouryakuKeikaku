# Zotero使用微软IIS的WebDAV服务器文件同步时无法下载

## 问题描述

按照教程在Windows 10上设置好IIS和自带的WebDAV文件服务器后，在Zotero中设置文件同步时，点击验证服务器报错：

```
Zotero 在您的 WebDAV 服务器上发现了一个潜在的问题。
上传的文件将不能立即下载。在您上传文件直至这些文件在服务器上可用之间有短暂的延迟，特别是当您使用云存储服务时。
如果Zotero文件同步正常工作，您可以忽略些条信息。如果您有问题，请在Zotero论坛中发帖。
```

英文：
```
A potential problem was found with your WebDAV server.
An uploaded file was not immediately available for download. There may be a short delay between when you upload files and when they become available, particularly if you are using a cloud storage service.
If Zotero file syncing appears to work normally, you can ignore this message. If you have trouble, please post to the Zotero Forums.
```

之后在一台客户端上可以上传附件，并且在WebDAV服务器上可以看到成功上传的文件；但是在需要同步的客户端上同步时只能同步文档信息，附件没有下载下来。

在WebDAV服务器上手动点击链接尝试下载，发现压缩文件可以下载，但是`.prop`文件报错404。

## 解决方法

https://forums.zotero.org/discussion/12918/

作为一项安全措施，IIS不会向GET请求提供未知的文件类型。

需要为未知扩展名的文件明确注册一个默认的MIME类型。在IIS管理器中，进入`zotero`文件夹，打开`MIME类型`功能。添加一个新的规则：拓展名为`.*`, MIME Type为`application/octet-stream`。现在，该文件夹和子文件夹中的所有文件将被提供服务。