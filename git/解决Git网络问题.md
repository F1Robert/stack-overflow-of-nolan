# **解决Git网络问题**

# **unable to access 'https://github.com': Failed to connect to github.com port 443 after 21082 ms: Couldn't connect to server**



**以上问题出现，是因为git需要使用网络代理进行访问，**

**需要对git本身进行网络代理配置**

**本地梯子可以使用以下方法进行配置**

```
git config --global http.proxy 127.0.0.1:7890
```

**配好之后重新连接github**

**会提示认证和登录，按提示进行即可**

**至此问题解决**

**总结:**

**在开启vpn的情况下访问 github时，**

**需要为github自身配置代理服务器和端口**