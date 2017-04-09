---
layout: post
title:  "实施 Jinja2 模板(407-6)"
categories: Linux
tags: RHCA 407
---

### 描述 Jinja2 模板

###### 分隔符

*    {% EXPR %} 用于表达式或逻辑（如循环）
*    {{ EXPR }} 则用于向最终用户输出表达式或变量的结果

###### 控制结构

1. 循环

```
{% for user in users %}
    {{ user }}
{% endfor %}
```

```
loop.index 变量扩展至循环当前所处的索引号。它在循环第一次执行时值为 1，每一次迭代递增 1。 

{% for myuser in users if not myuser == "Snoopy"%}
    {{loop.index}} - {{ myuser }}
{% endfor %}
```

2. 条件

```
{% if finished %}
    {{ result }}
{% endif %}
```

###### 变量过滤器

Jinja2 提供了过滤器，更改模板表达式的输出格式（例如，输出到 JSON）。有适用于 YAML 或 JSON 等语言的过滤器。

```
to_json 过滤器使用 JSON 格式化表达式输出，而 to_yaml 过滤器则将 YAML 用作格式化语法。
{{ output | to_json }}
{{ output | to_yaml }}

to_nice_json 和 to_nice_yaml 过滤器，它们将表达式输出格式化为 JSON 或 YAML 等人类可读格式。
{{ output | to_nice_json }}
{{ output | to_nice_yaml }}

from_json 和 from_yaml 过滤器要求 JSON 或 YAML 格式的字符串，并对它进行解析。
{{ output | from_json }}
{{ output | from_yaml }}
```

用于测试返回值的内置 Ansible 过滤器包括 failed、changed、succeeded 和 skipped。以下任务演示了如何在条件表达式内使用过滤器。

```
tasks:
... Output omitted ...
  - debug: msg="the execution was aborted"
    when: returnvalue | failed
```

### 实施 Jinja2 模板

###### 构建 Jinja2 模板

Jinja2 模板由多个元素组成：数据、变量和表达式。在呈现 Jinja2 模板时，这些变量和表达式被替换为对应的值。模板中使用的变量可以在 playbook 的 vars 部分中指定。可以将受管主机的事实用作模板中的变量。 

> 可以使用 ansible system_hostname -i inventory_file -m setup 命令来获取与受管主机相关的事实。

```
Welcome to {{ ansible_hostname }}.
Today's date is: {{ ansible_date_time.date }}.
```

```
{% for myhost in groups['myhosts'] %}
{{ myhost }}
{% endfor %}
```

###### 在 Playbook 中使用 Jinja2 模板

Jinja2 模板是功能强大的工具，可用于自定义要在受管主机上部署的配置文件。创建了适用于配置文件的 Jinja2 模板后，它可以通过 template 模块部署到受管主机上，该模块支持将控制节点中的本地文件转移到受管主机。

若要使用 template 模块，请使用下列语法。与 src 键关联的值指定来源 Jinja2 模板，而与 dest 键关联的值指定要在目标主机上创建的文件。

```
  tasks:
    - name: template render
      template:
        src: /tmp/j2-template.j2
        dest: /tmp/dest-config-file.txt
```


### Summary

*    Ansible 使用 Jinja2 模板系统来呈现文件，然后再将文件分发到受管主机。
*    YAML 允许在 playbook 定义内使用基于 Jinja2 的变量，并可带有或不带有过滤器。
*    Jinja2 模板由两个元素组成，即变量和表达式。在呈现 Jinja2 模板时，这些变量和表达式被替换为对应的值。
*    通过 Jinja2 中的过滤器，模板表达式可以从一种数据转换为另一种。
