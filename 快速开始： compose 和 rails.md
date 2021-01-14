快速开始： compose 和 rails

这个快速上手指南告诉你怎么用docker compose去设置和运行一个

rails/postgresql 应用，在开始前，请安装 docker 和 docker compose

 

定义这个项目

 

我们从设置构建这个应用需要的文件开始，这个应用会在一个包含自己依赖的docker容器中运行

。定义这些依赖是用一个叫dockerfile的文件来完成的。现在开始，这个dockefile

由这些代码段组成。

 

这些代码段将把你应用的代码放进一个image，这个image创建了一个包含ruby，bundler和 所有依赖的容器。想了解更多如何写dockerfile，去看Docker user guide 和 Dockerfile reference。

下面，创造一个独立的Gemfile，他只是用来加载rails，他会被重新覆写但运行rails new 的时候。

 

创建一个空的Gemfile.lock去创建我们的Dockerfile

 

然后，提供一个entrypoint脚本去处理rails特有的问题，这个问题会保护服务器重启当一个确定的server.pid 文件曾经存在过。当每次容器启动时这个脚本会被执行。Entrypoint.sh 包含：

 

最后，docker-compose.yml 是魔法发生的地方。这个文件描述了包含你app（一个数据库和web app）的服务，怎么去得到他的docker image（数据库只是在一个预先做好的postgresql image上运行，web app是在当前目录创建的），配置文件需要把他们连接起来并暴露 web app的端口。

 

 

构建项目

 

当这些文件到位了，你可以用docker-compose run构建一个rails的骨架app

首先，compose 用dockerfile为web服务构建这个镜像。--no-deps告诉compose不要去

启动关联的服务。然后在一个新的容器里用这个镜像运行 rails new，一旦他完成，你应该由了一个新的app。

列出这些文件。

 

 

如果你是在linux上运行docker，rails new创建的文件是被root拥有的，这会发生是因为这个容器时已root用户运行的。如果是这种情况，改变新文件的所有权。

 

如果你是在mac或者windows上运行docker，你因该已经有了这些文件的所有权，包括被rails new 命令创建的文件。

现在你有了一个新的Gemfile，你需要再次构造这个镜像，（这种行为，Gemfile和Dockerfile的改动，应该是你唯一需要重新构建的时候）

 

连接到数据库

这个app现在是可以启动的，但是还没有到那里。默认的，Rails希望有一个数据库运行在

Localhost – 所以你需要在db容器里指出他。你也需要该改变数据库和用户名去和postgres镜像的默认设置保持一致。

用以下内容替代config/database.yml的内容

 

你可以用docker-compose up启动这个app

如果一切都好，你应该看到一些postgresql输出。

 

最后，你需要创建数据库，在另一个终端，运行：

 

这是一个命令行输出的例子：

 

浏览rails的欢迎页面

这就是了，你的app应该在你的docker服务里运行在3000端口。

在mac或Windows的docker桌面版，直接用浏览器访问http://localhost:3000去看到rails的欢迎页面。

 

停止app

为了停止app，在项目的目录下运行docker-compose down。你可以用你启动数据库的终端窗口，或则另一个你有权限运行命令的窗口，这是一个清爽的停止应用的方法。

重启应用

在项目目录运行docker-compose up 去重启应用。

 

重新构建应用

如果你对Gemfile或则composefile做了一些不同的配置，你需要重新构建他。一些改变只需要docker-compose up –build， 但是一个完全的重构需要重新运行docker-compose run web bundle install 去同步Gemfile.lock里的改变，在docker-compose up –build 之后。

 

这是第一种情况的一个例子，这里一个完全的重构不是必要的，假设你只是简单的想要改变暴露的端口在本机的3000改为3001.在compose file 做出改变暴露3000端口到一个新端口，3001，在客户机，保存改变。

 

现在重构和重启app用 docker-compose up –build

 

在容器内部，你的app正在一样3000端口运行，但是rails的欢迎界面已经可以在http://localhost:3001上访问了。