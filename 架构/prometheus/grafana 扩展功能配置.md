# grafana 扩展功能配置



## 匿名登录配置

1. 新增 `grafana.ini` 文件

   ```ini
   # 匿名登录配置
   [auth.anonymous]
   # 是否启用
   enabled = true
   # 组织名称
   org_name = Main Org.
   # 权限
   org_role = Viewer
   ```

2. 启动grafana增加该配置文件

   ```shell
   docker run --privileged=true  -dt -p 3000:3000 \
    -v $(pwd)/grafana-storage:/var/lib/grafana \
    -v $(pwd)/grafana.ini:/etc/grafana/grafana.ini \
    --name grafana grafana
   ```

3. 重新启动 grafana