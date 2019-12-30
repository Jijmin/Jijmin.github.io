### docker 中 images 无法删除

```
[root@zhouying ~]# docker images
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
zy/ulove            latest              e3d67b62e6a4        6 days ago          1.27GB
mongo               latest              965553e202a4        6 weeks ago         363MB
node                10.16               a68faf70e589        8 weeks ago         904MB
hello-world         latest              fce289e99eb9        11 months ago       1.84kB
node                8.4                 386940f92d24        2 years ago         673MB
[root@zhouying ~]# docker rmi -f fce289e99eb9
Error: No such image: fce289e99eb9

cd `find /var/lib/docker -path '*/imagedb/*tent/sha256'`
ll
rm -f fce289e99eb9bca977dae136fbe2a82b6b7d4c372474c9235adc1741675f587e
```
