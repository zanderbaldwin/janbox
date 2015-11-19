# The Janbox

Isolated PHP server environment.

## Linked Containers

The Janbox links to other containers to provide MySQL and Redis servers.

```bash
# Root password should be memorable because the MySQL server will only be available to linked containers.
$ docker run -d --restart=always --name=janmysql -e "MYSQL_ROOT_PASSWORD=janmysql" mysql:latest
$ docker run -d --restart=always --name=janredis redis:latest
```

## Building

Super simple!

```bash
docker build -t janbox .
```

## Running

```bash
docker run -d --env TERM=xterm --restart=always --name=janbox --link=janmysql --link=janredis -p 32526:80 janbox
```

Then add the following Nginx site configuration:

```nginx
server {
    listen       80;
    server_name  $DOMAIN;

    access_log   /var/log/nginx/$DOMAIN.access.log;
    error_log    /var/log/nginx/$DOMAIN.error.log;

    location / {
        proxy_pass           http://localhost:32526;
        proxy_next_upstream  error timeout invalid_header http_500 http_502 http_503 http_504;
        proxy_redirect       off;
        proxy_buffering      off;
        proxy_set_header     Host            $host;
        proxy_set_header     X-Real-IP       $remote_addr;
        proxy_set_header     X-Forwarded-For $proxy_add_x_forwarded_for;
    }
}
```

## SSH Access

Instead of adding a public key to `authorized_keys` as normal, add the following line (on the host machine):

```
command="docker exec -it -u www-data janbox bash -l",no-port-forwarding,no-X11-forwarding,no-agent-forwarding <ssh public key "ssh-rsa .... user@email">
```
