#Gitlab 持续集成
从Gitlab 8.0开始, Gitlab把之前独立的项目 gitlab-ci 集成到Gitlab里面, 并且默认为所有的项目启用.


只要你的项目代码根目录下有.gitlab-ci.yml文件, 并且项目配置了相关的runner, 每天代码提交或合并, 就会触发构建.


Environments
就是我们通常说的测试环境, 生产环境等的'环境', 是代码部署的地方.

Deployments
部署
