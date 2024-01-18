查看容器占用的磁盘空间大小
```
docker ps -s 

CONTAINER ID   IMAGE     COMMAND                  CREATED       STATUS       PORTS                               NAMES     SIZE
8c3896ec7608   mysql     "docker-entrypoint.s…"   2 days ago    Up 2 days    0.0.0.0:3306->3306/tcp, 33060/tcp   mysql     1.17GB (virtual 1.81GB)
dd8f628a32a7   redis     "docker-entrypoint.s…"   3 weeks ago   Up 3 weeks   0.0.0.0:6379->6379/tcp              redis     0B (virtual 158MB)
```


查看实时的CPU、IO情况
```
docker stat mysql

CONTAINER ID   NAME      CPU %     MEM USAGE / LIMIT     MEM %     NET I/O          BLOCK I/O       PIDS

8c3896ec7608   mysql     13.84%    1.064GiB / 11.92GiB   8.92%     1.41GB / 685MB   1.52MB / 15GB   51
```



