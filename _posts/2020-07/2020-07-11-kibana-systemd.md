---
title: Kibana systemd 설정
comments: true
tags: [devops, kibana]
categories: devops
header:
  teaser: "/assets/images/kibana.png"
---

Kibana는 실행시점에 /var/run/kibana 디렉토리에 PID를 기록하는데 
이 때, /var/run/kibana 디렉토리가 없거나, 
/var/run/kibana 디렉토리의 쓰기 권한이 없을 경우
Kibana 는 더 이상 실행되지 않습니다. 

이러한 문제를 방지하기 위해서는 
Kibana 의 실행 이전에 /var/run/kibana 디렉토리를 만들고,
실행 시점에 sudo -u 명령어로 Kibana를 실행하면 됩니다.

##### /etc/systemd/system/kibana.service 예제


```ini
[Unit]
Description=Kibana
StartLimitIntervalSec=30
StartLimitBurst=3

[Service]
Type=simple
User=root
Group=root

# Load env vars from /etc/default/ and /etc/sysconfig/ if they exist.
# Prefixing the path with '-' makes it try to load, but if the file doesn't
# exist, it continues onward.
EnvironmentFile=-/etc/default/kibana
EnvironmentFile=-/etc/sysconfig/kibana
ExecStartPre=/usr/bin/mkdir -p /var/run/kibana
ExecStartPre=/usr/bin/chown -R kibana:kibana /var/run/kibana
ExecStart=/usr/bin/sudo -u kibana /usr/share/kibana/bin/kibana "-c /etc/kibana/kibana.yml"
Restart=always
WorkingDirectory=/

[Install]
WantedBy=multi-user.target
```

