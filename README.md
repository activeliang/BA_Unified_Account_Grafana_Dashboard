这是一个docker上运行的基于grafana+influxdb的仪表盘。主要用来显示币安统一账户的资金曲线、最后下单时间。当然你也可以添加监控警报。


### 部署及使用方法：
1. 克隆本仓库
2. 复制config.py.example并重命名去掉末尾的.example，然后配置里边的账号列表信息，
3. 复制vars.env.example并重命名去掉末尾的.example，然后修改里边的环境参数：
    - password和token改成复杂一点的
    - 其中CRONTAB_SCHEDULE是定时扫描账户的频率，目前是默认配置是1分钟扫描一次，你也可以改成其他的，但请小于4小时间隔（关于cron定时的格式，可以问gpt)
    - 其他的参数按需改 (其中BUCKET参数如果修改了，需要在模板里修改成对应的BUCKET，这个参数建议不改)
4. 安装docker环境：
    ```
    # 安装docker 
    wget -qO- https://get.docker.com | sh
    # 安装docker-compose
    curl -L "https://github.com/docker/compose/releases/download/1.29.2/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
    chmod +x /usr/local/bin/docker-compose
    ```
5. 在本项目根目录执行：`docker-compose up -d` 即可运行, 首次可能需要等待大概3分钟才完成启动，访问3000号端口，进入主界面，输入密码，默认账户和密码为admin/admin
6. 进入路径/profile下，找到Language项，设置成中文，然后保存。
7. 添加数据源，进入路径/connections/datasources/new下，找到influxdb，进入添加页面：
    - Query language选择 Flux
    - HTTP中的URL填：http://influxdb:8086
    - Custom HTTP Headers点击`Add header`, 左边Header填Authorization，右边Value填入`Token <db_token>` (这里的db_token是你在第3步中配置的token)
    - InfluxDB Details中，Organization、Token填入第3步中配置的
    - 点蓝色按钮『Save % test』，不出意外的话会提示『datasource is working. 3 buckets found』代表OK了
8. 添加模板：
    - 进入路径/dashboards下，右边新增-导入，上传本仓库中grafana_template目录下的模板即可。
    - 重新进入路径/dashboards下，应该就能看到仪表盘了，点进去就是图表了。(其中部分图表可能需要24小时有足够数据才能正常展示出来)


### 代码更新：
- 日后若本仓库更新，如果更新部署，你只需要拉取最新代码，然后执行`docker-compose down && docker-compose up -d  --build`即可。（这个指令会先停止docker-compose并重构镜像并重启）


### 关于docker-compose的一些常用方法：
> 以下指令需要在项目根目录下执行。

- 检查容器状态：
  - `docker ps`

- 启动docker-comopose:
  - `docker-compose up` 前台启动，临时启动
  - `docker-compose up -d` 后台启动版本

- 进入容器内部： 
  - `docker-compose exec python-cron bash` （针对已经启动容器）
  - `docker-compose run python-cron bash`  （针对未启动容器)

- 查看docker-compose运行日志
  - `docker-compose logs -f`

- 停止docker-compose:
  - `docker-compose down`