---
layout: post
title:  "管理项目配额(110-3)"
categories: Linux
tags: RHCA 110
---

### 管理配额

###### 配额

在 OpenStack 中，管理员可以配置配额来防止系统资源被用尽。项目配额以每个项目为基础设置，可限制分配给该项目的资源数量。这些运行上的限制可以让管理员密切控制 OpenStack 平台中的项目。例如，可以控制一个项目拥有的 RAM 数量，从而防止客户端提交超出需要的内存，或者使用超过其合约所允许的数量。每个项目都有一组默认的配额值。可以为新项目修改这些默认值，也可以为现有项目更新它们。

###### 查看默认项目配额

1. 以 admin 用户身份登录 Horizon 控制面板。
2. 在管理选项卡中，选择系统选项卡，再单击 Defaults。
3. 此时将显示默认的配额值。

###### 更新项目配额

1. 以 admin 用户身份登录 Horizon 控制面板。
2. 在管理选项卡中，选择系统选项卡，再单击 Defaults。
3. 选择 Update Defaults。
4. 默认配额可以在 Update Default Quotas 窗口。
5. 单击更新默认值按钮。

###### 默认配额参数

*    参数 	                        描述
*    Security-group-rules 	        每个安全组的规则数
*    内核数 	                        每个项目允许的实例核心数
*    fixed-ips 	                    每个项目允许的固定 IP 地址数
*    floating-ips 	                每个项目允许的浮动 IP 数
*    injected-file-content-bytes 	每个项目允许的内容字节数
*    injected-file-path-bytes 	    所注入文件路径的长度
*    injected-files 	            每个项目允许注入的文件数
*    instances 	                    每个项目允许的实例核心数
*    key-pairs 	                    每个用户允许的密钥对数
*    metadata-items 	            每个实例允许的元数据项数
*    ram 	                        每个项目允许的实例 RAM 的兆字节数
*    security-groups 	            每个项目允许的安全组数


### 验证配额功能

OpenStack 项目配额可以从 Horizon 和 CLI 进行验证，但课程的这一部分将重点阐述使用 Horizon 验证项目配额。为项目修改配额时，无论修改的值是什么，Horizon 都不会一开始就指出错误。修改的配额值是否足够取决于实例所请求的资源需求。例如，云管理员可能决定最好将项目的 RAM 配额限值设置为 2048 以节省资源。当云管理员尝试重新启动实例时，有些实例无法启动，因为存在与项目配额限值相关的错误。云管理员可能没有考虑到，根据所请求的类别，请求的 RAM 资源或许与项目的不同。仅仅为项目配置新配额不会指出错误。但是，当实例尝试使用项目配额时，如果请求超出配额，Horizon 中就会显示错误。

###### 项目配额选项

*    参数 	                        描述
*    security-group-rules 	        每个安全组的规则数
*    cores 	                        每个项目允许的实例核心数
*    fixed-ips 	                    每个项目允许的固定 IP 地址数
*    floating IPs 	                每个项目允许的浮动 IP 数
*    injected-file-content-bytes 	每个项目允许的内容字节数
*    injected-file-path-bytes 	    所注入文件路径的长度
*    injected-files 	            每个项目允许注入的文件数
*    instances 	                    每个项目允许的实例核心数
*    key-pairs 	                    每个用户允许的密钥对数
*    metadata-items 	            每个实例允许的元数据项数
*    ram 	                        每个项目允许的实例 RAM 兆字节数


### 总结

在本章中，您可以学到：

*    可以使用项目配额来限制资源访问。
*    项目配额只能通过具有管理特权的帐户创建和修改。
*    每个项目在创建时都具有默认的配额。














security-groups 	每个项目允许的安全组数


















