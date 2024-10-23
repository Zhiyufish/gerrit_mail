# gerrit_mail
修改gerrit的mail通知布局

前言：该功能是在设置对相应仓库关注后，用户进行相关操作后，能及时收到通知
操作包括：changes，patches，comments，submits，abandons

[图片]
用户只需查看订阅者部分
一.开发者
开发者需要在本地搭建Gerrit服务，以docker为例，设置好自己端口
docker run -it -p 8082:8080 -p 29518:29418  --entrypoint bash micr.cloud.mioffice.cn/miflow/gerritcodereview/gerrit:3.10.0-ubuntu22-root-nginx
拉取镜像后进入容器
docker exec -it
初始是没有配置邮件发送功能[sendemail]，需要进入gerrit.config进行配置
find / -name gerrit.config 2>/dev/null

sudo vim /var/gerrit/etc/gerrit.config
[图片]

[sendemail]
        #smtpServer = localhost
        enable = true
        smtpServer = smtp.qq.com
        smtpServerPort = 465
        smtpEncryption = SSL
        smtpUser = 绑定邮箱@qq.com
        from =Code Review<绑定邮箱@qq.com>
        sslVerify = true
        smtpPass = 你的授权码
获取授权码见第三部分  
配置好后，进行gerrit重启更新配置，由于使用docker进行部署gerrit，不能使用restart进行重启
 ps aux | grep gerrit #查看gerrit运行进程号
 kill -9 id  #关闭进程
 /var/gerrit/bin/gerrit.sh run #重启gerrit
现在配置好了
相关文档：
https://gerrit-review.googlesource.com/Documentation/config-gerrit.html#sendemail
二.邮箱开启smtp服务
可以使用gmail，qq邮箱，163邮箱进行配置
1. 登录QQ邮箱（mail.qq.com）。
2. 点击“设置”按钮，选择“账户”选项卡。
3. 在“POP3/IMAP/SMTP/Exchange/CardDAV/CalDAV服务”栏目下，找到“开启POP3/SMTP服务”选项，并点击“开启”按钮。
4. 按照提示绑定手机号码，并生成授权码。
5. 使用SMTP服务器：smtp.qq.com，端口号：465。
6. 填写完整的邮箱名作为身份认证用户名，填写QQ邮箱的授权码作为身份认证密码。
7. 选“是”进行SMTP身份认证，选“是”开启SSL加密。
[图片]

[图片]
[图片]
[图片]
[图片]
谷歌邮箱配置如下
https://www.cnblogs.com/jiyuwu/p/16313476.html
三.订阅者
开发者开启smtp邮件通知功能后，订阅者即可通过邮件获取到更改通知
（一）首先在gerrit设置界面进行相应配置
Email notifications   ：设置接收通知情况，包括接收所有人的通知，除我外的其他人，只有我关注的，及不接收通知
Emali format    ：   可以设置接收邮件通知的类型，选择HTML and plaintext
设置好后点击save changes
在进行选择的时候首先要进行修改，在Email notifications中选择 only when i an in the attention set,并save changes
这样在其他人修改相应订阅事件时，订阅者会收到邮件通知
[图片]
（二）进行通知的设置
邮件通知包含两种方式
（1）未订阅：
[图片]
当显示Bring to attention of 时，即使用户未订阅，仍可接收到该操作通知,点击右侧modify，能够选择是否通知

（2）进行仓库或分支的订阅
setting->emali addresses->notifications
[图片]
其中，Repo和branch分别填写仓库和分支名称，填写后点击ADD，可以进行勾选订阅事件
暂时无法在飞书文档外展示此内容
订阅后，在仓库进行修改后能够收到通知，下面以进行comment为例，能看出经过comment后，邮箱能够收到通知
[图片]
[图片]
四.邮件内容更改
为了对邮件效果进行提升，本文对邮件格式进行了优化
cd /var/gerrit/etc/mail
其中有若干配置文件，修改流程为
cp CommentHtml.soy.example CommentHtml.soy
# 这样只需要修改CommentHtml.soy，保存后，gerrit会自动更新配置文件
[图片]
soy文件是Google的Closure Templates模板文件，用于生成动态HTML内容，并支持模板继承，条件语句，循环等功能。以ChangeFooterHtml.soy为例，首先使用namespace生成名称空间，并调用template模板，接收两个参数change和email后续进行调用。
参数相关文档：
https://gerrit-review.googlesource.com/Documentation/config-mail.html#_change_emails

[图片]
 
评论通知                                                                                                      code-review通知                                          
[图片]

[图片]

                                            
删除reviewer                                                                                                     添加reviewer
[图片]

[图片]


废弃通知                                                                                                              恢复通知                                                                                 
[图片]

[图片]


push上传通知                                                                                              追加上传
[图片]
[图片]


submit提交                                                                                                       撤销提交
[图片]
[图片]

1. AddKeyHtml.soy：用于生成关于添加新密钥的 HTML 内容的模板。
2. AddToAttentionSetHtml.soy：生成将某个项目或用户添加到关注集的 HTML 内容。
3. ChangeFooterHtml.soy：定义更改页面底部内容的 HTML 模板。
4. ChangeHeaderHtml.soy：用于生成更改页面头部内容的 HTML 模板。
5. ChangeSubjectHtml.soy：生成更改主题的 HTML 内容。
6. CommentHtml.soy：生成评论内容的 HTML 模板。
7. DeleteKeyHtml.soy：生成关于删除密钥的 HTML 内容。
8. DeleteReviewerHtml.soy：生成关于删除审阅者的 HTML 内容。
9. DeleteVoteHtml.soy：定义删除投票的 HTML 内容模板。
10. InboundEmailRejectionHtml.soy：生成关于拒绝入站电子邮件的 HTML 内容。
11. HttpPasswordUpdateHtml.soy：生成关于 HTTP 密码更新的 HTML 内容。
12. MergedHtml.soy：定义合并操作的 HTML 内容。
13. NewChangeHtml.soy：生成关于新变更的 HTML 内容。
14. RegisterNewEmailHtml.soy：定义新电子邮件注册的 HTML 内容。
15. RemoveFromAttentionSetHtml.soy：生成将某个项目或用户从关注集移除的 HTML 内容。
16. ReplacePatchSetHtml.soy：定义替换补丁集的 HTML 内容。
17. RestoredHtml.soy：生成关于恢复操作的 HTML 内容。
18. RevertedHtml.soy：定义关于回退操作的 HTML 内容。

https://github.dev/GerritCodeReview/gerrit
