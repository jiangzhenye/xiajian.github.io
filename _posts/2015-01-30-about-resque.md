---
layout: post
title: 关于resque
description: "ruby, resque"
---

## 前言

果然服务器端最先开始的是要搞后台任务，刚好又同redis扯上关系，这真是天赐良机啊，果断一起搞。

项目中的resque相关的gem包如下: 

* [resque](https://github.com/resque/resque), 很重要的，简介为：用于创建后台任务的基于Redis的Ruby库，其将任务放置到多个队列中，并用于随后的处理。
* [redis-namespace](https://github.com/resque/redis-namespace) 
* [resque-scheduler](https://github.com/resque/resque-scheduler), 任务安排相关。
* [rufus-scheduler](https://github.com/resque/resque-loner), ruby的任务安排, resque-scheduler的依赖项
* [resque-loner](https://github.com/resque/resque-loner), 支持在resque中的特殊任务，**已删除**
* [resque_mailer](https://github.com/zapnap/resque_mailer), 邮件任务
* [capistrano-resque](https://github.com/sshingler/capistrano-resque), 部署任务相关

> resque的依赖项: `mono_logger`, `multi_json` , `redis-namespace`(依赖redis) , sinatra , vegas(rack)

以下是对resque 2.x的官方文档的翻译。github中的readme是3.0的，所以，看起来区别比较大。

## resque

Resque是用来创建后台任务的基于Redis的库，其将任务放到多个队列中，并稍后处理。

后台任务是可以是任何响应`perform`方法的Ruby类或者模型。已存在的类可以很容易的转换成后台任务。
创建一个新类处理特定的job或者直接在已有的类中添加，都是可行的。

Resque受[DelayedJob](https://github.com/collectiveidea/delayed_job)启发，并由三个部件组成: 

1. 创建、查询和处理jobs的Ruby库
2. 启动worker进程，处理任务的Rake任务
3. 监视队列、jobs以及worker进程的Sinatra App

Resque的工作进程可以分布在不同的机器之间，支持优先级，弹性处理内存(bloat)/泄漏，针对
REE优化(即所谓的copy on write)，自行通知，以及失效期望。

Resque队列可持久化，支持固定时间的原子推送和弹出(感谢Redis)，提供内容的可视化，以及可将
jobs存储成简单的JSON包。

Resque前端展示工作进程做了什么，没做什么，那些在队列，那些在队列，提供通用的使用状态，并辅助追踪失效队列。

### Resque的背景

Github尝试了各种各样的后台任务: SQS, ActiveMessaging, BackgroundJob 

- SQS, 资源限制，存在时延，放弃
- ActiveMessaging, 太复杂，延续了Rails框架的设计
- BackgroundJob, 加载整个Rails环境很慢，耗时且CPU计算大
- DelayedJob, 类似BackgroundJob， 数据库支持的队列以及优先级。带有持久化(yaml持久化)的工作进程，
  仅在启动的加载Rails，在循环中处理jobs

最后，Github列出自己对后台任务的需求: 

- 持久化
- 观察等待的进程
- 即时修改等待进程
- tag
- 优先级
- 快速推送和弹出
- 观察完成的任务和正在做的任务
- 干掉过分臃肿的，陈旧的，运行时间过长的工作进程
- 保持Rails加载和持久化的进程。
- 分布式且可参考多个标签的工作进程
- 不重试不释放失效的进程

Redis的特征和特性: 

* Atomic, O(1) list push and pop
* Ability to paginate over lists without mutating them
* Queryable keyspace, high visibility
* Fast
* Easy to install - no dependencies
* Reliable Ruby client library
* Store arbitrary strings
* Support for integer counters
* Persistent - 持久化
* Master-slave replication - 主从复制
* Network aware - 网络感知

使用Redis处理队列问题，Resque关注工作进程的问题: 可见性，可靠性以及状态。

### 概述

在Resque中，创建jobs并将其放到队列中，然后，从队列中弹出jobs并处理。

Resque任务是响应`perform`方法的Ruby类或模块，如下是个简单的例子:

``` ruby
class Archive
  @queue = :file_serve

  def self.perform(repo_id, branch = 'master')
    repo = Repository.find(repo_id)
    repo.create_archive(branch)
  end
end
```

`@queue`类实例变量决定了`Archive`任务放置的队列。队列是任意的，且即刻创建。随即命名，任意数目。

为了将`Archive`任务放置到`file_serve`队列中，可将其添加到应用中已存在的`Repository`类中: 

``` ruby
class Repository
  def async_create_archive(branch)
    Resque.enqueue(Archive, self.id, branch) # 将任务插入队列
  end
end
```
> 原来，下载代码库的tarball是后台任务。此外，fork和create repo都是后台任务。

当在应用中调用`repo.async_create_archive('masterbrew')`时，就会创建一个job并将其放置到
`file_serve`队列中。

最后，将会启动一个worker进程，运行类似如下的代码处理: 

``` ruby
klass, args = Resque.reserve(:file_serve) # 从队列中提取任务
klass.perform(*args) if klass.respond_to? :perform
```

上述代码等价于:

``` ruby
Archive.perform(44, 'masterbrew')
```

启动一个工作进程，并运行的`file_serve`任务代码如下: 

    $ cd app_root
    $ QUEUE=file_serve rake resque:work  # 使用rake任务启动工作进程

上述代码启动了Resque worker进程，并告知其处理`file_serve`队列上的任务。一旦运行上述
`Resque.reserve`代码片段，就会一直处理到全部结束，休眠一段时间，重复的从队列中获取
更多的任务。

Workers可以给定多个队列(队列表)，并运行在多个机器上。实际，其可运行在Redis
服务器可访问的任何地方。

备注：rake任务启动后，没有任何显示，但是，可以从其他的一些方面获取反馈。比如，redis的key，和 resque-web的web界面程序（查看有无活跃的woker)

### 任务

什么应该运行在后台? 任何费时的事物，比如，缓慢的插入语句，磁盘操作，数据处理等等。

Github中使用Resque处理如下类型的任务: 

* Warming caches  - 预热缓存
* Counting disk usage - 统计磁盘使用率
* Building tarballs - 构建tarball球
* Building Rubygems - 构建Rubygems
* Firing off web hooks - 触发 web 钩子方法
* Creating events in the db and pre-caching them - 在数据库中创建事件，并进行预缓存
* Building graphs - 构建图形
* Deleting users - 删除用户
* Updating our search index - 更新搜索索引也用后台任务??

至今为止，Github中已有35个不同类型的后台任务。

注意：并不需要一个web app来运行Resque - 前台后台的区分仅仅是为了概念上的描述清晰。可以将爬站和粘数据
的处理放到队列中。

**Persistence**

任务可以持久化为JSON对象，以`Archive`为例，运行如下代码创建job，

``` ruby
repo = Repository.find(44)
repo.async_create_archive('masterbrew')
```

如下的JSON数据将会被存储在`file_serve`队列中: 

``` javascript
{
    'class': 'Archive',
    'args': [ 44, 'masterbrew' ]
}
```

Because of this your jobs must only accept arguments that can be JSON encoded.

由于Job必须要将接受的参数进行json编码，所以，不能使用`Resque.enqueue(Archive, self, branch)`，而应该
使用`Resque.enqueue(Archive, self.id, branch)`的调用方式。

这也是为何例子中使用对象id而不是传递整个对象的原因: 

使用对象ID，并重数据库或缓存中获取数据并非约定，但相比封装对象有一点优势：
封装对象存在使用带有过时信息的陈旧记录。

**send_later / async**

延后发送(DelayedJob支持该特性)在例子(`examples/`)中有所介绍，异步处理则在以后的发布会提供。

在后续的版本中，将提供一等的`async`支持。

**Failure**

如果job抛出异常，该异常将被记录到`Resque::Failure`模块。失效将被记录到本地，或者Redis，
或者其他后端。

For example, Resque ships with [Airbrake](https://github.com/airbrake/airbrake) support.

例如，Resque自带了[Airbrake](https://github.com/airbrake/airbrake)的支持。

谨记： 在编写任务时，可能想要抛出异常，从而辅助调试。

### Workers

Resque任务是永远运行的Rake任务，其基本结构如下: 

``` ruby
start
loop do # 就是一个无限循环
  if job = reserve
    job.process
  else
    sleep 5 # Polling frequency = 5
  end
end
shutdown
```

启动工作进程非常简单，具体例子如下: 

    $ QUEUE=file_serve rake resque:work

默认情况下，Resque并不知道应用的环境，这意味着其不能发现并运行jobs，它需要将整个应用加载到内存中。

若将Resque安装成Rails的插件，需要在RAILS_ROOT目录下运行如下命令，即可启动工作进程: 

    $ QUEUE=file_serve rake environment resque:work

上述脚本将会在启动工作进程时，加载环境。当然，可以定义一个依赖`environment`的
rake任务的`resque:setup`任务: 

``` ruby
task "resque:setup" => :environment
```

GitHub的启动任务看起来像下面这样:

``` ruby
task "resque:setup" => :environment do
  Grit::Git.git_timeout = 10.minutes
end
```

我们当然不希望在web app中执行10分钟一次的`git_timeout`任务，但是，在Resque工作进程中
这就是合理的。

**Logging**

工作进程支持记录到STDOUT的基本日志。如果在启动的时候，设置`VERBOSE`环境变量，将会在
输出中打印调试信息。当然，也可以设置`VVERBOSE`(极其详细)的环境变量。

    $ VVERBOSE=1 QUEUE=file_serve rake environment resque:work

**Process IDs (PIDs)**

记录resque的工作进程的PID，在某些情况下非常的有用。使用PIDFILE选项，可以很方便的访问
PID: 

    $ PIDFILE=./resque.pid QUEUE=file_serve rake environment resque:work

**后台运行**

(Only supported with ruby >= 1.9). There are scenarios where it's helpful for
the resque worker to run itself in the background (usually in combination with
PIDFILE).  Use the BACKGROUND option so that rake will return as soon as the
worker is started.

(仅支持ruby >= 1.9)。 某些情况下，让resque的工作进程在后台自己运行非常有用(通常，与
PIDFILE选项配套使用)。使用BACKGROUND选项，从而rake任务可以在worker进程启动时，尽快的返回。

    $ PIDFILE=./resque.pid BACKGROUND=yes QUEUE=file_serve \
        rake environment resque:work

**轮询频率(Polling frequency)**

You can pass an INTERVAL option which is a float representing the polling frequency.
The default is 5 seconds, but for a semi-active app you may want to use a smaller value.

可以设置INTERVAL选项，从而指定轮询频率的浮动表示。默认设置为5秒，但是，对于一个半活跃的
App，可以使用一个更小的值。

    $ INTERVAL=0.1 QUEUE=file_serve rake environment resque:work

**优先级和队列列表(Priorities and Queue Lists)**

Resque不支持数字优先级，但是可以使用给定的队列顺序来模拟。这里将队列的列表称之为
queue list"。

这里，添加一个新队列`warm_cache`，然后在启动工作进程，启动命令看似如下: 

    $ QUEUES=file_serve,warm_cache rake resque:work

当worker进程查找新jobs时，首先检查`file_serve`队列，找到一个就立即处理，然后
继续检查`file_serve`，直到`file_serve`无可用jobs后，worker进程才会检查`warm_cache`....

> 这里罗嗦了半天，就是`file_serve`队列总是优先`warm_cache`队列进行处理的。

> 结合`capistrano-resque`，在Capfile中设置worker中多个队列的方式如下: 

```
require "capistrano-resque"
role :resque_worker, "115.23.23.56"  # 设置worke所在的服务器
set :workers, { "mailer,async_mailer" => 1,
                "recommend_email_send,sitemap_refresh" => 1
              }

```

利用上述方法，我们可以优先某些队列。在Github中，以如下的方式启动工作进程:

    $ QUEUES=critical,archive,high,low rake resque:work

注意: `archive`队列是用作未来架构的特定的队列，并且将会在单独的机器上运行。

At that point we'll start workers on our generalized background
machines with this command:

随后，在通用的后台机器上使用如下的命令启动worker进程:

    $ QUEUES=critical,high,low rake resque:work

然而，在特定的archive的机器上启动worker的命令如下：

    $ QUEUE=archive rake resque:work

**Running All Queues**

如果，想要worker在每个队列上工作，包括那些即刻创建的，可以使用如下的命令:

    $ QUEUE=* rake resque:work

上述命令表明，队列将会以字母顺序进行处理。

**Running Multiple Workers**

Github使用god启动和停止多个工作进程。简单的god配置文件位于`examples/god`，强烈推荐
使用这中方法。

如果想要在开发者模式下运行多个工作进程，可以使用`resque:workers`rake任务: 

    $ COUNT=5 QUEUE=* rake resque:workers

上述命令将产生5个Resque工作进程，使用ctrl-c足以将它们全部暂停。

**Forking**

在某些平台中，当Resque工作进程在接受任务时，立即fork出一个子进程来处理job。当子进程
处理完job就退出，如果成功退出，工作进程接受另一个job并重复上述过程。

为何? 这是因为Resque假设chaos(混沌)。

Resque assumes your background workers will lock up, run too long, or
have unwanted memory growth.

Resque假设后台工作进程可能会死锁，运行太久，或意外的内存增长。

If Resque workers processed jobs themselves, it'd be hard to whip them
into shape. Let's say one is using too much memory: you send it a
signal that says "shutdown after you finish processing the current
job," and it does so. It then starts up again - loading your entire
application environment. This adds useless CPU cycles and causes a
delay in queue processing.

如果resque 的worker进程自己处理

Plus, what if it's using too much memory and has stopped responding to
signals?

另外，

Thanks to Resque's parent / child architecture, jobs that use too much memory
release that memory upon completion. No unwanted growth.

And what if a job is running too long? You'd need to `kill -9` it then
start the worker again. With Resque's parent / child architecture you
can tell the parent to forcefully kill the child then immediately
start processing more jobs. No startup delay or wasted cycles.

The parent / child architecture helps us keep tabs on what workers are
doing, too. By eliminating the need to `kill -9` workers we can have
parents remove themselves from the global listing of workers. If we
just ruthlessly killed workers, we'd need a separate watchdog process
to add and remove them to the global listing - which becomes
complicated.

Workers instead handle their own state.

**Parents and Children**

如下是父子进程对处理相同的任务: 

    $ ps -e -o pid,command | grep [r]esque
    92099 resque: Forked 92102 at 1253142769
    92102 resque: Processing file_serve since 1253142769

可以清楚的看到，92099 fork 92102，并从1253142769开始处理。

可以使用monit或god 来杀死陈旧的进程。

当父进程空闲下来，它将通知你他在等待那个队列: 

    $ ps -e -o pid,command | grep [r]esque
    92099 resque: Waiting for file_serve,warm_cache

**Signals**

Resque工作进程响应不同的信号：

* `QUIT` - Wait for child to finish processing then exit
* `TERM` / `INT` - Immediately kill child then exit
* `USR1` - Immediately kill child but don't exit
* `USR2` - Don't start to process any new jobs
* `CONT` - Start to process new jobs again after a USR2

优雅的干掉Resque工作进程，使用`QUIT`。

如果，想要杀死一个陈旧的或卡顿的进程，使用`USR1`。除非找不到子进程，进程将持续处理。
在此情况下，Resque假设父进程处于坏的状态下，并将其关闭。

如果，想要杀死陈旧的或卡顿的进程，并将其关闭，使用`TERM`。

如果想要停止处理jobs，但是又要保持worker持续运行，例如，为了临时的缓解负载，可以使用`USR2`来停止进程，
然后使用`CONT`来启动进程。

**Mysql::Error: MySQL server has gone away**

如果你的工作进程闲置了太长时间，则可能是MySQL链接断开。如果发生这种情况，推荐使用
[这个gist](http://gist.github.com/238999)

### 前端

Resque自带了一个基于Sinatra的前端，用来查看队列的状态。其可单独使用，也可与其他的东西进行集成。

**单独使用**

安装Resque之后，就会自动安装`resque-web`gem包，运行命令介绍如下: 

    $ resque-web
    $ resque-web -p 8282  # sinatra app是`rackup`薄的包装层，可以进行一定的配置
    $ resque-web -p 8282 rails_root/config/initializers/resque.rb # 可将配置文件作为参数执行
    $ resque-web -p 8282 -N myapp  # 可以通过`-N`直接指定命名空间
    $ resque-web -p 8282 -r localhost:6379:2  # -r选项设置连接字符串

**与Passenger配合**

使用Passenger? Resque自带了一个`config.ru`，更多的参考如下的Phusion指南: 

Apache: <http://www.modrails.com/documentation/Users%20guide%20Apache.html#_deploying_a_rack_based_ruby_application>
Nginx: <http://www.modrails.com/documentation/Users%20guide%20Nginx.html#deploying_a_rack_app>

**Rack::URLMap**

如果想在子路径中加载Resque，比如使用其他的app，这可以简单的通过Rack的`URLMap`进行处理: 

```ruby
require 'resque/server'

run Rack::URLMap.new \
  "/"       => Your::App.new,
  "/resque" => Resque::Server.new
```

Check `examples/demo/config.ru` for a functional example (including HTTP basic auth).

查看`examples/demo/config.ru`提供的功能性的例子(包含HTTP的基本权限验证)。

**与Rails 3集成**

You can also mount Resque on a subpath in your existing Rails 3 app by adding `require 'resque/server'` to the top of your routes file or in an initializer then adding this to `routes.rb`:

通过在路由文件的顶端或初始化脚本中添加`require 'resque/server'`，可将Resque挂载到已存在的Rails 3 的app中: 

``` ruby
mount Resque::Server.new, :at => "/resque"
```

### Resque vs DelayedJob

How does Resque compare to DelayedJob, and why would you choose one
over the other?

如何对比Resque和DelayedJob，为何选择这个而不选择另一个？

* Resque supports multiple queues
* DelayedJob supports finer grained priorities(细粒度的优先级)
* Resque workers are resilient to memory leaks / bloat (弹性的内存使用)
* DelayedJob workers are extremely simple and easy to modify
* Resque requires Redis
* DelayedJob requires ActiveRecord
* Resque can only place JSONable Ruby objects on a queue as arguments(只能使用可JSON化的Ruby对象作为参数)
* DelayedJob can place _any_ Ruby object on its queue as arguments(可以使用任何Ruby对象)
* Resque includes a Sinatra app for monitoring what's going on(包含单独的Sinatra app，从而用作监控)
* DelayedJob can be queried from within your Rails app if you want to
  add an interface

If you're doing Rails development, you already have a database and
ActiveRecord. DelayedJob is super easy to setup and works great.
GitHub used it for many months to process almost 200 million jobs.

Choose Resque if:

* You need multiple queues
* You don't care / dislike numeric priorities
* You don't need to persist every Ruby object ever
* You have potentially huge queues
* You want to see what's going on
* You expect a lot of failure / chaos
* You can setup Redis
* You're not running short on RAM

Choose DelayedJob if:

* You like numeric priorities
* You're not doing a gigantic amount of jobs each day
* Your queue stays small and nimble
* There is not a lot failure / chaos
* You want to easily throw anything on the queue
* You don't want to setup Redis

In no way is Resque a "better" DelayedJob, so make sure you pick the
tool that's best for your app.

并不是说Resque比DelayedJob更好，所以，确保选择最适合的自己的APP的。

### Resque的安装和依赖

    $ gem install bundler
    $ bundle install

Resque本身依赖`redis-namespace`，而`redis-namespace`本身依赖`redis-rb`。此外，发现gem包的依赖的
一些特别的情况，`Gemfile`和`resque.gemspec`中不同的依赖描述的关系。

**Resque的安装**

首先安装gem，然后在应用中包含，最后启动应用，然后，就可以在应用中创建Resque任务了。

```
$ gem install resque
require 'resque'
$ rackup config.ru
```

在根目录下创建Rakefile(或直接在已有的)中，创建任务: 

``` ruby
require 'your/app'
require 'resque/tasks' # 这里的意思是，加载`resque/tasks`下的所有任务
```

然后，运行如下的命令：

    $ QUEUE=* rake resque:work

可选的，可以在Rakefile中定义`resque:setup`钩子方法，从而避免在每次加载应用时运行rake。

在Rails 3中，将Resque作为Gem安装的步骤: 

1. 在Gemfile 加入`resque`的gem包: `gem 'resque'`
2. 然后，安装bundler: `$ bundle install`
3. 启动应用程序: `$ rails server`

然后，就可以在应用程序中创建Resque任务。通过在`lib/tasks`目录下，添加文件(比如: `lib/tasks/resque.rake`)
从而启动工作进程。

然后，就可以在项目中创建Resque任务了。为了启动一个worker进程，则需要在`lib/tasks`下添加相应的文件，比如:
`lib/tasks/resque.rake`，其内容如下:

``` ruby
require 'resque/tasks'
```

现在，使用如下命令启动任务: 

    $ QUEUE=* rake environment resque:work

不要忘记在`lib/tasks/whatever.rake`中定义`resque:setup`钩子，从而每次加载
`environment`任务。

### 配置

可能需要修改Resque连接的Redis的主机和端口，或者在启动时设置其他的选项。这意味着，如果已经在App中使用了redis，
Resque将重用已有的链接。以下是配置的两种形式: 

* 字符串形式: `Resque.redis = 'localhost:6379'`
* Redis形式: `Resque.redis = $redis`

在Rails中，可以在`config/initializers/resque.rb`的初始化文件中手动加载`config/resque.yml`，然后，恰当的设置Redis
信息。

下面是`config/resque.yml`的样例: 

    development: localhost:6379
    test: localhost:6379
    staging: redis1.se.github.com:6379
    fi: localhost:6379 
    production: redis1.ae.github.com:6379

初始化文件为: 

``` ruby
rails_root = ENV['RAILS_ROOT'] || File.dirname(__FILE__) + '/../..'
rails_env = ENV['RAILS_ENV'] || 'development'

resque_config = YAML.load_file(rails_root + '/config/resque.yml')
Resque.redis = resque_config[rails_env]
```

可以看到，相当的简单，但是为何不直接使用`RAILS_ROOT`和`RAILS_ENV`？ 这是因为，想要直接告诉Sinatra app
配置文件的路径: 

    $ RAILS_ENV=production resque-web rails_root/config/initializers/resque.rb

现在，所有程序都在一个页面。

此外，可以通过设置'inline'属性来禁用任务查询。例如，想要在cucumber的相同进程中运行所有的任务，可以尝试: 

``` ruby
Resque.inline = ENV['RAILS_ENV'] == "cucumber"
```

### 插件和钩子

可用的插件列表参考: <http://wiki.github.com/resque/resque/plugins>

如果想要编写自己的插件，或者想使用钩子定制化Resque，可以参考[docs/HOOKS.md](http://github.com/resque/resque/blob/master/docs/HOOKS.md)

### Namespaces

如果运行了多个，单独的Resque实例，可能需要使用命名空间来隔离键空间，从而避免彼此覆盖。
这种方法与memcached客户端实现的有所不同。

该特性由[redis-namespace][rs]库提供， Resque默认使用该库在Redis服务器中隔离键。

`Resque.redis.namespace`的访问器简单使用如下: 

``` ruby
Resque.redis.namespace = "resque:GitHub"
```

推荐在初始化Redis配置中的某处，设置命令空间配置。

### Demo

Resque自带了一个Sinatra的demo app，从而用来创建随后在后台处理的任务。

更多可以查看`examples/demo/README.markdown`文件。

### 监控(Monitoring)

可以使用god或者monit来监控Resque的worker进程。

**god**： 如果使用god，可以参考位于`examples/god/`的例子配置文件。其中一个是用来启动和停止worker进程，
另一个是用来杀死运行时间过长的工作进程。

**monit**：如果使用monit，项目同样提供配置文件:`examples/monit/resque.monit`。该配置没有在Github的生产
环境中使用。所以，欢迎发送patch提高配置性能。


### 问题(Questions)

遇到问题，可以将其添加[FAQ](https://github.com/resque/resque/wiki/FAQ)中，或直接在Mail List发问。

### Development

想要探索Resque？

首先，克隆代码库，然后运行测试: 

    git clone git://github.com/resque/resque.git
    cd resque
    rake test

如果测试没有通过，先检查是否在本地安装了Redis。测试尝试启动一个隔离的Redis实例来运行测试。

此外，还要确认是否已经正确的安装了所有的依赖。例如，如下代码将简单尝试是否已经安装了`redis-namespace`。

    $ irb
    >> require 'rubygems'
    => true
    >> require 'redis/namespace'
    => true

如果，在加载依赖时出错了，可能会是安装失败或者看看是不是加载路径的问题。

出了问题，可以参考邮件列表。

### Contributing

Read the [Contributing][cb] wiki page first.

想要贡献代码，首先参看[Contributing][cb] 的wiki页面，然后，作出自己的提交: 

1. [Fork][1] Resque
2. Create a topic branch - `git checkout -b my_branch`
3. Push to your branch - `git push origin my_branch`
4. Create a [Pull Request](http://help.github.com/pull-requests/) from your branch
5. That's it!

### Mailing List

发送邮件到<resque@librelist.com>，即可加入到邮件列表中。其中包含订阅和非订阅的信息。

邮件归档可在<http://librelist.com/browser/resque/>看到。

### Meta

* Code: `git clone git://github.com/resque/resque.git`
* Home: <http://github.com/resque/resque>
* Docs: <http://rubydoc.info/gems/resque>
* Bugs: <http://github.com/resque/resque/issues>
* List: <resque@librelist.com>
* Chat: <irc://irc.freenode.net/resque>
* Gems: <http://gemcutter.org/gems/resque>

This project uses [Semantic Versioning][sv].

### Author

Chris Wanstrath :: chris@ozmm.org :: @defunkt

[0]: http://github.com/blog/542-introducing-resque
[1]: http://help.github.com/forking/
[2]: http://github.com/resque/resque/issues
[sv]: http://semver.org/
[rs]: http://github.com/resque/redis-namespace
[cb]: http://wiki.github.com/resque/resque/contributing

## 后记

本该周末(1月31左右)学习的东西，结果，周末禁不住的看了两天动漫，感觉很不好。我的自控力果然是一坨烂泥。

现有网站的实践是这样的: 

* gem包: resque, resque-scheduler, rufus-scheduler, resque_mailer
* 启动脚本: `config/initializers/load_resque.rb`
* Jobs实现的类: `app/jobs/*.rb`, 即响应`perform`方法的类。
* 任务安排文件: `config/resque_schedule.yml`，以及配置文件

后台任务的处理，是个很典型的生产者消费者模型，通过调用`Resque.enqueue`将任务存储到Redis中，worker进程通过读取
`resque_schedule.yml`来执行任务，worker本身是通过在部署时启动的。生产者和消费者是彼此分离的。
