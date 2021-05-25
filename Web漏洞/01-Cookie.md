# Cookie相关

Cookie保护是整个Web安全体系非常重要的一环，因为Cookie表示着用户的身份标识

## JavaScript获取Cookie

- 获取 - document.cookie
- 设置 - document.cookie = “username=zhangsan”
  - 如果当前的cookie不存在，则新增一个cookie
  - 当cookie名相同时，表示修改当前cookie
  - 将cookie的过期时间设置为过去的时间，表示删除cookie