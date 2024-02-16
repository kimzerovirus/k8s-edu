# install ansible
- install-vm 에서 실행
```sh 
## user로 한다  
sudo apt update
sudo apt install ansible --yes
ansible --version

ssh-keygen -t rsa -b 4096 
ls ~/.ssh/

cat ~/.ssh/id_rsa.pub 
## text 편집기에서 1줄로 정리한다 

```
## master-1 서버에서 ssh 설정 
```sh
## ubuntu 유저로 실행 
## master-1 서버 접속하여 install-vm의 id_rsa.pub 값을 master-1의 authorized_keys 에 추가한다
vi ~/.ssh/authorized_keys

## 패스워드 없이 접속하기 ( sudo 권한은 이미 부여 되어 있음 )
sudo vi /etc/sudoers

## 맨 하단에 아래 추가 한다 
ubuntu ALL=(ALL) NOPASSWD: ALL

```

## install-vm에서  ansible ping test
vi host-vm
```sh
master-1 ansible_host=172.26.12.109 ansible_user=ubuntu ansible_port=22 ansible_ssh_private_key_file=~/.ssh/id_rsa
```

ping test
```sh 
ansible -i host-vm all -m ping
## Are you sure you want to continue connecting (yes/no/[fingerprint])? yes 한다
## --아래와 같이 출력되면 성공-----
master-1 | SUCCESS => ....
```
## ansible task 작성 

