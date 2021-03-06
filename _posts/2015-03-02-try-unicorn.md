---
layout: post
title: 尝试Unicorn
description: "unicorn, ruby web服务器，socket"
---

## 前言

想尝试一下Unicorn，就使用公司最小的项目mobile试了一下，或者使用[wlog](https://github.com/windy/wblog)这个晚上随便找的项目。

## 正文

mobile的项目无从下手，所以，尝试wlog一半成功了。

```
RAILS_ENV=production rake db:setup
RAILS_ENV=production rake assets:precompile
```

Nginx的配置文件: 

```
upstream wblog {
  server unix:/tmp/unicorn_wblog.sock fail_timeout=0;
}

server {
  listen 80;
  server_name 192.168.1.239;
  root /home/ruby/wblog/current/public; # 注意这里不是项目的根目录，而是根目录下的public目录

  location ^~ /assets/ {
    gzip_static on;
    expires max;
    add_header Cache-Control public;
  }

  location ~ ^/(uploads)/  {
    expires max;
    break;
  }

  try_files $uri/index.html $uri @wblog;
  location @wblog {
    proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
    proxy_set_header Host $http_host;
    proxy_redirect off;
    proxy_pass http://wblog;
  }

  error_page 500 502 503 504 /500.html;
  client_max_body_size 20M;
  keepalive_timeout 10;
}
```

unicorn的配置文件:

```
app_path = "/home/ruby/wblog/current"

worker_processes   1 
preload_app        true
timeout            180                         # 审核worker进程
listen             '/tmp/unicorn_wblog.sock'
user               'ruby', 'ruby'
working_directory  app_path
pid                "#{app_path}/tmp/pids/unicorn_wblog.pid"
stderr_path        "log/unicorn.log"
stdout_path        "log/unicorn.log"

before_fork do |server, worker|
    old_pid = "#{server.config[:pid]}.oldbin"
    if File.exists?(old_pid) && server.pid != old_pid
      begin
        Process.kill("QUIT", File.read(old_pid).to_i)
      rescue Errno::ENOENT, Errno::ESRCH
        # someone else did our job for us
      end
    end
end

after_fork do |server, worker|
end
```

关于配置的一些说明: 

* 一核对应一个woker进程
* 无需root权限，master为root时，很麻烦


启动unicorn： `unicorn_rails -c config/unicorn/production.rb -D -E production`

> 备注： 花了点钱升级了服务器的内存，结果需要重启服务器，重启服务器时，发现了一个问题，unicorn没有设置开机自启动，有机会设置一下。

无缝重启: `kill -USR2 $(cat tmp/pids/unicorn_wblog.pid)`

遇到的问题: 

```
$ rake assets:precompile 
rake aborted!
NameError: uninitialized constant Rack::Cors
```

解决： 启动环境的问题。设置`RAILS_ENV=production`即可。后来，预编译失败了，是由于Fundation-rails的版本引起的，将版本由`5.5.0`切换为`5.4.5.0`，就能正常的编译了。

```
$ RAILS_ENV=production rake db:setup
rake aborted!
Moped::Errors::ConnectionFailure: Could not connect to a primary node for replica set #<Moped::Cluster:97556630 @seeds=[<Moped::Node resolved_address="127.0.1.1:27017">]>
```

解决： mongodb连接的问题，mongodb.conf中配置的是127.0.0.1, 应用程序中连接的是127.0.1.1。在`mongoid.yml`中将localhost修改为127.0.0.1即可。

```
production:
  sessions:
    default:
      database: w_blog_production
      hosts:
        - 127.0.0.1:27017
```

实践初步完成，接下来进行性能测试。

## mobile项目的测试

遇到了一个坑爹的错误：

```
*** [err :: 192.168.1.99] /var/www/apps/test/shared/bundle/ruby/1.9.1/gems/therubyracer-0.12.1/lib/v8/init.so: [BUG] Segmentation fault
*** [err :: 192.168.1.99] ruby 1.9.3p545 (2014-02-24) [x86_64-linux]
```

一看，是therubyracer的问题，将其注释掉，换成node，结果遇到一个更坑爹的错误: 

```
*** [err :: 192.168.1.99] rake aborted!
*** [err :: 192.168.1.99] ExecJS::RuntimeUnavailable: Could not find a JavaScript runtime. See https://github.com/sstephenson/execjs for a list of available runtimes.
```

魂淡，没道理啊，又来回注释了几遍，依然有错误。最后，上网查了一下，做了软连接: `ln -s /usr/local/nodejs/bin/node /usr/bin/node`，再次试了一下，我擦，居然通过了，execjs的
探测node的存在的路径是坑爹的啊，居然连PATH和/usr/local/bin都不识别。

参考: http://stackoverflow.com/questions/14187681/node-js-not-found-by-rails-execjs

遇到了问题: 

```
database configuration does not specify adapter (ActiveRecord::AdapterNotSpecified)
```

解决： 

部署脚本中环境配置设置错了，照抄的别人配置脚本并且不加思考，果然不靠谱。不过，总算是搞定了。


参考: http://stackoverflow.com/questions/413659/database-configuration-does-not-specify-adapter

## ab测试

ab测试的命令: `ab -n1000 -c10 http://115.28.165.58:5000/`

unicorn的配置为: 

```
worker_processes   1
preload_app        true
timeout            180
```

本地测试结果: 


```
Document Path:          /
Document Length:        5082 bytes

Concurrency Level:      10
Time taken for tests:   67.273 seconds
Complete requests:      1000
Failed requests:        835  # 这里的失败的请求数可能是由于重启Unicron的原因
   (Connect: 0, Receive: 0, Length: 835, Exceptions: 0)
Total transferred:      8858950 bytes
HTML transferred:       8044580 bytes
Requests per second:    14.86 [#/sec] (mean)
Time per request:       672.725 [ms] (mean)
Time per request:       67.273 [ms] (mean, across all concurrent requests)
Transfer rate:          128.60 [Kbytes/sec] received
```

远程主机测试结果(仅考虑Rails框架的动态处理能力): 

```
Document Path:          /
Document Length:        8630 bytes

Concurrency Level:      10
Time taken for tests:   21.336 seconds
Complete requests:      1000
Failed requests:        0
Total transferred:      9448000 bytes
HTML transferred:       8630000 bytes
Requests per second:    46.87 [#/sec] (mean)
Time per request:       213.356 [ms] (mean)
Time per request:       21.336 [ms] (mean, across all concurrent requests)
Transfer rate:          432.45 [Kbytes/sec] received
```

> 备注: 发现Rails的动态处理能力还是可以接受的，并发数为10时，能处理46.87个请求，性能还是可以的。

将Unicorn的工作进程数设置为2时，结果如下: 

```
# 远程服务器测试
Document Path:          /
Document Length:        8630 bytes

Concurrency Level:      10
Time taken for tests:   25.578 seconds
Complete requests:      1000
Failed requests:        141
   (Connect: 0, Receive: 0, Length: 141, Exceptions: 0)
Total transferred:      8951772 bytes
HTML transferred:       8136830 bytes
Requests per second:    39.10 [#/sec] (mean)
Time per request:       255.782 [ms] (mean)
Time per request:       25.578 [ms] (mean, across all concurrent requests)
Transfer rate:          341.77 [Kbytes/sec] received

# 本地测试
Concurrency Level:      10
Time taken for tests:   73.114 seconds
Complete requests:      1000
Failed requests:        0
Total transferred:      9448000 bytes
HTML transferred:       8630000 bytes
Requests per second:    13.68 [#/sec] (mean)
Time per request:       731.143 [ms] (mean)
Time per request:       73.114 [ms] (mean, across all concurrent requests)
Transfer rate:          126.19 [Kbytes/sec] received
```

看到服务器动态内容处理要比本地响应的请求慢很多，想到服务器本地测试可以作为Rails动态内容处理的速率上限。于是乎，
我对比了正式和测试服务器，发现正式服务器请求处理：14 req/s 对 11 req/s ，测试服务器上是 71.91 req/s 对 11.74 req/s， 
由于测试服务器是将mysql，mongodb、nginx以及应用服务器等都安装在一台机器上。

于是，猜想：**可能是由于不合理的分布式架构引起的性能的质降**。之后的事情就是要验证的这个猜想，这就需要做一些其他相关的事情。

> 注： 某日将数据库服务器分离后，测试服务器上的ab测试的结果为: 27.39 req/s 对11.09 req/s，果然是不合里的分布式数据库引起的
> 处理量的下降。或者说，服务器内网之间数据传送延迟较大。

**注**: unicorn一定要放在反向代理缓存之后。

### mobile项目本地的AB测试

Nginx + Passenger: `ab -n100 -c10 http://192.168.1.99:3000/`

外部测试结果: 

```
Document Path:          /
Document Length:        13247 bytes
Concurrency Level:      10
Time taken for tests:   9.780 seconds
Complete requests:      100
Failed requests:        47
   (Connect: 0, Receive: 0, Length: 47, Exceptions: 0)
Total transferred:      1382947 bytes
HTML transferred:       1324747 bytes
Requests per second:    10.22 [#/sec] (mean)
Time per request:       978.009 [ms] (mean)
Time per request:       97.801 [ms] (mean, across all concurrent requests)
Transfer rate:          138.09 [Kbytes/sec] received
```

server上结果: 

```
Document Path:          /
Document Length:        13248 bytes
Concurrency Level:      10
Time taken for tests:   6.240 seconds
Complete requests:      100
Failed requests:        52
   (Connect: 0, Receive: 0, Length: 52, Exceptions: 0)
Write errors:           0
Total transferred:      1213184 bytes
HTML transferred:       1156065 bytes
Requests per second:    16.02 [#/sec] (mean)
Time per request:       624.029 [ms] (mean)
Time per request:       62.403 [ms] (mean, across all concurrent requests)
Transfer rate:          189.86 [Kbytes/sec] received
```

Nginx + Unicorn: `ab -n100 -c10 http://192.168.1.99:5000/`

测试结果: 

```
Document Path:          /
Document Length:        9658 bytes
Concurrency Level:      10
Time taken for tests:   5.108 seconds
Complete requests:      100
Failed requests:        0
Total transferred:      1015000 bytes
HTML transferred:       965800 bytes
Requests per second:    19.58 [#/sec] (mean)
Time per request:       510.786 [ms] (mean)
Time per request:       51.079 [ms] (mean, across all concurrent requests)
Transfer rate:          194.06 [Kbytes/sec] received
```

server上结果: 

```
Document Path:          /
Document Length:        9658 bytes
Concurrency Level:      10
Time taken for tests:   4.712 seconds
Complete requests:      100
Failed requests:        0
Write errors:           0
Total transferred:      1015000 bytes
HTML transferred:       965800 bytes
Requests per second:    21.22 [#/sec] (mean)
Time per request:       471.185 [ms] (mean)
Time per request:       47.119 [ms] (mean, across all concurrent requests)
Transfer rate:          210.37 [Kbytes/sec] received
```

对比上述四组数据，从效果上来看，网络对passenger的影响更大(请求处理下降了6个)，而对unicorn的影响很小(请求处理下降了2个)，而且unicorn本身处理速度就比
passenger要快。

上述数据对比不太公平，unicorn的工作进程启动了两个，passenger的可能只有一个。不过，不太清楚passenger的并发工作机制。

正式服务器上Passenger的测试结果: `ab -n100 -c10 http://m.tophold.com/`

```
Document Path:          /
Document Length:        17415 bytes
Concurrency Level:      10
Time taken for tests:   12.692 seconds
Complete requests:      100
Failed requests:        53
   (Connect: 0, Receive: 0, Length: 53, Exceptions: 0)
Total transferred:      1799647 bytes
HTML transferred:       1741447 bytes
Requests per second:    7.88 [#/sec] (mean)  # 服务器上的测试是 7.25 [#/sec] (mean)
Time per request:       1269.243 [ms] (mean)
Time per request:       126.924 [ms] (mean, across all concurrent requests)
Transfer rate:          138.47 [Kbytes/sec] received
```

Unicron的测试结果:  `ab -n100 -c10 http://114.80.67.240:5000`

```
# 服务器本地测试
Document Path:          /
Document Length:        17415 bytes
Concurrency Level:      10
Time taken for tests:   7.294 seconds
Complete requests:      100
Failed requests:        42
   (Connect: 0, Receive: 0, Length: 42, Exceptions: 0)
Write errors:           0
Total transferred:      1792958 bytes
HTML transferred:       1741458 bytes
Requests per second:    13.71 [#/sec] (mean)
Time per request:       729.390 [ms] (mean)
Time per request:       72.939 [ms] (mean, across all concurrent requests)
Transfer rate:          240.05 [Kbytes/sec] received
# 本地计算机测试
Document Path:          /
Document Length:        17415 bytes
Concurrency Level:      10
Time taken for tests:   7.705 seconds
Complete requests:      100
Failed requests:        39
   (Connect: 0, Receive: 0, Length: 39, Exceptions: 0)
Total transferred:      1792961 bytes
HTML transferred:       1741461 bytes
Requests per second:    12.98 [#/sec] (mean)
Time per request:       770.541 [ms] (mean)
Time per request:       77.054 [ms] (mean, across all concurrent requests)
Transfer rate:          227.24 [Kbytes/sec] received
```

可以看到，网络对Unicorn的影响真的不是很大，请求处理下降的不大。此外，折算成每个实例的处理的请求数的话，Unicorn和Passenger应该差不多。

> 备注： ab测试的时候遇到了一个问题 `apr_sockaddr_info_get() for staging.api.tophold.com: Unknown error 14642 (14642)`， 解决方法是设置`ulimit -n 65535`或者在`/etc/security/limits.conf`文件中添加如下的行: 

```
root soft nofile 65535
root hard nofile 65535
* soft nofile 65535
* hard nofile 65535
```

最近的一组新的测试，让我越来越感到迷惑了，相同的命令`ab -n1000 -c50 http://staging.api.tophold.com/`在server和本地测试，server上32req/s, 本地居然52 req/s。

此外，发现了个现象，无论`-c`的数目多大，passenger都不会不响应请求，而unicron有可能不响应请求，unicron的并发数在100以下(具体为105), passenger的并发数大概在105左右，与具体项目无关。
从性能上来看，Unicorn似乎不及Passenger，果然是自己盲目跟风了，太过迷恋无缝重启之类的功能上了，感觉很不好。

## 部署测试

上面自己将代码拷贝到server上，自已运行自己启动的方法一点都不优雅，简直就是找罪受。今天，想来想去决定尝试部署。

首先，是部署脚本: 

```ruby
server "192.168.12.1", :app, :web, :db, :primary => true
set :stage, "production"
set :deploy_env, 'production'
set :rails_env, "production"
set :deploy_to, "/var/www/apps/mobile"
set :branch, 'master'

set :unicorn_config, "#{current_path}/config/unicorn/production.rb"
set :unicorn_pid, "#{current_path}/tmp/pids/unicorn.pid"

before "deploy:restart", "deploy:cleanup"

namespace :deploy do
  task :start, :roles => :app, :except => { :no_release => true } do
    run "cd #{current_path} && RAILS_ENV=#{deploy_env} bundle exec unicorn_rails -c #{unicorn_config} -D"
  end

  task :stop, :roles => :app, :except => { :no_release => true } do
    run "if [ -f #{unicorn_pid} ]; then kill -QUIT `cat #{unicorn_pid}`; fi"
  end

  task :restart, :roles => :app, :except => { :no_release => true } do
    # 用USR2信号来实现无缝部署重启
    run "if [ -f #{unicorn_pid} ]; then kill -s USR2 `cat #{unicorn_pid}`; fi"
  end

  task :custom_symlinks, :roles => :app do
    run "ln -nfs /var/www/apps/mobile/deploy/database.yml #{release_path}/config/database.yml"
    run "ln -nfs /var/www/apps/mobile/deploy/mongoid.yml #{release_path}/config/mongoid.yml"
    run "ln -s /home/share_101/uploads #{release_path}/public/"
    run "ln -s /home/share_101/public_images/uploads/th/public_images #{release_path}/public/"
  end
end
```

执行命令: 

```
mina setup    # 设置部署相关的路径 , capistrano 也有相应的命令
mina deploy   # 部署项目
```

以下是自己遇到的一些问题: 

问题: `invoke :'rails:assets_precompile'` 报错，找不到执行execjs的执行环境? 

解决: 

1.  注释掉，直接跳过。结果，blog看不到样式了。手动在项目目录下执行：`RAILS_ENV=production rake assets:precompile`
2.  在deploy.rb中添加如下的行: 
    
    ```
    task :environment do
      queue! %[source /root/.nvm/nvm.sh]  # 添加nvm的脚本执行环境，从而识别node
    end
    ```

> 为了确切的找到问题的答案，还特地去手动把文件和发布版本号修改了，结果证明，第二种方法果然可行。

一切尽在无缝重启中，响应速度非比寻常的快。

## GC

Ruby毕竟是动态语言，需要考虑GC。GC本身很耗时间，不能在处理请求时进行GC操作，OOD-GC，在请求处理结束后进行GC，从而，
提升性能和响应度。

## 争论

和前辈争论应用服务器无缝重启时，发现自己没有注意到，passenger也是热启动的，重启速度和unicorn不相上下。看来自己对Passenger的理解多少有点问题。

关于workers数的，passenger也可以配置，只不过需要在nginx中进行配置，然后需要重启nginx。

## 后记

原本，就不是特别熟悉unicorn等之类的web应用服务器，搞了一圈。Passegner就不熟悉，还去搞更不熟悉的unicorn，
判断失误了。 后来，还是搞起来了。

## 参考文献

1. [Nginx + Unicorn 部署 Rails 完整配置](http://blog.csdn.net/menxu_work/article/details/17077617)
2. [rails部署nginx+unicorn详解](http://www.mojidong.com/rails/2013/04/20/rails-nginx-unicorn/)
2. [Ruby-China.org 选择用 Thin 还是 Unicorn?](https://ruby-china.org/topics/35)
3. [使用Nginx + unicorn搭建ruby on rails的生产环境](http://my.oschina.net/mogralee/blog/299890)
4. [WBlog Ruby on Rails 项目布署指南](https://github.com/windy/wblog/wiki)
5. [Unicorn Rawk: Kick GC Out of the Band](http://blog.newrelic.com/2013/05/28/unicorn-rawk-kick-gc-out-of-the-band/)
6. [如何修改ulimit](http://zhidao.baidu.com/link?url=IeBUGnm5w8qD9lpYDidGF9_S4HNu2nos8caiU89OozV46KAW-wVt7puAIgkTfhXtepQXw84cL8HmIOvdcyz_j8-N0G8sugOb5qxNOXD11m3)
