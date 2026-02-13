# ComfyUI образ

## Референсы

- [AI on-demand with Kubernetes (Part 2)](https://snowgoons.ro/posts/2024-07-03-ai-on-demand-with-kubernetes--part-2-/) - тут чел показывает пример Docker образа, который он потом в кубе запускает. Я вдохновился этим образом, но со временем все же существенного его изменил
- [How to install ComfyUI manually in different systems](https://docs.comfy.org/installation/manual_install) - официальная дока по установке ядра ComfyUI

## Важные моменты

Чтобы в образе была конкретная и актуальная версия CPython, я собираю его бинарь прям в образе с помощь multi-stage сборки. Важно то, что для сборки используется тот же самый образ, который потом будет использоваться в рантайме (nvidia/cuda:xxx), благодаря этому я гарантирую бинарную и библиотечную совместимость, чтобы динамически слинкованный на первом этапе сборки CPython нашел все необходимые ему либы в рантайме на втором этапе сборки

Компиляция питона выполняется в соответствии с официальной докой https://docs.python.org/3.13/using/unix.html

В моем Dockerfile нет явного указания `ENTRYPOINT` или `CMD`, я решил, что лучше буду указывать это явно в спеке пода. В таком случае не придется играть в угадайку и вспоминать, что там уже было указано

## Сборка образа

```sh
# COMMIT_SHA=$(date +'%s')

# cuda 12
COMMIT_SHA='cuda12-1771003088'
docker buildx build \
  --file dockerfiles/Dockerfile-cuda12 \
  --tag "78945789345654/comfyui:$COMMIT_SHA" \
  .
docker push 78945789345654/comfyui:$COMMIT_SHA

# cuda 13
COMMIT_SHA='cuda13-1770807898'
docker buildx build \
  --file dockerfiles/Dockerfile-cuda13 \
  --tag "78945789345654/comfyui:$COMMIT_SHA" \
  .
docker push 78945789345654/comfyui:$COMMIT_SHA
```

## Smoke проверка собранного образа:

```sh
COMMIT_SHA='cuda12-1771003088'
docker run -it --rm --entrypoint python 78945789345654/comfyui:$COMMIT_SHA main.py --help

COMMIT_SHA='cuda13-1770807898'
docker run -it --rm --entrypoint python 78945789345654/comfyui:$COMMIT_SHA main.py --help
```
