# Api Mock Server

1. API Blueprint Document Server，基于Nginx + Aglio
2. API Blueprint Mock Server，基于Drakov

## 启动

### 指定API Blueprint文档Git公有仓库

```shell
docker run --name api-mock -e "repository={repository}" -p 80:80 -p 8080:8080 -p 3000:3000 -d ghosts/api-mock
```

> 通过 `-e "repository={repository}"` 来指定仓库，具体使用时替换 `{repository}` 为正式仓库地址即可。

mac指令:

```shell
docker run --name api-mock -e "aglio=--theme-template triple" -v ~/.ssh:/root/.ssh -e "repository=git@github.com:chrisgong/api-mock.git" -e "password=Gc880316" -p 80:80 -p 8080:8080 -p 3000:3000 -d ghosts/api-mock
```

### 指定API Blueprint文档Git私有仓库

```shell
docker run --name api-mock -v ~/.ssh:/root/.ssh -e "repository={repository}" -p 80:80 -p 8080:8080 -p 3000:3000 -d ghosts/api-mock
```
> 通过 `-v ~/.ssh:/root/.ssh` 来讲本地的`private key`映射到Docker容器中的ROOT账号

### 指定本地API Blueprint文档目录

```shell
docker run --name api-mockt -v ${api-path}:/opt/api-blueprint -p 80:80 -p 8080:8080 -p 3000:3000 -d ghosts/api-mock
```

mac指令:

```shell
docker run --name api-mock -v /Users/gc/mac/tools/api-docs:/opt/api-blueprint -p 80:80 -p 8080:8080 -p 3000:3000 -d ghosts/api-mock
```

### 指定API Blueprint文档模板风格

```shell
docker run --name api-mock -e "aglio=--theme-template triple" -e "repository={repository}" -p 80:80 -p 8080:8080 -p 3000:3000 -d ghosts/api-mock
```
> 通过 `-e "aglio=--theme-template triple"` 来指定aglio生成HTML的风格，比如现在指定的就是`"triple"`。更多aglio相应内容可以查看[Aglio文档](https://github.com/danielgtaylor/aglio#executable)

mac指令:

```shell
docker run --name api-mock -e "aglio=--theme-template triple" -v /Users/gc/mac/tools/api-docs:/opt/api-blueprint -p 80:80 -p 8080:8080 -p 3000:3000 -d ghosts/api-mock
```
