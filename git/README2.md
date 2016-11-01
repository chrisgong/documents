# 删除

* `rm *.*` : 删除工作区文件
* `git rm *.*` : 删除工作区及暂存, 需要commit提交.

查看历史版本文件列表:

```
$ git ls-files --with-tree=HEAD^
```

查看历史版本中尚在的删除文件内容:

```
$ git cat-file -p HEAD^:*.txt
```

* `git add -u` : 快速标记删除， 将本地有改动（包括修改和删除）的文件记到暂存区。
