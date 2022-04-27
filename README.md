mininet

ryu controller에서 To switch ofpPacket() 구조 확인
어떤식으로 보내고. 어떤식으로 받는지 확인

2x2 grid topology에서 host1 -> host4로 ros2 run talker, listener실행
switch routing path, controller가 수정하게 해주기

    [--if-exists] del-br bridge
    Deletes bridge and all of  its  ports.   If  bridge  is  a  real
    bridge,  this  command  also  deletes any fake bridges that were
    created with bridge as parent, including all of their ports.

    Without --if-exists, attempting to delete a bridge that does not
    exist  is  an  error.   With --if-exists, attempting to delete a
    bridge that does not exist has no effect.


sudo mn --topo linear,3 --mac --switch ovsk --controller remote -x

* switch CMD *
ovs-vsctl set Bridge s1 protocols=OpenFlow13
ovs-vsctl set Bridge s2 protocols=OpenFlow13
ovs-vsctl set Bridge s3 protocols=OpenFlow13

* host h1 CMD *
ip addr del 10.0.0.1/8 dev h1-eth0
ip addr add 172.16.20.10/24 dev h1-eth0

* host h2 CMD *
ip addr del 10.0.0.2/8 dev h2-eth0
ip addr add 172.16.10.10/24 dev h2-eth0

* host h3 CMD *
ip addr del 10.0.0.3/8 dev h3-eth0
ip addr add 192.168.30.10/24 dev h3-eth0

* host h4 CMD *
ip addr del 10.0.0.4/8 dev h4-eth0
ip addr add 192.168.40.10/24 dev h4-eth0

* Controller c1 CMD *
ryu-manager ryu.app.rest_router

-> 구성 끝
    컨트롤러에서 스위치/라우터에 주소 설정
* s1 *
curl -X POST -d '{"address":"172.16.20.1/24"}' http://localhost:8080/router/0000000000000001
curl -X POST -d '{"address": "172.16.30.30/24"}' http://localhost:8080/router/0000000000000001

* s2 *
curl -X POST -d '{"address":"172.16.10.1/24"}' http://localhost:8080/router/0000000000000002
curl -X POST -d '{"address": "172.16.30.1/24"}' http://localhost:8080/router/0000000000000002
curl -X POST -d '{"address": "192.168.10.1/24"}' http://localhost:8080/router/0000000000000002

* s3 *
curl -X POST -d '{"address": "192.168.30.1/24"}' http://localhost:8080/router/0000000000000003
curl -X POST -d '{"address": "192.168.10.20/24"}' http://localhost:8080/router/0000000000000003

* s4 *
curl -X POST -d '{"address": "192.168.40.1/24"}' http://localhost:8080/router/0000000000000003
curl -X POST -d '{"address": "192.168.10.20/24"}' http://localhost:8080/router/0000000000000003

-> 각 host에 라우터의 정보를 기본 게이트웨이로 등록

* host h1 CMD * # To s1
ip route add default via 172.16.20.1

* host h2 CMD * # To s2
ip route add default via 172.16.10.1

* host h3 CMD * # To s3
ip route add default via 192.168.30.1

* host h4 CMD * # To s4

* 기본 경로 설정 Controller에서 입력 *
s1 -> s2
curl -X POST -d '{"gateway": "172.16.30.1"}' http://localhost:8080/router/0000000000000001

s2 -> s1
curl -X POST -d '{"gateway": "172.16.30.30"}' http://localhost:8080/router/0000000000000002

s3 -> s2
curl -X POST -d '{"gateway": "192.168.10.1"}' http://localhost:8080/router/0000000000000003

