# nginx-load-balancer
🐳 Nginx docker containers load balancer and reverse proxy


## 🛳 Acesso aos containers
```sh
# subindo a instância do container
docker-compose up -d

# instalando o bash, não presente na imagem alpine
docker-compose exec << nginx | node_1 | node_2 >> apk add bash
```

```sh
# entrando no nginx
docker-compose exec << nginx | node_1 | node_2 >> bash
```

```sh
# adicionando o vim
apk add vim
```

```sh
# acessando o arquivo de configurações
vim /etc/nginx/nginx.conf
vim /etc/nginx/conf.d/default.conf
```

```sh
# configurando e testando o arquivo de configurações, após editar
nginx -t
```

```sh
# editando o html do node_1
vim /usr/share/nginx/html/index.html
```

```sh
# re-startando o nginx
nginx -s reload
```

```sh
# acessando os logs de acesso do nginx
tail -f /var/log/nginx/access.log
# ou
tail -f /var/log/nginx/nginx-access-log
```

## 🏗 Máquina principal (proxy reverso)
```sh
# /etc/nginx/conf.d/default.conf
# conjunto de máquinas que recebe um nome (upstream)
upstream nodes {
    server node_1;
    server node_2;
}

server {
    listen       80;
    server_name  localhost;

    location / {
        # será redirecionado para uma das maquinas do upstream :D
        proxy_pass http://nodes;

        # capturando, para registrar em log, o ip do usuário que acessou o servidor
        proxy_set_header X-Real-IP $remote_addr;
    }

    # dividindo os acessos, de rotas específicas, entre máquinas/containers
    location /node_1 {
        # quando a rota '/node_1' for acessada, redireciona para o node 1
        proxy_pass http://node_1;
    }

    location /node_2 {
        # quando a rota '/node_2' for acessada, redireciona para o node 2
        proxy_pass http://node_2;
    }

    # registro de logs
    acess_logs /var/log/nginx/nginx-access-log;
    # ou, com uma formatação
    acess_logs /var/log/nginx/nginx-access-log main;
}
```

## 🐳 Node 1
Um proxy reverso em relação a ele

```sh
# o ip real deve ser definido nos registros do log do nodes (/etc/nginx/nginx.conf)
log_format main '$http_x_real_ip - ...'
```

## 🐳 Node 2
Um proxy reverso em relação a ele
```sh
# o ip real deve ser definido nos registros do log do nodes (/etc/nginx/nginx.conf)
log_format main '$http_x_real_ip - ...'
```

## 🗺 Fluxo do load balancer
```sh
# cliente -> rota '/' -> nginx (proxy reverso) -> node_1 | node_2
```