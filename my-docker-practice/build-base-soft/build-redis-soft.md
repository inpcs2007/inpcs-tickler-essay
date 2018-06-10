```
docker pull redis:4.0.9
```

```

```

### 运行镜像

```
docker run --name myredis  -p 6379:6379 -v $PWD/data:/data -d redis:4.0.9 redis-server --appendonly yes
```

## redis安全漏洞的排查与修复

http://blog.51cto.com/simeon/2115184

