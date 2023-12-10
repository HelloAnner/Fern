
## Install Step

[GitHub - osixia/docker-openldap: A docker image to run OpenLDAP 🐳](https://github.com/osixia/docker-openldap)

```bash
docker run -p 389:389 --name openldap --network bridge --hostname openldap-host --env LDAP_ORGANISATION="fr" --env LDAP_DOMAIN="fr.com" --env LDAP_ADMIN_PASSWORD="123456" --detach osixia/openldap
```

登录的账号： cn=admin,dc=fr,dc=com

## 可视化显示ldap数据

[GitHub - osixia/docker-phpLDAPadmin: A docker image to run phpLDAPadmin 🐳](https://github.com/osixia/docker-phpLDAPadmin)

```bash
docker run -d --privileged -p 10004:80 --name ldapAdmin --env PHPLDAPADMIN_HTTPS=false --env PHPLDAPADMIN_LDAP_HOSTS=172.17.0.6 --detach osixia/phpldapadmin
```

创建用户前，对于其GID属性，需要先创建模版 , 即选创建 posixGroup

## 桥接模式补充

桥接模式下

`系统会自动添加一个供docker使用的网桥docker0，我们创建一个新的容器时，`

`容器通过DHCP获取一个与docker0同网段的IP地址，并默认连接到docker0网桥，以此实现容器与宿主机的网络互通`

![[Pasted image 20231030190546.png|400]]


docker run -tid --name db -p 3306:3306 MySQL

在宿主机上，可以通过iptables -t nat -L -n，查到一条DNAT规则：

DNAT tcp -- 0.0.0.0/0 0.0.0.0/0 tcp dpt:3306 to:172.17.0.5:3306

上面的172.17.0.5即为bridge模式下，创建的容器IP。

很明显，bridge模式的容器与外界通信时，必定会占用宿主机上的端口，从而与宿主机竞争端口资源，对宿主机端口的管理会是一个比较大的问题。同时，由于容器与外界通信是基于三层上iptables NAT，性能和效率上的损耗是可以预见的

查看容器IP

```bash
docker inspect --format '{{ .NetworkSettings.IPAddress }}' <container-ID>
```
