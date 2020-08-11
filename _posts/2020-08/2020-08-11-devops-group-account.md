---
title: Centos group 계정
tags: [devops, centos]
comments: true
categories: devops
header:
  teaser: "/assets/images/linux.png"
---



Group에 속한 사용자들 사이에서는 파일 공유가 가능합니다. 

예를 들어, tomas와 james가 public 그룹에 속해있다고 가정하였을 때,

```sh
# groupadd public
# useradd tomas -G public
# useradd james -G public
```

<br/>

tomas가 만든 문서에 james는 수정을 할 수 있습니다.

```sh
[tomas@centos1 ~]$ echo "hi~ my name is tomas" >> /tmp/chat.txt
[tomas@centos1 ~]$ chown tomas:public /tmp/chat.txt 

[james@centos1 ~]$ echo "nice to meet you" >> /tmp/chat.txt
```

<br/>

반면에, 문서의 소유권한에 그룹계정이 생략된 경우에는 <br/>

동일한 그룹 맴버라 할지라도 수정은 불가능하며 읽기만 가능합니다.

```sh
[tomas@centos1 ~]$ echo "my private diary" >> /tmp/diary.txt
[james@centos1 ~]$ cat /tmp/diary.txt 
my private diary
[james@centos1 ~]$ echo "test" >> /tmp/diary.txt
-bash: /tmp/diary.txt: 허가 거부
```

<br/>

그리고, 이와 같은 Group의 특징을 잘 활용하면,<br/>

서로 다른 n개의 서비스에서 발생하는 로그를 Group 으로 묶고,<br/>

Group 에 속한 계정으로 서비스들의 로그를 수집하고자 할 때  유용하게 사용할 수 있습니다.






