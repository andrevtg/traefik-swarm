traefik-swarm
=====

Exemplo de Traefik como LB de swarm mode.

1. Rodar traefik

```sh
docker network create --driver=overlay traefik-net
docker service create \
    --name traefik \
    --constraint=node.role==manager \
    --publish 8000:80 --publish 8080:8080 \
    --mount type=bind,source=/var/run/docker.sock,target=/var/run/docker.sock \
    --network traefik-net \
    traefik \
    --docker \
    --docker.swarmMode \
    --docker.domain=traefik \
    --docker.watch \
    --api
```

Na porta 8000 do host estará o traefik (erro 404 pois ainda não há app nenhuma na retaguarda) e na 8080 o dashboard.

2. Rodar apps na retaguarda

```sh
# app 0
docker service create \
    --name whoami0 \
    --label traefik.port=80 \
    --network traefik-net \
    containous/whoami
# app 1
docker service create \
    --name whoami0 \
    --label traefik.port=80 \
    --network traefik-net \
    containous/whoami
# testando app 0
curl -H Host:whoami0.traefik localhost:8000
# testando app 1
curl -H Host:whoami1.traefik localhost:8000
```

3. Testando regras de host e path (mesmo hostname, paths distintos)

```sh
docker service update \
    --label-add "traefik.frontend.rule=Host:whoami.aqui.com;PathPrefixStrip:/app0" \
    whoami0
docker service update \
    --label-add "traefik.frontend.rule=Host:whoami.aqui.com;PathPrefixStrip:/app1" \
    whoami1
curl -H Host:whoami.aqui.com localhost:8000/app0/
curl -H Host:whoami.aqui.com localhost:8000/app1/
```


