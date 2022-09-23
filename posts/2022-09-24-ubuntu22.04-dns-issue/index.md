# Ubuntu 22.04 설치 후 특정 도메인에 접근 못하는 문제 수정


* 증상
Ubuntu 22.04.1 설치 후 Github.com 사이트에 접근이 안되는 문제가 발생했다. 다른 사이트는 모두 접근이 가능하고, 심지어 와이파이로 인터넷 연결 시에는 Github.com 사이트도 접근이 가능하지만 유선 인터넷 연결 시에는 Github.com 도메인을 포함한 일부 도메인은 접근을 못하는 문제가 발생하고 있는 상태이다.
* 원인
~DHCP~ 관련 문제이다. 이유는 모르겠지만 DHCP가 제대로 동작하지 않아서 특정 도메인만 접근할 수 없는 것으로 보인다. ~sudo dhclient -r~ 명령어를 실행하면 정상적으로 Github.com 사이트에 접근할 수 있지만 *재부팅을 하면 다시 접근이 불가능해지는 문제가 생긴다.*
* 해결법
~dhcpcd~ 데몬을 이용해서 해결할 수 있다. 아래의 명령어를 통해 관련 패키지를 설치하면 이제 재부팅 후에도 정상적으로 인터넷을 이용할 수 있다.

#+BEGIN_SRC bash
sudo apt update
sudo apt install dhcpcd-gtk
systemctl enable --now dhcpcd
#+END_SRC

