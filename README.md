

<div align="center">  

<img src="https://i.loli.net/2020/02/21/rfOGvKlTcHCmM92.png"  /> 
<br/>

[![codecov](https://codecov.io/gh/crossoverJie/cim/graph/badge.svg?token=oW5Gd1oKmf)](https://codecov.io/gh/crossoverJie/cim)
[![Build Status](https://img.shields.io/badge/cim-cross--im-brightgreen.svg)](https://github.com/crossoverJie/cim)
[![](https://badge.juejin.im/entry/5c2c000e6fb9a049f5713e26/likes.svg?style=flat-square)](https://juejin.im/post/5c2bffdc51882509181395d7)

📘[Introduction](#介绍) |📽[Demonstration video](#视频演示) | 🏖[Liste des taches](#todo-list) | 🌈[Architecture du systeme](#系统架构) |💡[Organigramme](#流程图)|🌁[Demarrage rapide](#快速启动)|👨🏻‍✈️[Commandes intégrées](#客户端内置命令)|🎤[Communication](#群聊私聊)|❓[QA](https://github.com/crossoverJie/cim/blob/master/doc/QA.md)|💌[Contacter l'auteur](#联系作者)


</div>
<br/>

# V2.0
- [x] Mise à niveau vers JDK17 & Spring Boot 3.0
- [x] SDK Client 
- [ ] Le client utilise [picocli](https://picocli.info/) à la place de Spring Boot
- [x] Prise en charge des tests d'intégration
- [ ] Intégration d'OpenTelemetry
- [ ] Prise en charge du démarrage en nœud unique (sans composants externes)
- [ ] Possibilité de remplacer les composants tiers (Redis/Zookeeper, etc.)
- [ ] Support du client web (WebSocket)
- [x] Support des conteneurs Docker
- [ ] Support des déploiements Kubernetes
- [ ] Support du client binaire (compilé avec Golang)

## Introduction

`CIM (CROSS-IM) est un système de messagerie instantanée (IM) conçu pour les développeurs ; il fournit également des composants pour aider les développeurs à construire leur propre système IM évolutif.

Avec CIM, vous pouvez répondre aux besoins suivants :

Système de messagerie instantanée (IM).

Intergiciel de notification push pour les applications mobiles.

Intergiciel de messagerie pour les scénarios de connexion massive d'appareils IoT.

> Si vous avez des questions pendant l'utilisation ou le développement, vous pouvez [me contacter](#联系作者).

## Démonstration vidéo

> Cliquez sur les liens ci-dessous pour voir la version vidéo Demo :

| YouTube | Bilibili|
| :------:| :------: | 
| [群聊](https://youtu.be/_9a4lIkQ5_o) [私聊](https://youtu.be/kfEfQFPLBTQ) | [群聊](https://www.bilibili.com/video/av39405501) [私聊](https://www.bilibili.com/video/av39405821) | 
| <img src="https://i.loli.net//2019//05//08//5cd1d9e788004.jpg"  height="295px" />  | <img src="https://i.loli.net//2019//05//08//5cd1da2f943c5.jpg" height="295px" />

![demo.gif](pic/demo.gif)

## TODO LIST

* [x] [群聊](#群聊)
* [x] [私聊](#私聊)
* [x] [内置命令](#客户端内置命令)
* [x] [聊天记录查询](#聊天记录查询)。
* [x] [一键开启价值 2 亿的 `AI` 模式](#ai-模式)
* [x] 使用 `Google Protocol Buffer` 高效编解码
* [x] 根据实际情况灵活的水平扩容、缩容
* [x] 服务端自动剔除离线客户端
* [x] 客户端自动重连
* [x] [延时消息](#延时消息)
* [x] SDK 开发包
* [ ] 分组群聊
* [ ] 离线消息
* [ ] 消息加密



## Architecture

![](pic/architecture.png)

- Chaque composant de `CIM` est construit en utilisant `SpringBoot`
- Le client est construit avec [cim-client-sdk](https://github.com/crossoverJie/cim/tree/master/cim-client-sdk)
- Utilise `Netty` pour construire la communication de bas niveau.
- `MetaStore` est utilisé pour l'enregistrement et la découverte des services `IM-server`.


### cim-server
Le serveur IM est utilisé pour recevoir les connexions clients, le transfert de messages, l'envoi de notifications push, etc.
Prend en charge le déploiement en cluster.

### cim-route

Serveur de routage ; utilisé pour traiter le routage des messages, leur transfert, la connexion des utilisateurs, leur déconnexion, ainsi que certaines opérations de gestion (obtenir le nombre d'utilisateurs en ligne, etc.).

### cim-client
Terminal client IM, une simple commande permet de le démarrer et d'initier la communication avec d'autres personnes (chat de groupe, chat privé).

## Flow Chart

![](https://s2.loli.net/2024/10/13/8teMn7BSa5VWuvi.png)

- Le serveur s'enregistre auprès de `MetaStore`
- Le routeur s'abonne à `MetaStore`
- Le client se connecte au routeur
  - Le routeur récupère les informations du serveur depuis `MetaStore`
- Le client ouvre une connexion vers le serveur
- Le Client1 envoie un message au routeur
- Le routeur sélectionne un serveur et transfère le message vers ce serveur
- Le serveur envoie (push) le message au Client2


## Démarrage rapide

Utilisez la commande allin1 Docker pour démarrer le serveur :

```shell
docker pull docker pull ghcr.io/crossoverjie/allin1-ubuntu:latest
docker run -p 2181:2181 -p 6379:6379 -p 8083:8083 --rm --name cim-allin1  ghcr.io/crossoverjie/allin1-ubuntu:latest
```

### Demarrage Local
```shell

首先需要安装 `Zookeeper、Redis` 并保证网络通畅。

```shell
docker run --rm --name zookeeper -d -p 2181:2181 zookeeper:3.9.2
docker run --rm --name redis -d -p 6379:6379 redis:7.4.0
```

```shell
git clone https://github.com/crossoverJie/cim.git
cd cim
mvn clean install -DskipTests=true
cd cim-server && cim-client && cim-forward-route
mvn clean package spring-boot:repackage -DskipTests=true
```

### Déploiement IM-server(cim-server)

```shell
cp /cim/cim-server/target/cim-server-1.0.0-SNAPSHOT.jar /xx/work/server0/
cd /xx/work/server0/
nohup java -jar  /root/work/server0/cim-server-1.0.0-SNAPSHOT.jar --cim.server.port=9000 --app.zk.addr=zk地址  > /root/work/server0/log.file 2>&1 &
```

> cim-server Le déploiement en cluster suit le même principe，Il suffit de s'assurer que Zookeeper les adresses soient identiques。

### Déployer le serveur de routage (cim-forward-route)

```shell
cp /cim/cim-server/cim-forward-route/target/cim-forward-route-1.0.0-SNAPSHOT.jar /xx/work/route0/
cd /xx/work/route0/
nohup java -jar  /root/work/route0/cim-forward-route-1.0.0-SNAPSHOT.jar --app.zk.addr=zk地址 --spring.redis.host=redis地址 --spring.redis.port=6379  > /root/work/route/log.file 2>&1 &
```

> cim-forward-route est intrinsèquement sans état，peut être déployé sur plusieurs instances；Utilisez Nginx un proxy suffit.


### Démarrer le client

```shell
cp /cim/cim-client/target/cim-client-1.0.0-SNAPSHOT.jar /xx/work/route0/
cd /xx/work/route0/
java -jar cim-client-1.0.0-SNAPSHOT.jar --server.port=8084 --cim.user.id=唯一客户端ID --cim.user.userName=用户名 --cim.route.url=http://路由服务器:8083/
```

![](https://ws2.sinaimg.cn/large/006tNbRwly1fylgxjgshfj31vo04m7p9.jpg)
![](https://ws1.sinaimg.cn/large/006tNbRwly1fylgxu0x4uj31hy04q75z.jpg)

Comme illustré ci-dessus, démarrez deux clients qui pourront communiquer entre eux.。

### Démarrer le client localement

#### S'inscrire / Créer un compte
```shell
curl -X POST --header 'Content-Type: application/json' --header 'Accept: application/json' -d '{
  "reqNo": "1234567890",
  "timeStamp": 0,
  "userName": "zhangsan"
}' 'http://路由服务器:8083/registerAccount'
```

Extraire / Obtenir depuis les résultats retournés `userId`

```json
{
    "code":"9000",
    "message":"成功",
    "reqNo":null,
    "dataBody":{
        "userId":1547028929407,
        "userName":"test"
    }
}
```

#### Démarrer le client local
```shell
# 启动本地客户端
cp /cim/cim-client/target/cim-client-1.0.0-SNAPSHOT.jar /xx/work/route0/
cd /xx/work/route0/
java -jar cim-client-1.0.0-SNAPSHOT.jar --server.port=8084 --cim.user.id=上方返回的userId --cim.user.userName=用户名 --cim.route.url=http://路由服务器:8083/
```

## Commandes intégrées du client

| Commande | Description|
| ------ | ------ | 
| `:q!` | Quitter le client| 
| `:olu` | Obtenir les informations de tous les utilisateurs en ligne | 
| `:all` | Obtenir toutes les commandes disponibles | 
| `:q [option]` | 【:q [Mot-clé] Rechercher l'historique des discussions | 
| `:ai` | Activer le mode IA | 
| `:qai` | Désactiver le mode IA | 
| `:pu` |Recherche approximative d'utilisateurs | 
| `:info` | Obtenir les informations du client | 
| `:emoji [option]` | Rechercher des emojis/mèmes [option: numéro de page] | 
| `:delay [msg] [delayTime]` | Envoyer un message différé | 
| `:` | D'autres commandes sont en cours de développement。 | 

![](https://ws3.sinaimg.cn/large/006tNbRwly1fylh7bdlo6g30go01shdt.gif)

### Consultation de l'historique des discussions

![](https://i.loli.net/2019/05/08/5cd1c310cb796.jpg)

Utilisez la commande `:q 关键字` pour consulter l'historique des discussions liées à votre compte。

> L'historique des discussions du client est stocké par défaut dans `/opt/logs/cim/`，Il est donc nécessaire d'avoir les permissions d'écriture sur ce répertoire. Vous pouvez également ajouter dans la commande de démarrage :

 `--cim.msg.logger.path = /自定义` le paramètre pour personnaliser le répertoire.


### AI Mode

![](https://i.loli.net/2019/05/08/5cd1c30e47d95.jpg)

Utilisez la commande `:ai` pour activer le mode IA, après quoi tous les messages seront traités par `AI` pour générer une réponse.

`:qai` pour quitter le mode AI。

### Recherche d'utilisateurs par préfixe de nom

![](https://i.loli.net/2019/05/08/5cd1c32ac3397.jpg)

Utilisez la commande `:qu prefix` pour rechercher des informations d'utilisateur par préfixe.

> Cette fonction est principalement utilisée pour rechercher des utilisateurs depuis les champs de saisie sur les appareils mobiles.
### Discussion de groupe / Discussion privée

#### Discussion de groupe
![](https://ws1.sinaimg.cn/large/006tNbRwly1fyli54e8e1j31t0056x11.jpg)
![](https://ws3.sinaimg.cn/large/006tNbRwly1fyli5yyspmj31im06atb8.jpg)
![](https://ws3.sinaimg.cn/large/006tNbRwly1fyli6sn3c8j31ss06qmzq.jpg)

Pour une discussion de groupe, il suffit de saisir le message dans la console et d'appuyer sur Entrée pour l'envoyer. Tous les clients en ligne recevront alors le message.
#### Discussion privée

Pour une discussion privée, il faut d'abord connaître l'identifiant (ID) de l'autre personne `userID` avant de pouvoir communiquer。

Saisissez la commande `:olu` pour lister tous les utilisateurs en ligne.

![](https://ws4.sinaimg.cn/large/006tNbRwly1fyli98mlf3j31ta06mwhv.jpg)

接着使用 `userId;;消息内容` 的格式即可发送私聊消息。

![](https://ws4.sinaimg.cn/large/006tNbRwly1fylib08qlnj31sk082zo6.jpg)
![](https://ws1.sinaimg.cn/large/006tNbRwly1fylibc13etj31wa0564lp.jpg)
![](https://ws3.sinaimg.cn/large/006tNbRwly1fylicmjj6cj31wg07c4qp.jpg)
![](https://ws1.sinaimg.cn/large/006tNbRwly1fylicwhe04j31ua03ejsv.jpg)

同时另一个账号收不到消息。
![](https://ws3.sinaimg.cn/large/006tNbRwly1fylie727jaj31t20dq1ky.jpg)



### emoji 表情支持

使用命令 `:emoji 1` 查询出所有表情列表，使用表情别名即可发送表情。

![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j910cqrzj30dn05qjw9.jpg)
![](https://tva1.sinaimg.cn/large/006y8mN6ly1g6j99hazg6j30ax03hq35.jpg)
 
### 延时消息

发送 10s 的延时消息：

```shell
:delay delayMsg 10
```

![](pic/delay.gif)

## 联系作者

## Contributing

Nous accueillons volontiers les contributions ! Avant de soumettre une Pull Request (PR), assurez-vous que votre code passe la vérification Checkstyle.
### Code Style

This project uses [Checkstyle](https://checkstyle.org/) to enforce code style. The rules are defined in `checkstyle/checkstyle.xml`.

**Run Checkstyle locally before committing:**

```shell
mvn checkstyle:check
```

**Key rules:**
- Use spaces around `{`, `}`, and operators
- No trailing whitespace
- Files must end with a newline
- Remove unused imports
- Constants (`static final`) must be `UPPER_SNAKE_CASE`
- Use Java-style array declarations: `String[] args` (not `String args[]`)

**Skip Checkstyle for quick builds:**

```shell
mvn package -Dcheckstyle.skip=true
```

<div align="center">  

<a href="https://t.zsxq.com/odQDJ" target="_blank"><img src="https://s2.loli.net/2024/05/17/zRkabDu2SKfChLX.png" alt="202405171520366.png"></a>
</div>

J'ai récemment ouvert une "Knowledge Planet" (communauté de savoir). Merci à tous pour votre soutien à CIM. Je vous propose 100 coupons de réduction de 10 yuans, soit 69-10 = 59 yuans. Pour les avantages spécifiques, vous pouvez scanner le code QR pour plus de détails avant de décider de rejoindre ou non.
PS : Par la suite, je commencerai la refonte de la version 2.0 dans la Knowledge Planet. Ceux que cela intéresse peuvent rejoindre la communauté pour suivre de près l'avancement (le code restera bien sûr open source).

- [crossoverJie@gmail.com](mailto:crossoverJie@gmail.com)
- Compte public WeChat

![index.jpg](https://i.loli.net/2021/10/12/ckQW9LYXSxFogJZ.jpg)



