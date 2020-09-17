# 在项目中使用 travis

项目中添加 .travis.yml 文件：

```yaml
language: go

go:
  - "1.14"
env:
  - GO111MODULE=on

install: true

script:
  - go fmt ./... && [[ -z `git status -s` ]]
  - echo 'start unit-test'
  - go mod vendor && go test ./... -coverprofile=coverage.txt -covermode=atomic

notifications:
  webhooks: https://oapi.dingtalk.com/robot/send?access_token=

```

