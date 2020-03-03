# GDIndex

![preview](https://i.imgur.com/ENkZwCU.png)

> GDIndex 是一个类似 [GOIndex](https://github.com/donwa/goindex) 的东西，可以在 CloudFlare Workers 上架设 Google Drive 的目录，并提供许多功能
>
> 另外，这个并不是从 GOIndex 修改来了，而是直接重写

[Demo](https://gdindex-demo.maple3142.workers.dev/)

## 和 GOIndex 不同之处

-   前端使用 Vue 完成
-   查看图片不用另开新窗口
-   视频播放器支持字幕(目前只支持 srt)
-   支持在线阅读 PDF, EPUB
-   不支持目录加密(.password)
-   支持 Http Basic Auth
-   无需修改程序，即可接入多个云端硬盘(个人、团队)
-   支持导出文件下载链接
-   支持导出 Aria2 下载

## 使用教学

1. 安装 [rclone](https://rclone.org/)
2. 设定 Google Drive: https://rclone.org/drive/
3. 执行 `rclone config file` 以找到你的 `rclone.conf`
4. 在 `rclone.conf` 中寻找 `refresh_token` 以及 `root_folder_id` (可选)
5. 复制 [worker/dist/worker.js](worker/dist/worker.js) 的内容到 CloudFlare Workers
6. 在脚本顶端填上 `refresh_token`, `root_folder_id` 以及其他的选项
7. 部署!

或

1. 访问 [Albany MOD](https://gdindex-albanymod-builder.glitch.me/)
2. 根据网站提示生成CF Workers的代码
3. 将生成的代码复制到Workers项目

### 启用文件下载链接导出

1. 在 `worker.js` 中添加:
    ```
    export_url: true
    ```
2. 重新部署即可

### 启用 Aria2 下载

1. 在 `worker.js` 中添加配置 `download_aria2: true`：
    ```
	default_root_id: '...',
	client_id: '...',
	client_secret: '...',
	refresh_token: '...',
    ...
	download_aria2: true
    ```
2. 重新部署，此时你应该可以在文件列表上方看到“使用 Aria2 下载”以及“Aria2 RPC 配置”两个按钮
3. 在“Aria2 RPC 配置”中填写 Aria2 RPC 连接信息
4. 前往你要下载的文件夹，点击“使用 Aria2 下载”，开始添加下载任务

#### 解决 HTTPS 连接问题

当部署在 CloudFlare 上时，应用的内容（页面及文件信息）都以 HTTPS 方式传送。然而，你可能会使用 HTTP 连接 aria2。但在 HTTPS 页面中获取 HTTP 内容会被浏览器阻止。为了解决此问题，你可以：

- 通过签署证书的方式使用 HTTPS 方式连接 aria2
- 在浏览器中允许“不安全内容”

我们建议选择后者。在 Chrome 中，你可以：

1. 点击 URL 左边的 HTTPS 锁
2. 选择“站点设置”
3. 将“不安全内容”设置为“允许”

### 启用按需复制

Google Drive 限制了每个用户分享文件的流量（一般是每天 750GB）。如果你试图从超出流量限制的用户处下载一个分享文件，会收到 `403 - forbidden` 错误。将文件复制一份可能可以解决此问题，但是在网页端上一次操作只能复制一个文件。

因此，我们引入了“按需复制”功能。

1. 新建一个用于存放所有复制文件的文件夹。打开“开发者工具”获得这个文件夹的 ID
2. 添加以下配置到你的 `worker.js`：

```
  ...
  copy_on_forbidden: true,
  copy_parent_id: 'YOUR_COPY_FOLDER_ID' // 替换 YOUR_COPY_FOLDER_ID 为你的文件夹 ID
```

3. 正常开始下载即可，如果 worker 检测到文件分享流量超出限制，会自动复制一份到之前的文件夹并返回复制的文件给你。该过程是透明的，因此你无需进行额外处理

注意：请在下载完一段时间后删除复制的文件，否则，随着复制的文件越多，它们会占用更多的空间。另外，复制过程中不会检测是否已有复制的文件，因此多次下载会触发多次复制动作。
