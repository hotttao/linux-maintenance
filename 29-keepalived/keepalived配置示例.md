# 29.3 keepalived 配置示例
前面我们已经介绍了如何安装和配置 keepalived，本节我们就来看看如何使用 keepalived 对 nginx 的负载均衡集群做高可用。需要提醒大家注意的是无论是学习还是以后工作，当我们配置一个复杂服务时，都应该按照简单到复杂的顺序一步步进行配置，完成一步，验证一次，成功之后在进行下一步，这样便于排错。所以本节的示例我们将按照如下顺序展示，最终完成我们的LVS 双主模型的高可用集群配置。
1. 单主模型下配置 keepalived 完成地址流动
2. 双主模型下配置 keepalived 完成地址流动
3. 单主模型的 LVS 高可用集群配置
4. 双主模型的 LVS 高可用集群配置
5. 双主模型 nginx 高可用集群配置


## 1. 单主模型下完成地址流动

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
    192.168.1.168/24 dev eno16777736
  }

  notify_master "/etc/keepalived/notify.sh master"
  notify_backup "/etc/keepalived/notify.sh backup"
  notify_fault "/etc/keepalived/notify.sh fault"
}
```

#### 通知脚本的使用方式：
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


## 2. 双主模型下完成地址流动
双主模型的地址流动，只需在单主模型下额外添加一个 vrrp 实例，在新的实例下:
1. 修改 `virtual_router_id`
2. 修改 vrrp 认证的密码
3. 修改 `virtual_ipaddress` 绑定的地址
4. 原来的 MASTER 变成 BACKUP，BACKUP 变成 MASTER

```bash
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
    192.168.1.169/24 dev eno16777736
  }
}						
```

## 3. 双主模型的 LVS 高可用集群配置
配置步骤如下:
1. 首先要配置 LVS 集群的后端 RS，可参照[27.5 LVS 4层负载均衡-DR模型](27-LVS4层负载均衡/LVS4_DR模型.md)
2. 在"双主模型下完成地址流动"的基础上添加 virtual_server，配置方式如下所示

```
virtual_server 192.168.1.168 80 {
  delay_loop 3
  lb_algo rr
  lb_kind DR
  protocol TCP

  sorry_server 127.0.0.1 80

real_server 192.168.1.137 80 {
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

real_server 192.168.1.107 80 {
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

virtual_server 192.168.1.169 80 {
  .......                           # 配置同上
}  
```


## 4. 单主模型的nginx高可用集群
nginx 服务的高可用，我们需要使用  `vrrp_script{}` 定义 nginx 的检测方式，并将这种检测通过 `track_script` 添加到 vrrp 实例中去，让 vrrp 能够在检测到 nginx 服务异常时进行主备服务器切换。

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

vrrp_script chk_nginx {                            # 定义
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

## 5. 双主模型的nginx高可用集群
双主模型在单主模型基础上添加一个 vrrp 实例即可
