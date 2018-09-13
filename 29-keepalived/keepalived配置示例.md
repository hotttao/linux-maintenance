# 29.3 keepalived 配置示例
前面我们已经介绍了如何安装和配置 keepalived，本节我们就来看看如何使用 keepalived 对 nginx 的负载均衡集群做高可用。需要提醒大家注意的是无论是学习还是以后工作，当我们配置一个复杂服务时，都应该按照简单到复杂的顺序一步步进行配置，完成一步，验证一次，成功之后在进行下一步，这样便于排错。所以本节的示例我们将按照如下顺序展示，最终完成我们的LVS 双主模型的高可用集群配置。
1. 单主模型下配置 keepalived 完成地址流动
2. 双主模型下配置 keepalived 完成地址流动
3. 单主模型的 LVS 高可用集群配置
4. 双主模型的 LVS 高可用集群配置
5. 双主模型 nginx 高可用集群配置


## 1. 单主模型下完成地址流动


## 2. 双主模型下完成地址流动
```bash		
! Configuration File for keepalived

global_defs {
  notification_email {
    root@localhost
  }
  notification_email_from keepalived@localhost
  smtp_server 127.0.0.1
  smtp_connect_timeout 30
  router_id node1
  vrrp_mcast_group4 224.0.100.19
}

vrrp_instance VI_1 {
  state MASTER
  interface eno16777736
  virtual_router_id 14
  priority 100
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass 571f97b2
  }
  virtual_ipaddress {
    10.1.0.91/16 dev eno16777736
  }

  notify_master "/etc/keepalived/notify.sh master"
  notify_backup "/etc/keepalived/notify.sh backup"
  notify_fault "/etc/keepalived/notify.sh fault"
}

vrrp_instance VI_2 {
  state BACKUP
  interface eno16777736
  virtual_router_id 15
  priority 98
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass 578f07b2
  }
  virtual_ipaddress {
    10.1.0.92/16 dev eno16777736
  }
}						
```

## 3. 通知脚本的使用方式：
```bash
#!/bin/bash
#
contact='root@localhost'

notify() {
  local mailsubject="$(hostname) to be $1, vip floating"
  local mailbody="$(date +'%F %T'): vrrp transition, $(hostname) changed to be $1"
  echo "$mailbody" | mail -s "$mailsubject" $contact
}

case $1 in
master)
  notify master
  ;;
backup)
  notify backup
  ;;
fault)
  notify fault
  ;;
*)
  echo "Usage: $(basename $0) {master|backup|fault}"
  exit 1
  ;;
esac			
```

脚本的调用方法：
1. `notify_master "/etc/keepalived/notify.sh master"`
2. `notify_backup "/etc/keepalived/notify.sh backup"`
3. `notify_fault "/etc/keepalived/notify.sh fault"`			

## 4. 单主模型的 LVS 高可用集群配置
```
! Configuration File for keepalived

global_defs {
notification_email {
  root@localhost
}
notification_email_from keepalived@localhost
smtp_server 127.0.0.1
smtp_connect_timeout 30
router_id node1
vrrp_mcast_group4 224.0.100.19
}

vrrp_instance VI_1 {
state MASTER
interface eno16777736
virtual_router_id 14
priority 100
advert_int 1
authentication {
  auth_type PASS
  auth_pass 571f97b2
}
virtual_ipaddress {
  10.1.0.93/16 dev eno16777736
}
notify_master "/etc/keepalived/notify.sh master"
notify_backup "/etc/keepalived/notify.sh backup"
notify_fault "/etc/keepalived/notify.sh fault"
}

virtual_server 10.1.0.93 80 {
delay_loop 3
lb_algo rr
lb_kind DR
protocol TCP

sorry_server 127.0.0.1 80

real_server 10.1.0.69 80 {
  weight 1
  HTTP_GET {
  url {
    path /
    status_code 200
  }
  connect_timeout 1
  nb_get_retry 3
  delay_before_retry 1
  }
}
real_server 10.1.0.71 80 {
  weight 1
  HTTP_GET {
  url {
    path /
    status_code 200
  }
  connect_timeout 1
  nb_get_retry 3
  delay_before_retry 1
  }
}
}
```

## 5. 双主模型的 LVS 高可用集群配置


## 6. 高可用nginx服务
```
! Configuration File for keepalived

global_defs {
  notification_email {
    root@localhost
  }
  notification_email_from keepalived@localhost
  smtp_server 127.0.0.1
  smtp_connect_timeout 30
  router_id node1
  vrrp_mcast_group4 224.0.100.19
}

vrrp_script chk_down {
  script "[[ -f /etc/keepalived/down ]] && exit 1 || exit 0"
  interval 1
  weight -5
}

vrrp_script chk_nginx {
  script "killall -0 nginx && exit 0 || exit 1"
  interval 1
  weight -5
  fall 2
  rise 1
}

vrrp_instance VI_1 {
  state MASTER
  interface eno16777736
  virtual_router_id 14
  priority 100
  advert_int 1
  authentication {
    auth_type PASS
    auth_pass 571f97b2
  }
  virtual_ipaddress {
    10.1.0.93/16 dev eno16777736
  }
  track_script {
    chk_down
    chk_nginx
  }
  notify_master "/etc/keepalived/notify.sh master"
  notify_backup "/etc/keepalived/notify.sh backup"
  notify_fault "/etc/keepalived/notify.sh fault"
}
```
