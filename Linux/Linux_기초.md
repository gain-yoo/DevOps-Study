# Linux 기초
1. AWS Free-tier 계정으로 EC2 Amazon Linux2 인스턴스(t2a.micro)를 생성하고 ssh로 접속해보세요.
    - 위 sshd 접속 과정을 설명해주세요.
        1. 먼저 client에서 `ssh-keygen` 도구를 사용하여 server에 접속하기 위한 public key와 private key를 생성한다.  
            - `/home/ec2-user/.ssh/id_rsa` : **private key**로 client가 가지고 있는다
            - `/home/ec2-user/.ssh/id_rsa.pub` : **public key**로 client에서 server에게 전달하여  `authorized_keys` 파일에 이어쓰기한다.

            ``` bash
            [ec2-user@client bin]$ ssh-keygen -t rsa
            Generating public/private rsa key pair.
            Enter file in which to save the key (/home/ec2-user/.ssh/id_rsa):
            Enter passphrase (empty for no passphrase):
            Enter same passphrase again:
            Your identification has been saved in /home/ec2-user/.ssh/id_rsa.
            Your public key has been saved in /home/ec2-user/.ssh/id_rsa.pub.
            The key fingerprint is:
            SHA256:gAKz4PUfobOgDGv3MyHOyXQ2Vw81jN+g260bEGIfwns ec2-user@ip-172-31-36-114.ap-northeast-2.compute.internal
            The key''s randomart image is:
            +---[RSA 2048]----+
            |+  .   .  oo     |
            |o+. ..o ...o.    |
            |o....+.* =o o    |
            |o.... =.B.=. .   |
            |.+ + * +SEo..    |
            |. * * + .... .   |
            |   = +     ..    |
            |      o    ..    |
            |           ..    |
            +----[SHA256]-----+
            
            [ec2-user@client .ssh]$ ls
            authorized_keys  id_rsa  id_rsa.pub

            [ec2-user@client .ssh]$ cat id_rsa
            -----BEGIN RSA PRIVATE KEY-----
            MIIEpgIBAAKCAQEAzJIW...중략...iwLdTgGI43kdt3uQ
            -----END RSA PRIVATE KEY-----

            [ec2-user@client .ssh]$ cat id_rsa.pub
            ssh-rsa AAAAB3NzaC1yc2EAAAADAQABAAABAQDMkhZe4TMZpJzQhruJ15...중략...+Q1kU9h5fFfyU6CPLhrdJcqJsf ec2-user@ip-172-31-36-114.ap-northeast-2.compute.internal
            ```

        2. 키를 생성하였으면 `ssh-copy-id` 도구로 client에서 server로 **public key**를 전송한다.  
        그러나 그냥 접속하게 되면 `Permission denied (publickey,gssapi-keyex,gssapi-with-mic).` 오류가 발생한다. 그래서 먼저 server의 `sshd_config` 파일에서 **PubkeyAuthentication**와 **PasswordAuthentication**을 허용해 주고 ssh 프로세스를 restart해 줘야 한다.

            ```bash
            [ec2-user@server .ssh]$ sudo vi /etc/ssh/sshd_config
                PubkeyAuthentication yes
                PasswordAuthentication yes
            [ec2-user@server .ssh]$ sudo systemctl restart sshd.service

            [ec2-user@client ~]$ ssh-copy-id -i ~/.ssh/id_rsa.pub ec2-user@3.34.177.229
            /usr/bin/ssh-copy-id: INFO: Source of key(s) to be installed: "/home/ec2-user/.ssh/id_rsa.pub"
            /usr/bin/ssh-copy-id: INFO: attempting to log in with the new key(s), to filter out any that are already installed
            /usr/bin/ssh-copy-id: INFO: 1 key(s) remain to be installed -- if you are prompted now it is to install the new keys
            ec2-user@3.34.177.229's password:

            Number of key(s) added: 1

            Now try logging into the machine, with:   "ssh 'ec2-user@3.34.177.229'"
            and check to make sure that only the key(s) you wanted were added.
            ```
            
            그럼 server의 `authorized_keys` 파일에 **public key**가 자동으로 등록되어 있다.

            ```bash
            [ec2-user@server ~]$ cat .ssh/authorized_keys
            ssh-rsa AAAAB3NzaC1yc2E...중략...hrdJcqJsf ec2-user@ip-172-31-36-114.ap-northeast-2.compute.internal
            ```

        3. 이제 client에서의 private key와 server에서의 public key를 가지고 ssh handshake를 맺어 암호화 통신을 하고 원격 접속이 가능하게 된다.

            ```bash
            [ec2-user@client ~]$ ssh ec2-user@3.34.177.229
            Last login: Tue Oct 24 13:06:22 2023
            ,     #_
            ~\_  ####_        Amazon Linux 2
            ~~  \_#####\
            ~~     \###|       AL2 End of Life is 2025-06-30.
            ~~       \#/ ___
            ~~       V~' '->
                ~~~         /    A newer version of Amazon Linux is available!
                ~~._.   _/
                    _/ _/       Amazon Linux 2023, GA and supported until 2028-03-15.
                _/m/'           https://aws.amazon.com/linux/amazon-linux-2023/

            21 package(s) needed for security, out of 23 available
            Run "sudo yum update" to apply all updates.
            [ec2-user@server ~]$
            ```

    - 퍼블릭 키는 어디에 있나요?
        - server에서 `~/.ssh/authorized_keys`에 있다.
        
    - 리눅스 내부에서 접속 포트 번호를 22에서 2022로 변경하려면 어떻게 할까요?
        1. 먼저 server의 `sshd_config`에서 port를 변경해 준다.

            ```bash
            [ec2-user@server .ssh]$ sudo vi /etc/ssh/sshd_config
            Port 2022
            [ec2-user@server .ssh]$ sudo systemctl restart sshd.service
            ```

        2. 그럼 client에서는 22번으로는 Connection refused가 발생하고 변경된 port로 접속할 수 있게 된다.

            ```bash
            [ec2-user@client ~]$ ssh ec2-user@3.34.177.229 -p 22
            ssh: connect to host 3.34.177.229 port 22: Connection refused

            [ec2-user@client ~]$ ssh ec2-user@3.34.177.229 -p 2022
            ```
2. 현재 사용 중인 리눅스의 파일 시스템을 조회하는 명령어를 입력하고 결과를 작성해주세요.
3. 최상위 루트 디렉토리('/')의 하위 디렉토리를 간략하게 설명해주세요.
    - /bin
    - /dev
    - /etc
    - /lib
    - /mnt
    - /proc
    - /usr
4. 현재 사용 중인 쉘과 사용 가능한 쉘의 목록을 확인하는 명령어를 각각 입력해주세요.
    - 쉘이란 무엇일까요?
5. ls 명령어를 입력하면 현재 디렉토리 내 파일과 디렉토리의 목록을 반환합니다. 해당 명령어의 입력부터 출력까지의 과정을 설명해주세요.
    > 작성 키워드 : **fork & exec(system call)**
    - system call이란 무엇일까요?
6. ps -ef | grep "/bin/sh" 명령어를 입력해보시고, 파이프라인('**|**') 문자가 어떤 기능인지 설명해주세요.
    - 마찬가지로 ls | sort | cd /home/ec2-user 명령어를 입력하고, 어떤 내용이 출력되는지 작성해주세요.
        - 만약 출력되지 않는다면 그 이유를 설명해주세요.
7. pstree 명령어를 입력해주세요.
    - 최상단 프로세스(init 또는 systemd)의 역할은 무엇인지 설명해주세요.
    - init과 systemd의 차이점을 설명해주세요.
8. 네이버(www.naver.com)의 IP 주소를 확인하는 명령어는 무엇인가요?
    - 실행 결과(IP 주소)를 받는 과정을 설명해주세요(DNS 질의 과정)
    - /etc/hosts와 /etc/resolv.conf의 차이점을 설명해주세요.
9. curl, telnet, ping의 차이점을 설명해주세요.
10. yum 패키지를 통해 httpd를 설치하고, 80 포트를 개방하여 서비스를 실행해주세요.
    - yum과 apt의 차이점은 무엇인가요?
    - httpd 포트 상태(LISTEN…)를 확인하는 명령어를 입력하고 결과를 작성해주세요.
11. 다음은 리소스 모니터링을 수행하는 명령어 중 top 명령어를 수행한 결과입니다. 각각의 값이 무엇을 의미하는지 설명해주세요.
    ```bash
    $top
    top - 22:39:40 up 0 min,  0 users,  load average: 0.00, 0.00, 0.00
    Tasks:   5 total,   1 running,   4 sleeping,   0 stopped,   0 zombie
    %Cpu(s):  0.0 us,  0.0 sy,  0.0 ni,100.0 id,  0.0 wa,  0.0 hi,  0.0 si,  0.0 st
    MiB Mem :   6948.5 total,   6844.0 free,     75.0 used,     29.5 buff/cache
    MiB Swap:      0.0 total,      0.0 free,      0.0 used.   6733.4 avail Mem

    PID USER     PR  NI    VIRT    RES    SHR  S  %CPU  %MEM     TIME+  COMMAND
    1   root     20   0    1276    788    472  S   0.0   0.0   0:00.03  init
    14  root     20   0    1284    388     20  S   0.0   0.0   0:00.00  init
    15  root     20   0    1284    396     20  S   0.0   0.0   0:00.00  init
    16  eljoelee 20   0    6200   5028   3316  S   0.0   0.1   0:00.02  bash
    27  eljoelee 20   0    7788   3268   2904  R   0.0   0.0   0:00.00  top
    ```
    - load average
    - tasks
        - sleeping
        - zombie
    - Cpu(s)
        - hi
        - si
    - MiB
        - buff/cache
    - PR
    - VIRT
    - RES
    - SHR
    - MEM
12. 메모리 사용량을 확인하는 명령어를 입력하고 결과를 작성해주세요.
    - swap이란 항목은 무엇을 의미하는 걸까요?
        - 2GB 가량의 swap 메모리를 설정하고 메모리 사용량을 확인하는 명령어를 통해 결과를 작성해주세요.
