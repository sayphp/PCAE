Travis-CI（Continuous Integration）是一款提供持续持续集成服务的工具，用以提高软件开发效率，构建和测试的自动化，PHP目前就支持travis-ci。

#### 简介

使用travis-ci需要具备一个github账号、以及托管项目的所有权。

travis-ci支持包括Linux、macOS、Windows三种虚拟环境的构建，默认使用Linux的ubuntu，详细参考资料[4]。

如果要自定义配置，细节很多，要大量翻阅文档，而如果只是使用默认配置，使用起来那是相当简单。

#### 使用

```yaml
# 设定当前项目所使用的语言
# 设定后，travis会自动加载php的默认配置，并使用phpunit启动测试
language: php
# 设定测试环境的php版本，可以指定多个
php:
  - 7.1.9
# 设定测试执行脚本，可以制定多个
script: 
  - php travis/test.php
```

在项目中，加入简单的.travis.yml后，在https://travis-ci.org/account/repositories开启项目，然后git commit、git push接口观察到实际效果。

#### 资料

[1]: http://www.ruanyifeng.com/blog/2017/12/travis_ci_tutorial.html	"阮一峰的travis-ci快速教程"
[2]: https://travis-ci.org/php/php-src	"PHP的Travis-CI"
[3]: https://docs.travis-ci.com/	"Travis-CI文档"
[4]: https://docs.travis-ci.com/user/reference/overview/	"Travis-CI构建环境"
[5]: https://docs.travis-ci.com/user/common-build-problems/	"Travis-CI常见问题"







