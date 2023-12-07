# docker-compose-laravel
一个相当简化的 Docker Compose 工作流程

## Usage

首先，请确保您的系统上 [安装了 Docker](https://docs.docker.com/docker-for-mac/install/) , 然后克隆此存储库。

接下来，在终端中导航到您克隆的目录，并通过运行启动 Web 服务器的容器 `docker-compose up -d --build app`.

完成后，按照 [src/README.md](src/README.md) 文件中的步骤添加 Laravel 项目（或创建一个新的空白项目）。

**Note**: 您的 MySQL 数据库主机名应该是mysql，而不是 localhost。homestead用户名和数据库的密码都应为secret。

Bringing up the Docker Compose network with `app` instead of just using `up`, ensures that only our site's containers are brought up at the start, instead of all of the command containers as well. The following are built for our web server, with their exposed ports detailed:

- **nginx** - `:80`
- **mysql** - `:3306`
- **php** - `:9000`
- **redis** - `:6379`
- **mailhog** - `:8025` 

其中包含三个附加容器，可处理 Composer、NPM 和 Artisan 命令，而无需在本地计算机上安装这些平台。使用项目根目录中的以下命令示例，修改它们以适合您的特定用例。

- `docker-compose run --rm composer update`
- `docker-compose run --rm npm run dev`
- `docker-compose run --rm artisan migrate`

## 权限问题

如果您在访问应用程序或运行容器命令时遇到文件系统权限的任何问题，请尝试完成以下一组步骤。

**如果您以 root 用户身份使用服务器或本地环境:**

- Bring any container(s) down with `docker-compose down`
- Replace any instance of `php.dockerfile` in the docker-compose.yml file with `php.root.dockerfile`
- Re-build the containers by running `docker-compose build --no-cache`

**如果您以非 root 用户身份使用服务器或本地环境:**

- Bring any container(s) down with `docker-compose down`
- In your terminal, run `export UID=$(id -u)` and then `export GID=$(id -g)`
- If you see any errors about readonly variables from the above step, you can ignore them and continue
- Re-build the containers by running `docker-compose build --no-cache`

Then, either bring back up your container network or re-run the command you were trying before, and see if that fixes it.

## MySQL 持久存储

默认情况下，每当您关闭 Docker 网络时，您的 MySQL 数据都会在容器被销毁后被删除。如果您希望在关闭和备份容器后保留保留的持久数据，请执行以下操作：

1. Create a `mysql` folder in the project root, alongside the `nginx` and `src` folders.
2. Under the mysql service in your `docker-compose.yml` file, add the following lines:

```
volumes:
  - ./mysql:/var/lib/mysql
```

## 生产中的使用

虽然我最初创建此模板是为了本地开发，但它足够强大，可以在基本的 Laravel 应用程序部署中使用。 The biggest recommendation would be to ensure that HTTPS is enabled by making additions to the `nginx/default.conf` file and utilizing something like [Let's Encrypt](https://hub.docker.com/r/linuxserver/letsencrypt) to produce an SSL certificate.

## 编译资产

This configuration should be able to compile assets with both [laravel mix](https://laravel-mix.com/) and [vite](https://vitejs.dev/). In order to get started, you first need to add ` --host 0.0.0.0` after the end of your relevant dev command in `package.json`. 例如，对于使用 Vite 的 Laravel 项目，您应该看到:

```json
"scripts": {
  "dev": "vite --host 0.0.0.0",
  "build": "vite build"
},
```

然后，运行以下命令来安装依赖项并启动开发服务器：

- `docker-compose run --rm npm install`
- `docker-compose run --rm --service-ports npm run dev`

之后，您应该能够使用@vite指令在本地 Laravel 应用程序上启用热模块重新加载。

想要为生产而构建吗？只需运行即可 `docker-compose run --rm npm run build`.

## MailHog

The current version of Laravel (9 as of today) uses MailHog as the default application for testing email sending and general SMTP work during local development. Using the provided Docker Hub image, getting an instance set up and ready is simple and straight-forward. The service is included in the `docker-compose.yml` file, and spins up alongside the webserver and database services.

To see the dashboard and view any emails coming through the system, visit [localhost:8025](http://localhost:8025) after running `docker-compose up -d site`.


# 系统架构
laravel-modules
laravel-modules 是如何自动注册模块的服务的？
生成模块时在配置文件src/modules_statuses.json中注册模块名称
然后在下面的文件中使用模块名使用 laravel ioc 注入模块服务
src/vendor/nwidart/laravel-modules/src/LaravelModulesServiceProvider.php