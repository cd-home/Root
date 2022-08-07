### Yaml

- [x] 大小写敏感
- [x] 空格缩进表示层级
- [x] 冒号后需要空格
- [x] \# 注释

数据结构

对象

~~~yaml
key: value
~~~

=> json

~~~json
{"key": "value"}
~~~

数组

~~~yaml
key:
	- item0
	- item1
	- item2
key: [item0, item1, item2]
~~~

=> json

~~~json
{
    "key": ["item0", "item1", "item2"]
}
~~~

纯量

~~~yaml
num: 12.12
debug: true
IsNull: ~
mydate: 2022-07-11
mystring: "Hello world"
~~~

引用(公共部分)

~~~yaml
defaults: &defaults
  adapter:  postgres
  host:     localhost

development:
  database: myapp_development
  <<: *defaults

test:
  database: myapp_test
  <<: *defaults
~~~

