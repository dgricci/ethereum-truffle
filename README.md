% Environnement Ethereum - Truffle environment
% Didier Richard
% rév. 0.0.1 du 11/11/2016

---

# Building #

```bash
$ docker build -t dgricci/ethereum-truffle:0.0.1 -t dgricci/ethereum-truffle:latest .
```

## Behind a proxy (e.g. 10.0.1.2:3128) ##

```bash
$ docker build \
    --build-arg http_proxy=http://10.0.1.2:3128/ \
    --build-arg https_proxy=http://10.0.1.2:3128/ \
    -t dgricci/ethereum-truffle:0.0.1 -t dgricci/ethereum-truffle:latest .
```

# Use #

See `dgricci/jessie` README for handling permissions with dockers volumes.

[Truffle](https://github.com/ConsenSys/truffle) works with a Ethereum client
when migrating and testing. In the below example,
[testRPC](https://github.com/ethereumjs/testrpc) is used ... The difficulty is
to make to two containers talking, so we create a network for that !

```bash
$ docker network create -d bridge ethereum-nw
...hash...
$ docker run -d -e USER_ID=${UID} -e USER_GP=`id -g` -e USER_NAME=${USER} --name=testrpc dgricci/ethereum-testrpc testrpc
...hash...
$ rpcId="$(docker ps -qa)"
$ docker logs ${rpcId}
EthereumJS TestRPC v3.0.0

Available Accounts
==================
...

Listening on localhost:8545
$ docker network connect ethereum-nw testrpc
$ cd your-ethereum-project
# initialisation of the project :
$ docker run -it --rm -e USER_ID=${UID} -e USER_GP=`id -g` -e USER_NAME=${USER} -v `pwd`:/yepdev -w /yepdev dgricci/ethereum-truffle truffle init
# edit your .sol files and co ...
# compilation of the project :
$ docker run -it --rm -e USER_ID=${UID} -e USER_GP=`id -g` -e USER_NAME=${USER} -v `pwd`:/yepdev -w /yepdev dgricci/ethereum-truffle truffle compile
# deployment : it needs to connect to the ethereum client ...
$ sed -i -e 's/localhost/testrpc/' truffle.js
$ docker run -it --rm --net ethereum-nw --link testrpc -e USER_ID=${UID} -e USER_GP=`id -g` -e USER_NAME=${USER} -v `pwd`:/yepdev -w /yepdev dgricci/ethereum-truffle truffle migrate --reset
# testing :
$ docker run -it --rm --net ethereum-nw --link testrpc -e USER_ID=${UID} -e USER_GP=`id -g` -e USER_NAME=${USER} -v `pwd`:/yepdev -w /yepdev dgricci/ethereum-truffle truffle test
$ cd ..
$ docker network disconnect ethereum-nw testrpc
$ docker stop testrpc
$ docker rm testrpc
$ docker network rm ethereum-nw
```

__Et voilà !__


_fin du document[^pandoc_gen]_

[^pandoc_gen]: document généré via $ `pandoc -V fontsize=10pt -V geometry:"top=2cm, bottom=2cm, left=1cm, right=1cm" -s -N --toc -o ethereum-truffle.pdf README.md`{.bash}

