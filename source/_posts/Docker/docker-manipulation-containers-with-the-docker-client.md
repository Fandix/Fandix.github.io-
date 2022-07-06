---
title: Docker - Manipulation Containers with the Docker Client
date: 2022-07-06 14:35:55
tags:
- Back-end
- Docker
- CI/CI

categories:
- Docker
---

上一章介紹完什麼是 docker 與為什麼需要使用 docker 後，本章將介紹 docker 常用的 CLI 指令。
<img src="https://nickjanetakis.com/assets/blog/cards/understanding-how-the-docker-daemon-and-docker-cli-work-together-470c6acb68e0705a6160adb85a53bbc37a620fc07b0dc8d214bff6ee6a1d6154.jpg" alt="drawing" width="900"/>

<!-- more -->

# Docker Run
使用 `docker run <image name>` 可以創建和執行指定 Image 所創建的 Container。

```docker
docker run hello-world 
```
當輸入 `docker run hello-world` 後，會將 `hello-world` 的 Image 創建一個 `Container` 並將要執行 hello-world 所需要的內容放進這個 `Container` 中。
<img src="https://ithelp.ithome.com.tw/upload/images/20220706/20124767eTbWMymPSM.png" alt="drawing" width="860"/>

---

# Overriding default commands

可以利用 `docker run <image name> command` 將我們希望執行的 command 放在後頭，這樣當 docker 利用指定的 Image 創建 Container 後就會執行指定的 Command，因為我們有指定要執行的 Command，所以當創建這個 Container 之後便 `不會執行這個 Container 預設要執行的所有動作`。

```docker
docker run busybox echo hi there
## hi there

docker run busybox ls
## dev
## etc
## home
## proc
## root
## sys
## tmp
## usr
## var
```

就跟上面執行 `hello-world` 時一樣，當使用 `run` 指令利用 Image 創建一個 Container 時，會將 `busybox` 所需要的東西放進 Container 中，所以當執行了 `ls` 時就可以看到這些檔案。
<img src="https://ithelp.ithome.com.tw/upload/images/20220706/20124767AV1EalAWVA.png" alt="drawing" width="860"/>

因為只有 `busybox` 這個 Image 才會有執行 `ehco`, `ls` 的依賴，所以如果要使用 `hello-world` 來執行 `ehco` 或 `ls` 的話就會出錯，因為 hello-world 的 Image 中並沒有可以執行這兩個指令的依賴。

```docker
docker run hello-world ls
## docker: Error response from daemon: failed to create shim: OCI runtime create failed: container_linux.go:380: starting container process caused: exec: "ls": executable file not found in $PATH: unknown.
## ERRO[0000] error waiting for container: context canceled
```

---

# Listing Running Containers
可以使用 `docker ps` 查看到所有正在執行的 Container

```docker
docker ps
## CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

透過 `ps` command 可以看到目前正在執行的 Container 的 `id`, `name`, `image`, `command` ...，因為目前沒有任何 Container 正在執行所以這個列表是空的，可以跑一個 Container 看看會發生什麼事

```docker
docker run busybox ping google.com
## hi there

docker ps
## CONTAINER ID   IMAGE     COMMAND             CREATED          STATUS          PORTS     NAMES
## b4641bd43fd2   busybox   "ping google.com"   13 seconds ago   Up 12 seconds             festive_kirch
```

這樣就可以看到目前正在執行的 Container，id 是 `b4641bd43fd2` 然後是透過 `busybox` 這個 Image 創建的 Container 等等的資訊。

當我們利用 `control C` 停止 `ping google.com` 之後再利用 `ps` 就可以看到他已經停下來了

```docker
docker ps
## CONTAINER ID   IMAGE     COMMAND             CREATED          STATUS          PORTS     NAMES
```

不過可以使用 `docker ps --all` 來觀察所有執行過的 Container

```docker
docker ps --all
## CONTAINER ID   IMAGE         COMMAND             CREATED         STATUS                          PORTS     NAMES
## b4641bd43fd2   busybox       "ping google.com"   5 minutes ago   Exited (1) About a minute ago             festive_kirch
## e77dc5137940   busybox       "echo hi there"     5 minutes ago   Exited (0) 5 minutes ago                  vigorous_bhabha
## afee7fef43c8   hello-world   "ls"                5 minutes ago   Created                                   wizardly_murdock
## 40cdbda4c0c6   busybox       "sh"                23 hours ago    Exited (137) 8 minutes ago                nice_jang
```

---

# View all images in my computer
可以利用 `docker image ls` 觀看目前你電腦中所有使用過的 Images

```docker
docker image ls
## REPOSITORY    TAG       IMAGE ID       CREATED       SIZE
## hello-world   latest    46331d942d63   5 days ago    9.14kB
## busybox       latest    d38589532d97   13 days ago   1.41MB
```

可以看到 Image 的一些資訊， `Tag`, `ID` , `Create time`, `size` 等等。

---

# Container Lifecycle
實際上在執行 `docker run` 會是一次執行兩個 command，分別是 `docker create` 和 `docker start` 

## Create a container
使用 `docker create <image name>` 可以利用指定的 image 創建出一個 Container， 他的行為會是將利用 image 創建出一個新的 container 並將 image 中的所有依賴都放進這個 Container 裏面。
<img src="https://ithelp.ithome.com.tw/upload/images/20220706/20124767SeUzI6wyKf.png" alt="drawing" width="860"/>

當使用 `docker create <image name>` 創建 Container 後，會將 Container 創建出來並在終端機中顯示一個 Container id。

```docker
docker create hello-world
# 120fa121f9d8e88b6e2a3476e16f73d66b25cd60b120f21caee83e8d0eda8bbe
```

## Start a container
使用 `docker start <container id` 可以將指定 id 的 container 跑起來，而他的行為會是將 image 中的指令放進 container 中執行（因為在 `docker create` 時就已經將所有執行這個指令所需要的依賴放進該 Container 裡面了 ）
<img src="https://ithelp.ithome.com.tw/upload/images/20220706/201247675Gf0gUQ3mt.png" alt="drawing" width="860"/>

可以利用透過 `docker create <image name>` 所得到的 container id 執行這個 container

```docker
docker start -a 120fa121f9d8e88b6e2a3476e16f73d66b25cd60b120f21caee83e8d0eda8bbe

## Hello from Docker!
## This message shows that your installation appears to be working correctly.
## ....
```

這邊的 `-a` 是為了將執行 container 的結果打印在終端機上。

---

# Restarting Stopped Containers
可以利用 `docker ps --all` 觀看到所有這台電腦中執行過的 Container，在這個清單中可以看到 Container id，所以也可以利用 `docker start <container id>` 重新執行已經停止的 Container

```docker
docker ps --all
## CONTAINER ID   IMAGE         COMMAND             CREATED         STATUS                     PORTS     NAMES
## 120fa121f9d8   hello-world   "/hello"            6 minutes ago   Exited (0) 5 minutes ago             nervous_elgamal

docker start -a 120fa121f9d8
## Hello from Docker!
## This message shows that your installation appears to be working correctly.
## ...
```

不過要注意的是，如果這個停下來的 Container 已經有預設的動作了，那就不能在 restart 的時候附加一個新的動作以取代原有的動作

```docker
docker run busybox echo hi there
## hi there

docker ps -a
## CONTAINER ID   IMAGE     COMMAND           CREATED          STATUS                      PORTS     NAMES
## 67ec9cb677b7   busybox   "echo hi there"   16 seconds ago   Exited (0) 15 seconds ago             nifty_lichterman

docker start -a 67ec9cb677b7 echo by there
## you cannot start and attach multiple containers at once
```

---

# Removing Stopped Containers
可以透過使用 `docker system prune` 來刪除已經停下的 Container

```docker
docker ps -a
## CONTAINER ID   IMAGE     COMMAND           CREATED       STATUS                   PORTS     NAMES
## 67ec9cb677b7   busybox   "echo hi there"   2 hours ago   Exited (0) 2 hours ago             nifty_lichterman
## 118a74cef87c   busybox   "sh"              2 hours ago   Exited (0) 2 hours ago             great_stonebraker

docker system prune
## WARNING! This will remove:
##   - all stopped containers
##   - all networks not used by at least one container
##   - all dangling images
##   - all dangling build cache

Are you sure you want to continue? [y/N] y

## Deleted Containers:
## 67ec9cb677b7467be1483963af39ac4bb2af81859dc96ec5f188ad475cb5c1e5
## 118a74cef87cf677e0d08c6e04496b126e15b7ef6ecf1c541d8ac28a28c60887

## Total reclaimed space: 0B
```

會出現一個 `Warning` 的介面，會跟你說他會做以下的動作，如果確認的話請輸入 `y` ，當完成移除停止的 Container 後在使用 `docker ps -a` 就可以看到所有 Container 都會被移除

```docker
docker ps -a
## CONTAINER ID   IMAGE     COMMAND   CREATED   STATUS    PORTS     NAMES
```

---

# Retrieving Log Output
當使用 `docker start <container id>` 時，因為 `-a` 是用於顯示 Container 執行的結果，如果沒有加的話則會

```docker
docker ps -a
## CONTAINER ID   IMAGE     COMMAND           CREATED         STATUS                     PORTS     NAMES
## 5899ed01fc52   busybox   "echo hi there"   6 seconds ago   Exited (0) 5 seconds ago             clever_shockley

docker start 5899ed01fc52
## 5899ed01fc52
```

他就只會顯示 `Container id` 而不會將結果顯示出來，如果要看到結果的話，除了在下一次 `docker start -a <container id>` 之外，還可以使用 `docker log <container id>`

```docker
docker logs 5899ed01fc52
## hi there
```

這樣的話就不需要重新執行一次 Container 的內容就可以看到 Container 的執行結果。

---

# Stopping Container
當我們執行一個持續運作的 Container 時，可以利用兩種 command 停止這個 Container

## Stop a Container
可以使用 `docker stop <container id>` 來停止指定 id 的 Container

```docker
docker run busybox ping google.com

docker ps
## CONTAINER ID   IMAGE     COMMAND             CREATED          STATUS          PORTS     NAMES
## dd14ec4cdce8   busybox   "ping google.com"   21 seconds ago   Up 20 seconds             peaceful_albattani

docker stop dd14ec4cdce8
## waiting for 10s
## dd14ec4cdce8

docker ps
## CONTAINER ID   IMAGE     COMMAND             CREATED          STATUS          PORTS     NAMES
```

使用 `docker stop <container id>` 後，會需要等待 10 秒才能夠將 Container 停止。

## Kill a Container
也可以使用 `docker kill <container id>` 來停止指定 id 的 Container，但與 `docker stop <container id>` 不同的是他會直接將這個 Container 結束掉，不需要等 10 秒。

```docker
docker run busybox ping google.com

docker ps
## CONTAINER ID   IMAGE     COMMAND             CREATED         STATUS         PORTS     NAMES
## dd14ec4cdce8   busybox   "ping google.com"   2 minutes ago   Up 6 seconds             peaceful_albattani

docker kill dd14ec4cdce8
## dd14ec4cdce8
```

---

# Multi-Commend Containers
當有多個 Command 需要在同一個 Container 中執行時，需要使用 `docker exec -it <container id> <command>` 以下會以 `redis` 這個 database 舉例

```docker
docker run redis
## ...
## 1:M 25 Mar 2022 08:57:59.208 * Ready to accept connections
```

上面的例子中利用 `docker run redis` 創建了一個可以執行 `redis` 的 Container，所以目前的狀況就是正在執行 `redis server` ，但如果要在這個 Container 中使用其他指令那麼就會出錯

```bash
redis-cli
## zsh: command not found: redis-cli
```

因為如為每個 Container 都是獨立運作的，所以如果開了另一個終端機他其實是與 `redis container` 沒有關係的
<img src="https://ithelp.ithome.com.tw/upload/images/20220706/20124767PWLp1vwqou.png" alt="drawing" width="860"/>

以上面的圖來看的話，就是實際上有在 `redis container` 中執行的 Command 只有 `redis-server` 但對於 `redis-cli` 來說是獨立在外面的，所以並沒有可以執行 `redis-cli` 的環境，所以如果要 `redis-cli` 也存在於 `redis container` 的話，就需要使用上面介紹的 `docker exec -it <container id> <command>` 指令，他會讓結構變成
<img src="https://ithelp.ithome.com.tw/upload/images/20220706/201247674fiATpXILP.png" alt="drawing" width="860"/>

```docker
docker ps
## CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS      NAMES
## 86d023911a69   redis     "docker-entrypoint.s…"   10 minutes ago   Up 10 minutes   6379/tcp   dreamy_chatelet

docker exec -it 86d023911a69 redis-cli
127.0.0.1:6379> set myvalue 5
## OK
127.0.0.1:6379> get myvalue
## "5"
```
這樣就可以在同一個 Container 中執行多個 Command

---

# The Purpose of the -it Flag
在上面利用 `docker exec -it <container id> <command>` 達成在同個 Container 執行多個指令的目的，而這個 `-it` 是一個聯合指令，在 linux 中的架構大概會長這樣
<img src="https://ithelp.ithome.com.tw/upload/images/20220706/20124767iDjRbcBxY7.png" alt="drawing" width="860"/>

正在執行的程式（ `redis-cli` ）中會有個輸入, 輸出與錯誤輸出，我們的終端機會將我們輸入的內容傳給 `STDIN` 讓程式執行使用，相對的當程式執行的結果或錯誤也會利用 `STDOUT` 或 `STDERR` 傳給我們的終端機讓我們知道結果或是有沒有錯誤。

所以當我們輸入 `-it` 事實上是兩個 flag 分別是 `-i` 與 `-t` ， `-i` 的目的是允許我們將終端機輸入的內容傳給 `STDIN` 相對的 `-t` 就是允許程式的結果輸出在我們的終端機上，所以可以做一些嘗試，比如不要加入 `-t`

```docker
docker ps
## CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS      NAMES
## 86d023911a69   redis     "docker-entrypoint.s…"   19 minutes ago   Up 19 minutes   6379/tcp   dreamy_chatelet

docker exec -i 86d023911a69 redis-cli

set myValue 5
## OK
get myValue
## 5
```

依然可以正常使用，只是 `redis` 的一些輸出就不會顯示在終端機上。

---

# Getting a Command Prompt in Container
在使用 Docker 時，雖然可以使用 `docker exec -it <container id> <command>` 不斷的下你所須要執行的指令，但如果遇到需要使用多個指令的時候，就會需要不斷的重複輸入 `docker exec ...` 這樣其實蠻麻煩的，所以一般都會使用 `shell access` 或 `terminal access` 來操作。

```docker
docker ps
## CONTAINER ID   IMAGE     COMMAND                  CREATED          STATUS          PORTS      NAMES
## 1d4f917b9525   redis     "docker-entrypoint.s…"   12 seconds ago   Up 10 seconds   6379/tcp   festive_perlman

docker exec -it 1d4f917b9525 sh
# redis-cli
127.0.0.1:6379> set myValue 5
## OK
127.0.0.1:6379> get myValue
## 5
```

可以使用 `control + D` 離開 shell。

同樣的也可以使用 `docker run <image id> sh` 直接創建一個 shell access 的 Container

```docker
docker run -it busybox sh
/ # ls
## bin   dev   etc   home  proc  root  sys   tmp   usr   var
/ # echo hi there
## hi there

docker ps
## CONTAINER ID   IMAGE     COMMAND   CREATED          STATUS          PORTS     NAMES
## 954fb185cf45   busybox   "sh"      30 seconds ago   Up 30 seconds             unruffled_ishizaka
```

---

# Container Isolation
就算是利用同的 Image 創建出的多個 Container，他們之間也是互相獨立的，舉個例子

```docker
# Terminal-1
docker run -it busybox sh
/ # ls
## bin   dev   etc   home  proc  root  sys   tmp   usr   var

/ # touch hiThere
/ # ls
## bin      dev      etc   hiThere  home     proc     root     sys      tmp      usr      var
```

在 Terminal-1 中利用 `touch` 創建一個新的檔案 `hiThere` 接著到 Terminal-2 看一下另一個 Container 是否有這個檔案

```docker
docker run -it busybox sh
/ # ls
## bin   dev   etc   home  proc  root  sys   tmp   usr   var
```

可以看到第二個 Container 是沒有剛剛 Container-1 所創建的檔案，這樣代表每個 Container 都是獨立運作的，並不會互相影響。

---

# Reference
- [Docker and Kubernetes: The Complete Guide](https://www.udemy.com/course/docker-and-kubernetes-the-complete-guide/)