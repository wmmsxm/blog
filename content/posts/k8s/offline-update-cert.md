---
title: "K8s离线部署-更新过期证书"
date: 2022-12-30T14:21:52+08:00
draft: false
tags: ["k8s"]
categories: ["k8s"]
---
## <center>K8s离线部署-更新过期证书</center>
#### 查看证书有效期
```
执行下面命令，可以看到apiserver.crt的有效期时间
# openssl x509 -in /etc/kubernetes/pki/apiserver.crt -noout -text  |grep Not
    Not Before: Dec  8 02:26:37 2021 GMT
    Not After : Dec  8 02:26:37 2022 GMT
```
K8S CA证书是10年，但是组件证书的日期只有1年，为了证书一直可用状态需要更新，需要执行下面步骤进行更新


#### 执行脚本
**如果使用 `containerd` 作为 CRI 运行时，使用 `update-kubeadm-cert-crictl.sh` 代替 `update-kubeadm-cert.sh`**
**如果有多个 master 节点，在每个 master 节点都执行一次**
**该脚本仅需要在 master 节点执行，无需在 worker node 节点执行**
```
1、创建update-kubeadm-cert.sh文件，内容在下面，自行填充。
2、执行授权命令：    chmod 755 update-kubeadm-cert.sh
3、执行脚本    ./update-kubeadm-cert.sh all
```
输出类似信息：
```
CERTIFICATE                                       EXPIRES
/etc/kubernetes/controller-manager.config         Sep 12 08:38:56 2022 GMT
/etc/kubernetes/scheduler.config                  Sep 12 08:38:56 2022 GMT
/etc/kubernetes/admin.config                      Sep 12 08:38:56 2022 GMT
/etc/kubernetes/pki/ca.crt                        Sep 11 08:38:53 2031 GMT
/etc/kubernetes/pki/apiserver.crt                 Sep 12 08:38:54 2022 GMT
/etc/kubernetes/pki/apiserver-kubelet-client.crt  Sep 12 08:38:54 2022 GMT
/etc/kubernetes/pki/front-proxy-ca.crt            Sep 11 08:38:54 2031 GMT
/etc/kubernetes/pki/front-proxy-client.crt        Sep 12 08:38:54 2022 GMT
/etc/kubernetes/pki/etcd/ca.crt                   Sep 11 08:38:55 2031 GMT
/etc/kubernetes/pki/etcd/server.crt               Sep 12 08:38:55 2022 GMT
/etc/kubernetes/pki/etcd/peer.crt                 Sep 12 08:38:55 2022 GMT
/etc/kubernetes/pki/etcd/healthcheck-client.crt   Sep 12 08:38:55 2022 GMT
/etc/kubernetes/pki/apiserver-etcd-client.crt     Sep 12 08:38:56 2022 GMT
[2021-09-12T16:41:25.93+0800][INFO] backup /etc/kubernetes to /etc/kubernetes.old-20210912
[2021-09-12T16:41:25.93+0800][INFO] updating...
[2021-09-12T16:41:25.99+0800][INFO] updated /etc/kubernetes/pki/etcd/server.conf
[2021-09-12T16:41:26.04+0800][INFO] updated /etc/kubernetes/pki/etcd/peer.conf
[2021-09-12T16:41:26.07+0800][INFO] updated /etc/kubernetes/pki/etcd/healthcheck-client.conf
[2021-09-12T16:41:26.11+0800][INFO] updated /etc/kubernetes/pki/apiserver-etcd-client.conf
[2021-09-12T16:41:26.54+0800][INFO] restarted etcd
[2021-09-12T16:41:26.60+0800][INFO] updated /etc/kubernetes/pki/apiserver.crt
[2021-09-12T16:41:26.64+0800][INFO] updated /etc/kubernetes/pki/apiserver-kubelet-client.crt
[2021-09-12T16:41:26.69+0800][INFO] updated /etc/kubernetes/controller-manager.conf
[2021-09-12T16:41:26.74+0800][INFO] updated /etc/kubernetes/scheduler.conf
[2021-09-12T16:41:26.79+0800][INFO] updated /etc/kubernetes/admin.conf
[2021-09-12T16:41:26.79+0800][INFO] backup /root/.kube/config to /root/.kube/config.old-20210912
[2021-09-12T16:41:26.80+0800][INFO] copy the admin.conf to /root/.kube/config
[2021-09-12T16:41:26.85+0800][INFO] updated /etc/kubernetes/kubelet.conf
[2021-09-12T16:41:26.88+0800][INFO] updated /etc/kubernetes/pki/front-proxy-client.crt
[2021-09-12T16:41:28.70+0800][INFO] restarted apiserver
[2021-09-12T16:41:29.17+0800][INFO] restarted controller-manager
[2021-09-12T16:41:30.07+0800][INFO] restarted scheduler
[2021-09-12T16:41:30.13+0800][INFO] restarted kubelet
[2021-09-12T16:41:30.14+0800][INFO] done!!!
CERTIFICATE                                       EXPIRES
/etc/kubernetes/controller-manager.config         Sep 11 08:41:26 2031 GMT
/etc/kubernetes/scheduler.config                  Sep 11 08:41:26 2031 GMT
/etc/kubernetes/admin.config                      Sep 11 08:41:26 2031 GMT
/etc/kubernetes/pki/ca.crt                        Sep 11 08:38:53 2031 GMT
/etc/kubernetes/pki/apiserver.crt                 Sep 11 08:41:26 2031 GMT
/etc/kubernetes/pki/apiserver-kubelet-client.crt  Sep 11 08:41:26 2031 GMT
/etc/kubernetes/pki/front-proxy-ca.crt            Sep 11 08:38:54 2031 GMT
/etc/kubernetes/pki/front-proxy-client.crt        Sep 11 08:41:26 2031 GMT
/etc/kubernetes/pki/etcd/ca.crt                   Sep 11 08:38:55 2031 GMT
/etc/kubernetes/pki/etcd/server.crt               Sep 11 08:41:25 2031 GMT
/etc/kubernetes/pki/etcd/peer.crt                 Sep 11 08:41:26 2031 GMT
/etc/kubernetes/pki/etcd/healthcheck-client.crt   Sep 11 08:41:26 2031 GMT
/etc/kubernetes/pki/apiserver-etcd-client.crt     Sep 11 08:41:26 2031 GMT
```

将更新以下证书和 kubeconfig 配置文件

```
/etc/kubernetes
├── admin.conf
├── controller-manager.conf
├── scheduler.conf
├── kubelet.conf
└── pki
    ├── apiserver.crt
    ├── apiserver-etcd-client.crt
    ├── apiserver-kubelet-client.crt
    ├── front-proxy-client.crt
    └── etcd
        ├── healthcheck-client.crt
        ├── peer.crt
        └── server.crt
```

#### 证书更新失败回滚

脚本会自动备份 `/etc/kubernetes` 目录到 `/etc/kubernetes.old-$(date +%Y%m%d)` 目录（备份目录命名示例：`kubernetes.old-20221230`）

若更新证书失败需要回滚，手动将备份 `/etc/kubernetes.old-$(date +%Y%m%d)`目录覆盖 `/etc/kubernetes` 目录


#### update-kubeadm-cert.sh 脚本
```
#!/usr/bin/env bash

set -o errexit
set -o pipefail
# set -o xtrace

# set output color
NC='\033[0m'
RED='\033[31m'
GREEN='\033[32m'
YELLOW='\033[33m'
BLUE='\033[34m'

log::err() {
  printf "[$(date +'%Y-%m-%dT%H:%M:%S.%2N%z')][${RED}ERROR${NC}] %b\n" "$@"
}

log::info() {
  printf "[$(date +'%Y-%m-%dT%H:%M:%S.%2N%z')][INFO] %b\n" "$@"
}

log::warning() {
  printf "[$(date +'%Y-%m-%dT%H:%M:%S.%2N%z')][${YELLOW}WARNING${NC}] \033[0m%b\n" "$@"
}

check_file() {
  if [[ ! -r ${1} ]]; then
    log::err "can not find ${1}"
    exit 1
  fi
}

# get x509v3 subject alternative name from the old certificate
cert::get_subject_alt_name() {
  local cert=${1}.crt
  local alt_name

  check_file "${cert}"
  alt_name=$(openssl x509 -text -noout -in "${cert}" | grep -A1 'Alternative' | tail -n1 | sed 's/[[:space:]]*Address//g')
  printf "%s\n" "${alt_name}"
}

# get subject from the old certificate
cert::get_subj() {
  local cert=${1}.crt
  local subj

  check_file "${cert}"
  subj=$(openssl x509 -text -noout -in "${cert}" | grep "Subject:" | sed 's/Subject:/\//g;s/\,/\//;s/[[:space:]]//g')
  printf "%s\n" "${subj}"
}

cert::backup_file() {
  local file=${1}
  if [[ ! -e ${file}.old-$(date +%Y%m%d) ]]; then
    cp -rp "${file}" "${file}.old-$(date +%Y%m%d)"
    log::info "backup ${file} to ${file}.old-$(date +%Y%m%d)"
  else
    log::warning "does not backup, ${file}.old-$(date +%Y%m%d) already exists"
  fi
}

# check certificate expiration
cert::check_cert_expiration() {
  local cert=${1}.crt
  local cert_expires

  cert_expires=$(openssl x509 -text -noout -in "${cert}" | awk -F ": " '/Not After/{print$2}')
  printf "%s\n" "${cert_expires}"
}

# check kubeconfig expiration
cert::check_kubeconfig_expiration() {
  local config=${1}.conf
  local cert
  local cert_expires

  cert=$(grep "client-certificate-data" "${config}" | awk '{print$2}' | base64 -d)
  cert_expires=$(openssl x509 -text -noout -in <(printf "%s" "${cert}") | awk -F ": " '/Not After/{print$2}')
  printf "%s\n" "${cert_expires}"
}

# check etcd certificates expiration
cert::check_etcd_certs_expiration() {
  local cert
  local certs

  certs=(
    "${ETCD_CERT_CA}"
    "${ETCD_CERT_SERVER}"
    "${ETCD_CERT_PEER}"
    "${ETCD_CERT_HEALTHCHECK_CLIENT}"
    "${ETCD_CERT_APISERVER_ETCD_CLIENT}"
  )

  for cert in "${certs[@]}"; do
    if [[ ! -r ${cert} ]]; then
      printf "%-50s%-30s\n" "${cert}.crt" "$(cert::check_cert_expiration "${cert}")"
    fi
  done
}

# check master certificates expiration
cert::check_master_certs_expiration() {
  local certs
  local kubeconfs
  local cert
  local conf

  certs=(
    "${CERT_CA}"
    "${CERT_APISERVER}"
    "${CERT_APISERVER_KUBELET_CLIENT}"
    "${FRONT_PROXY_CA}"
    "${FRONT_PROXY_CLIENT}"
  )

  kubeconfs=(
    "${CONF_CONTROLLER_MANAGER}"
    "${CONF_SCHEDULER}"
    "${CONF_ADMIN}"
  )

  printf "%-50s%-30s\n" "CERTIFICATE" "EXPIRES"

  for conf in "${kubeconfs[@]}"; do
    if [[ ! -r ${conf} ]]; then
      printf "%-50s%-30s\n" "${conf}.config" "$(cert::check_kubeconfig_expiration "${conf}")"
    fi
  done

  for cert in "${certs[@]}"; do
    if [[ ! -r ${cert} ]]; then
      printf "%-50s%-30s\n" "${cert}.crt" "$(cert::check_cert_expiration "${cert}")"
    fi
  done
}

# check all certificates expiration
cert::check_all_expiration() {
  cert::check_master_certs_expiration
  cert::check_etcd_certs_expiration
}

# generate certificate whit client, server or peer
# Args:
#   $1 (the name of certificate)
#   $2 (the type of certificate, must be one of client, server, peer)
#   $3 (the subject of certificates)
#   $4 (the validity of certificates) (days)
#   $5 (the name of ca)
#   $6 (the x509v3 subject alternative name of certificate when the type of certificate is server or peer)
cert::gen_cert() {
  local cert_name=${1}
  local cert_type=${2}
  local subj=${3}
  local cert_days=${4}
  local ca_name=${5}
  local alt_name=${6}
  local ca_cert=${ca_name}.crt
  local ca_key=${ca_name}.key
  local cert=${cert_name}.crt
  local key=${cert_name}.key
  local csr=${cert_name}.csr
  local common_csr_conf='distinguished_name = dn\n[dn]\n[v3_ext]\nkeyUsage = critical, digitalSignature, keyEncipherment\n'

  for file in "${ca_cert}" "${ca_key}" "${cert}" "${key}"; do
    check_file "${file}"
  done

  case "${cert_type}" in
  client)
    csr_conf=$(printf "%bextendedKeyUsage = clientAuth\n" "${common_csr_conf}")
    ;;
  server)
    csr_conf=$(printf "%bextendedKeyUsage = serverAuth\nsubjectAltName = %b\n" "${common_csr_conf}" "${alt_name}")
    ;;
  peer)
    csr_conf=$(printf "%bextendedKeyUsage = serverAuth, clientAuth\nsubjectAltName = %b\n" "${common_csr_conf}" "${alt_name}")
    ;;
  *)
    log::err "unknow, unsupported certs type: ${YELLOW}${cert_type}${NC}, supported type: client, server, peer"
    exit 1
    ;;
  esac

  # gen csr
  openssl req -new -key "${key}" -subj "${subj}" -reqexts v3_ext \
    -config <(printf "%b" "${csr_conf}") \
    -out "${csr}" >/dev/null 2>&1
  # gen cert
  openssl x509 -in "${csr}" -req -CA "${ca_cert}" -CAkey "${ca_key}" -CAcreateserial -extensions v3_ext \
    -extfile <(printf "%b" "${csr_conf}") \
    -days "${cert_days}" -out "${cert}" >/dev/null 2>&1

  rm -f "${csr}"
}

cert::update_kubeconf() {
  local cert_name=${1}
  local kubeconf_file=${cert_name}.conf
  local cert=${cert_name}.crt
  local key=${cert_name}.key
  local subj
  local cert_base64

  check_file "${kubeconf_file}"
  # get the key from the old kubeconf
  grep "client-key-data" "${kubeconf_file}" | awk '{print$2}' | base64 -d >"${key}"
  # get the old certificate from the old kubeconf
  grep "client-certificate-data" "${kubeconf_file}" | awk '{print$2}' | base64 -d >"${cert}"
  # get subject from the old certificate
  subj=$(cert::get_subj "${cert_name}")
  cert::gen_cert "${cert_name}" "client" "${subj}" "${CERT_DAYS}" "${CERT_CA}"
  # get certificate base64 code
  cert_base64=$(base64 -w 0 "${cert}")

  # set certificate base64 code to kubeconf
  sed -i 's/client-certificate-data:.*/client-certificate-data: '"${cert_base64}"'/g' "${kubeconf_file}"

  rm -f "${cert}"
  rm -f "${key}"
}

cert::update_etcd_cert() {
  local subj
  local subject_alt_name
  local cert

  # generate etcd server,peer certificate
  # /etc/kubernetes/pki/etcd/server
  # /etc/kubernetes/pki/etcd/peer
  for cert in ${ETCD_CERT_SERVER} ${ETCD_CERT_PEER}; do
    subj=$(cert::get_subj "${cert}")
    subject_alt_name=$(cert::get_subject_alt_name "${cert}")
    cert::gen_cert "${cert}" "peer" "${subj}" "${CERT_DAYS}" "${ETCD_CERT_CA}" "${subject_alt_name}"
    log::info "${GREEN}updated ${BLUE}${cert}.conf${NC}"
  done

  # generate etcd healthcheck-client,apiserver-etcd-client certificate
  # /etc/kubernetes/pki/etcd/healthcheck-client
  # /etc/kubernetes/pki/apiserver-etcd-client
  for cert in ${ETCD_CERT_HEALTHCHECK_CLIENT} ${ETCD_CERT_APISERVER_ETCD_CLIENT}; do
    subj=$(cert::get_subj "${cert}")
    cert::gen_cert "${cert}" "client" "${subj}" "${CERT_DAYS}" "${ETCD_CERT_CA}"
    log::info "${GREEN}updated ${BLUE}${cert}.conf${NC}"
  done

  # restart etcd
  docker ps | awk '/k8s_etcd/{print$1}' | xargs -r -I '{}' docker restart {} >/dev/null 2>&1 || true
  log::info "restarted etcd"
}

cert::update_master_cert() {
  local subj
  local subject_alt_name
  local conf

  # generate apiserver server certificate
  # /etc/kubernetes/pki/apiserver
  subj=$(cert::get_subj "${CERT_APISERVER}")
  subject_alt_name=$(cert::get_subject_alt_name "${CERT_APISERVER}")
  cert::gen_cert "${CERT_APISERVER}" "server" "${subj}" "${CERT_DAYS}" "${CERT_CA}" "${subject_alt_name}"
  log::info "${GREEN}updated ${BLUE}${CERT_APISERVER}.crt${NC}"

  # generate apiserver-kubelet-client certificate
  # /etc/kubernetes/pki/apiserver-kubelet-client
  subj=$(cert::get_subj "${CERT_APISERVER_KUBELET_CLIENT}")
  cert::gen_cert "${CERT_APISERVER_KUBELET_CLIENT}" "client" "${subj}" "${CERT_DAYS}" "${CERT_CA}"
  log::info "${GREEN}updated ${BLUE}${CERT_APISERVER_KUBELET_CLIENT}.crt${NC}"

  # generate kubeconf for controller-manager,scheduler and kubelet
  # /etc/kubernetes/controller-manager,scheduler,admin,kubelet.conf
  for conf in ${CONF_CONTROLLER_MANAGER} ${CONF_SCHEDULER} ${CONF_ADMIN} ${CONF_KUBELET}; do
    if [[ ${conf##*/} == "kubelet" ]]; then
      # https://github.com/kubernetes/kubeadm/issues/1753
      set +e
      grep kubelet-client-current.pem /etc/kubernetes/kubelet.conf >/dev/null 2>&1
      kubelet_cert_auto_update=$?
      set -e
      if [[ "$kubelet_cert_auto_update" == "0" ]]; then
        log::info "does not need to update kubelet.conf"
        continue
      fi
    fi

    # update kubeconf
    cert::update_kubeconf "${conf}"
    log::info "${GREEN}updated ${BLUE}${conf}.conf${NC}"

    # copy admin.conf to ${HOME}/.kube/config
    if [[ ${conf##*/} == "admin" ]]; then
      mkdir -p "${HOME}/.kube"
      local config=${HOME}/.kube/config
      local config_backup
      config_backup=${HOME}/.kube/config.old-$(date +%Y%m%d)
      if [[ -f ${config} ]] && [[ ! -f ${config_backup} ]]; then
        cp -fp "${config}" "${config_backup}"
        log::info "backup ${config} to ${config_backup}"
      fi
      cp -fp "${conf}.conf" "${HOME}/.kube/config"
      log::info "copy the admin.conf to ${HOME}/.kube/config"
    fi
  done

  # generate front-proxy-client certificate
  # /etc/kubernetes/pki/front-proxy-client
  subj=$(cert::get_subj "${FRONT_PROXY_CLIENT}")
  cert::gen_cert "${FRONT_PROXY_CLIENT}" "client" "${subj}" "${CERT_DAYS}" "${FRONT_PROXY_CA}"
  log::info "${GREEN}updated ${BLUE}${FRONT_PROXY_CLIENT}.crt${NC}"

  # restart apiserver, controller-manager, scheduler and kubelet
  for item in "apiserver" "controller-manager" "scheduler"; do
    docker ps | awk '/k8s_kube-'${item}'/{print$1}' | xargs -r -I '{}' docker restart {} >/dev/null 2>&1 || true
    log::info "restarted ${item}"
  done
  systemctl restart kubelet || true
  log::info "restarted kubelet"
}

main() {
  local node_type=$1

  CERT_DAYS=3650

  KUBE_PATH=/etc/kubernetes
  PKI_PATH=${KUBE_PATH}/pki

  # master certificates path
  # apiserver
  CERT_CA=${PKI_PATH}/ca
  CERT_APISERVER=${PKI_PATH}/apiserver
  CERT_APISERVER_KUBELET_CLIENT=${PKI_PATH}/apiserver-kubelet-client
  CONF_CONTROLLER_MANAGER=${KUBE_PATH}/controller-manager
  CONF_SCHEDULER=${KUBE_PATH}/scheduler
  CONF_ADMIN=${KUBE_PATH}/admin
  CONF_KUBELET=${KUBE_PATH}/kubelet
  # front-proxy
  FRONT_PROXY_CA=${PKI_PATH}/front-proxy-ca
  FRONT_PROXY_CLIENT=${PKI_PATH}/front-proxy-client

  # etcd certificates path
  ETCD_CERT_CA=${PKI_PATH}/etcd/ca
  ETCD_CERT_SERVER=${PKI_PATH}/etcd/server
  ETCD_CERT_PEER=${PKI_PATH}/etcd/peer
  ETCD_CERT_HEALTHCHECK_CLIENT=${PKI_PATH}/etcd/healthcheck-client
  ETCD_CERT_APISERVER_ETCD_CLIENT=${PKI_PATH}/apiserver-etcd-client

  case ${node_type} in
  # etcd)
  # # update etcd certificates
  #   cert::update_etcd_cert
  # ;;
  master)
    # check certificates expiration
    cert::check_master_certs_expiration
    # backup $KUBE_PATH to $KUBE_PATH.old-$(date +%Y%m%d)
    cert::backup_file "${KUBE_PATH}"
    # update master certificates and kubeconf
    log::info "${GREEN}updating...${NC}"
    cert::update_master_cert
    log::info "${GREEN}done!!!${NC}"
    # check certificates expiration after certificates updated
    cert::check_master_certs_expiration
    ;;
  all)
    # check certificates expiration
    cert::check_all_expiration
    # backup $KUBE_PATH to $KUBE_PATH.old-$(date +%Y%m%d)
    cert::backup_file "${KUBE_PATH}"
    # update etcd certificates
    log::info "${GREEN}updating...${NC}"
    cert::update_etcd_cert
    # update master certificates and kubeconf
    cert::update_master_cert
    log::info "${GREEN}done!!!${NC}"
    # check certificates expiration after certificates updated
    cert::check_all_expiration
    ;;
  check)
    # check certificates expiration
    cert::check_all_expiration
    ;;
  *)
    log::err "unknown, unsupported cert type: ${node_type}, supported type: \"all\", \"master\""
    printf "Documentation: https://github.com/yuyicai/update-kube-cert
  example:
    '\033[32m./update-kubeadm-cert.sh all\033[0m' update all etcd certificates, master certificates and kubeconf
      /etc/kubernetes
      ├── admin.conf
      ├── controller-manager.conf
      ├── scheduler.conf
      ├── kubelet.conf
      └── pki
          ├── apiserver.crt
          ├── apiserver-etcd-client.crt
          ├── apiserver-kubelet-client.crt
          ├── front-proxy-client.crt
          └── etcd
              ├── healthcheck-client.crt
              ├── peer.crt
              └── server.crt

    '\033[32m./update-kubeadm-cert.sh master\033[0m' update only master certificates and kubeconf
      /etc/kubernetes
      ├── admin.conf
      ├── controller-manager.conf
      ├── scheduler.conf
      ├── kubelet.conf
      └── pki
          ├── apiserver.crt
          ├── apiserver-kubelet-client.crt
          └── front-proxy-client.crt
"
    exit 1
    ;;
  esac
}

main "$@"


```

#### update-kubeadm-cert-crictl.sh脚本
```
#!/usr/bin/env bash

set -o errexit
set -o pipefail
# set -o xtrace

# set output color
NC='\033[0m'
RED='\033[31m'
GREEN='\033[32m'
YELLOW='\033[33m'
BLUE='\033[34m'

log::err() {
  printf "[$(date +'%Y-%m-%dT%H:%M:%S.%2N%z')][${RED}ERROR${NC}] %b\n" "$@"
}

log::info() {
  printf "[$(date +'%Y-%m-%dT%H:%M:%S.%2N%z')][INFO] %b\n" "$@"
}

log::warning() {
  printf "[$(date +'%Y-%m-%dT%H:%M:%S.%2N%z')][${YELLOW}WARNING${NC}] \033[0m%b\n" "$@"
}

check_file() {
  if [[ ! -r ${1} ]]; then
    log::err "can not find ${1}"
    exit 1
  fi
}

# get x509v3 subject alternative name from the old certificate
cert::get_subject_alt_name() {
  local cert=${1}.crt
  local alt_name

  check_file "${cert}"
  alt_name=$(openssl x509 -text -noout -in "${cert}" | grep -A1 'Alternative' | tail -n1 | sed 's/[[:space:]]*Address//g')
  printf "%s\n" "${alt_name}"
}

# get subject from the old certificate
cert::get_subj() {
  local cert=${1}.crt
  local subj

  check_file "${cert}"
  subj=$(openssl x509 -text -noout -in "${cert}" | grep "Subject:" | sed 's/Subject:/\//g;s/\,/\//;s/[[:space:]]//g')
  printf "%s\n" "${subj}"
}

cert::backup_file() {
  local file=${1}
  if [[ ! -e ${file}.old-$(date +%Y%m%d) ]]; then
    cp -rp "${file}" "${file}.old-$(date +%Y%m%d)"
    log::info "backup ${file} to ${file}.old-$(date +%Y%m%d)"
  else
    log::warning "does not backup, ${file}.old-$(date +%Y%m%d) already exists"
  fi
}

# check certificate expiration
cert::check_cert_expiration() {
  local cert=${1}.crt
  local cert_expires

  cert_expires=$(openssl x509 -text -noout -in "${cert}" | awk -F ": " '/Not After/{print$2}')
  printf "%s\n" "${cert_expires}"
}

# check kubeconfig expiration
cert::check_kubeconfig_expiration() {
  local config=${1}.conf
  local cert
  local cert_expires

  cert=$(grep "client-certificate-data" "${config}" | awk '{print$2}' | base64 -d)
  cert_expires=$(openssl x509 -text -noout -in <(printf "%s" "${cert}") | awk -F ": " '/Not After/{print$2}')
  printf "%s\n" "${cert_expires}"
}

# check etcd certificates expiration
cert::check_etcd_certs_expiration() {
  local cert
  local certs

  certs=(
    "${ETCD_CERT_CA}"
    "${ETCD_CERT_SERVER}"
    "${ETCD_CERT_PEER}"
    "${ETCD_CERT_HEALTHCHECK_CLIENT}"
    "${ETCD_CERT_APISERVER_ETCD_CLIENT}"
  )

  for cert in "${certs[@]}"; do
    if [[ ! -r ${cert} ]]; then
      printf "%-50s%-30s\n" "${cert}.crt" "$(cert::check_cert_expiration "${cert}")"
    fi
  done
}

# check master certificates expiration
cert::check_master_certs_expiration() {
  local certs
  local kubeconfs
  local cert
  local conf

  certs=(
    "${CERT_CA}"
    "${CERT_APISERVER}"
    "${CERT_APISERVER_KUBELET_CLIENT}"
    "${FRONT_PROXY_CA}"
    "${FRONT_PROXY_CLIENT}"
  )

  kubeconfs=(
    "${CONF_CONTROLLER_MANAGER}"
    "${CONF_SCHEDULER}"
    "${CONF_ADMIN}"
  )

  printf "%-50s%-30s\n" "CERTIFICATE" "EXPIRES"

  for conf in "${kubeconfs[@]}"; do
    if [[ ! -r ${conf} ]]; then
      printf "%-50s%-30s\n" "${conf}.config" "$(cert::check_kubeconfig_expiration "${conf}")"
    fi
  done

  for cert in "${certs[@]}"; do
    if [[ ! -r ${cert} ]]; then
      printf "%-50s%-30s\n" "${cert}.crt" "$(cert::check_cert_expiration "${cert}")"
    fi
  done
}

# check all certificates expiration
cert::check_all_expiration() {
  cert::check_master_certs_expiration
  cert::check_etcd_certs_expiration
}

# generate certificate whit client, server or peer
# Args:
#   $1 (the name of certificate)
#   $2 (the type of certificate, must be one of client, server, peer)
#   $3 (the subject of certificates)
#   $4 (the validity of certificates) (days)
#   $5 (the name of ca)
#   $6 (the x509v3 subject alternative name of certificate when the type of certificate is server or peer)
cert::gen_cert() {
  local cert_name=${1}
  local cert_type=${2}
  local subj=${3}
  local cert_days=${4}
  local ca_name=${5}
  local alt_name=${6}
  local ca_cert=${ca_name}.crt
  local ca_key=${ca_name}.key
  local cert=${cert_name}.crt
  local key=${cert_name}.key
  local csr=${cert_name}.csr
  local common_csr_conf='distinguished_name = dn\n[dn]\n[v3_ext]\nkeyUsage = critical, digitalSignature, keyEncipherment\n'

  for file in "${ca_cert}" "${ca_key}" "${cert}" "${key}"; do
    check_file "${file}"
  done

  case "${cert_type}" in
  client)
    csr_conf=$(printf "%bextendedKeyUsage = clientAuth\n" "${common_csr_conf}")
    ;;
  server)
    csr_conf=$(printf "%bextendedKeyUsage = serverAuth\nsubjectAltName = %b\n" "${common_csr_conf}" "${alt_name}")
    ;;
  peer)
    csr_conf=$(printf "%bextendedKeyUsage = serverAuth, clientAuth\nsubjectAltName = %b\n" "${common_csr_conf}" "${alt_name}")
    ;;
  *)
    log::err "unknow, unsupported certs type: ${YELLOW}${cert_type}${NC}, supported type: client, server, peer"
    exit 1
    ;;
  esac

  # gen csr
  openssl req -new -key "${key}" -subj "${subj}" -reqexts v3_ext \
    -config <(printf "%b" "${csr_conf}") \
    -out "${csr}" >/dev/null 2>&1
  # gen cert
  openssl x509 -in "${csr}" -req -CA "${ca_cert}" -CAkey "${ca_key}" -CAcreateserial -extensions v3_ext \
    -extfile <(printf "%b" "${csr_conf}") \
    -days "${cert_days}" -out "${cert}" >/dev/null 2>&1

  rm -f "${csr}"
}

cert::update_kubeconf() {
  local cert_name=${1}
  local kubeconf_file=${cert_name}.conf
  local cert=${cert_name}.crt
  local key=${cert_name}.key
  local subj
  local cert_base64

  check_file "${kubeconf_file}"
  # get the key from the old kubeconf
  grep "client-key-data" "${kubeconf_file}" | awk '{print$2}' | base64 -d >"${key}"
  # get the old certificate from the old kubeconf
  grep "client-certificate-data" "${kubeconf_file}" | awk '{print$2}' | base64 -d >"${cert}"
  # get subject from the old certificate
  subj=$(cert::get_subj "${cert_name}")
  cert::gen_cert "${cert_name}" "client" "${subj}" "${CERT_DAYS}" "${CERT_CA}"
  # get certificate base64 code
  cert_base64=$(base64 -w 0 "${cert}")

  # set certificate base64 code to kubeconf
  sed -i 's/client-certificate-data:.*/client-certificate-data: '"${cert_base64}"'/g' "${kubeconf_file}"

  rm -f "${cert}"
  rm -f "${key}"
}

cert::update_etcd_cert() {
  local subj
  local subject_alt_name
  local cert

  # generate etcd server,peer certificate
  # /etc/kubernetes/pki/etcd/server
  # /etc/kubernetes/pki/etcd/peer
  for cert in ${ETCD_CERT_SERVER} ${ETCD_CERT_PEER}; do
    subj=$(cert::get_subj "${cert}")
    subject_alt_name=$(cert::get_subject_alt_name "${cert}")
    cert::gen_cert "${cert}" "peer" "${subj}" "${CERT_DAYS}" "${ETCD_CERT_CA}" "${subject_alt_name}"
    log::info "${GREEN}updated ${BLUE}${cert}.conf${NC}"
  done

  # generate etcd healthcheck-client,apiserver-etcd-client certificate
  # /etc/kubernetes/pki/etcd/healthcheck-client
  # /etc/kubernetes/pki/apiserver-etcd-client
  for cert in ${ETCD_CERT_HEALTHCHECK_CLIENT} ${ETCD_CERT_APISERVER_ETCD_CLIENT}; do
    subj=$(cert::get_subj "${cert}")
    cert::gen_cert "${cert}" "client" "${subj}" "${CERT_DAYS}" "${ETCD_CERT_CA}"
    log::info "${GREEN}updated ${BLUE}${cert}.conf${NC}"
  done

  # restart etcd
  #docker ps | awk '/k8s_etcd/{print$1}' | xargs -r -I '{}' docker restart {} >/dev/null 2>&1 || true
  crictl ps | awk '/etcd-/{print$(NF-1)}' |xargs -r -I '{}' crictl stopp {} >/dev/null 2>&1 || true
  log::info "restarted etcd"
}

cert::update_master_cert() {
  local subj
  local subject_alt_name
  local conf

  # generate apiserver server certificate
  # /etc/kubernetes/pki/apiserver
  subj=$(cert::get_subj "${CERT_APISERVER}")
  subject_alt_name=$(cert::get_subject_alt_name "${CERT_APISERVER}")
  cert::gen_cert "${CERT_APISERVER}" "server" "${subj}" "${CERT_DAYS}" "${CERT_CA}" "${subject_alt_name}"
  log::info "${GREEN}updated ${BLUE}${CERT_APISERVER}.crt${NC}"

  # generate apiserver-kubelet-client certificate
  # /etc/kubernetes/pki/apiserver-kubelet-client
  subj=$(cert::get_subj "${CERT_APISERVER_KUBELET_CLIENT}")
  cert::gen_cert "${CERT_APISERVER_KUBELET_CLIENT}" "client" "${subj}" "${CERT_DAYS}" "${CERT_CA}"
  log::info "${GREEN}updated ${BLUE}${CERT_APISERVER_KUBELET_CLIENT}.crt${NC}"

  # generate kubeconf for controller-manager,scheduler and kubelet
  # /etc/kubernetes/controller-manager,scheduler,admin,kubelet.conf
  for conf in ${CONF_CONTROLLER_MANAGER} ${CONF_SCHEDULER} ${CONF_ADMIN} ${CONF_KUBELET}; do
    if [[ ${conf##*/} == "kubelet" ]]; then
      # https://github.com/kubernetes/kubeadm/issues/1753
      set +e
      grep kubelet-client-current.pem /etc/kubernetes/kubelet.conf >/dev/null 2>&1
      kubelet_cert_auto_update=$?
      set -e
      if [[ "$kubelet_cert_auto_update" == "0" ]]; then
        log::info "does not need to update kubelet.conf"
        continue
      fi
    fi

    # update kubeconf
    cert::update_kubeconf "${conf}"
    log::info "${GREEN}updated ${BLUE}${conf}.conf${NC}"

    # copy admin.conf to ${HOME}/.kube/config
    if [[ ${conf##*/} == "admin" ]]; then
      mkdir -p "${HOME}/.kube"
      local config=${HOME}/.kube/config
      local config_backup
      config_backup=${HOME}/.kube/config.old-$(date +%Y%m%d)
      if [[ -f ${config} ]] && [[ ! -f ${config_backup} ]]; then
        cp -fp "${config}" "${config_backup}"
        log::info "backup ${config} to ${config_backup}"
      fi
      cp -fp "${conf}.conf" "${HOME}/.kube/config"
      log::info "copy the admin.conf to ${HOME}/.kube/config"
    fi
  done

  # generate front-proxy-client certificate
  # /etc/kubernetes/pki/front-proxy-client
  subj=$(cert::get_subj "${FRONT_PROXY_CLIENT}")
  cert::gen_cert "${FRONT_PROXY_CLIENT}" "client" "${subj}" "${CERT_DAYS}" "${FRONT_PROXY_CA}"
  log::info "${GREEN}updated ${BLUE}${FRONT_PROXY_CLIENT}.crt${NC}"

  # restart apiserver, controller-manager, scheduler and kubelet
  for item in "apiserver" "controller-manager" "scheduler"; do
    #docker ps | awk '/k8s_kube-'${item}'/{print$1}' | xargs -r -I '{}' docker restart {} >/dev/null 2>&1 || true
    crictl ps | awk '/kube-'${item}'-/{print $(NF-1)}' | xargs -r -I '{}' crictl stopp {} >/dev/null 2>&1 || true 
    log::info "restarted ${item}"
  done
  systemctl restart kubelet || true
  log::info "restarted kubelet"
}

main() {
  local node_type=$1

  CERT_DAYS=3650

  KUBE_PATH=/etc/kubernetes
  PKI_PATH=${KUBE_PATH}/pki

  # master certificates path
  # apiserver
  CERT_CA=${PKI_PATH}/ca
  CERT_APISERVER=${PKI_PATH}/apiserver
  CERT_APISERVER_KUBELET_CLIENT=${PKI_PATH}/apiserver-kubelet-client
  CONF_CONTROLLER_MANAGER=${KUBE_PATH}/controller-manager
  CONF_SCHEDULER=${KUBE_PATH}/scheduler
  CONF_ADMIN=${KUBE_PATH}/admin
  CONF_KUBELET=${KUBE_PATH}/kubelet
  # front-proxy
  FRONT_PROXY_CA=${PKI_PATH}/front-proxy-ca
  FRONT_PROXY_CLIENT=${PKI_PATH}/front-proxy-client

  # etcd certificates path
  ETCD_CERT_CA=${PKI_PATH}/etcd/ca
  ETCD_CERT_SERVER=${PKI_PATH}/etcd/server
  ETCD_CERT_PEER=${PKI_PATH}/etcd/peer
  ETCD_CERT_HEALTHCHECK_CLIENT=${PKI_PATH}/etcd/healthcheck-client
  ETCD_CERT_APISERVER_ETCD_CLIENT=${PKI_PATH}/apiserver-etcd-client

  case ${node_type} in
  # etcd)
  # # update etcd certificates
  #   cert::update_etcd_cert
  # ;;
  master)
    # check certificates expiration
    cert::check_master_certs_expiration
    # backup $KUBE_PATH to $KUBE_PATH.old-$(date +%Y%m%d)
    cert::backup_file "${KUBE_PATH}"
    # update master certificates and kubeconf
    log::info "${GREEN}updating...${NC}"
    cert::update_master_cert
    log::info "${GREEN}done!!!${NC}"
    # check certificates expiration after certificates updated
    cert::check_master_certs_expiration
    ;;
  all)
    # check certificates expiration
    cert::check_all_expiration
    # backup $KUBE_PATH to $KUBE_PATH.old-$(date +%Y%m%d)
    cert::backup_file "${KUBE_PATH}"
    # update etcd certificates
    log::info "${GREEN}updating...${NC}"
    cert::update_etcd_cert
    # update master certificates and kubeconf
    cert::update_master_cert
    log::info "${GREEN}done!!!${NC}"
    # check certificates expiration after certificates updated
    cert::check_all_expiration
    ;;
  check)
    # check certificates expiration
    cert::check_all_expiration
    ;;
  *)
    log::err "unknown, unsupported cert type: ${node_type}, supported type: \"all\", \"master\""
    printf "Documentation: https://github.com/yuyicai/update-kube-cert
  example:
    '\033[32m./update-kubeadm-cert.sh all\033[0m' update all etcd certificates, master certificates and kubeconf
      /etc/kubernetes
      ├── admin.conf
      ├── controller-manager.conf
      ├── scheduler.conf
      ├── kubelet.conf
      └── pki
          ├── apiserver.crt
          ├── apiserver-etcd-client.crt
          ├── apiserver-kubelet-client.crt
          ├── front-proxy-client.crt
          └── etcd
              ├── healthcheck-client.crt
              ├── peer.crt
              └── server.crt

    '\033[32m./update-kubeadm-cert.sh master\033[0m' update only master certificates and kubeconf
      /etc/kubernetes
      ├── admin.conf
      ├── controller-manager.conf
      ├── scheduler.conf
      ├── kubelet.conf
      └── pki
          ├── apiserver.crt
          ├── apiserver-kubelet-client.crt
          └── front-proxy-client.crt
"
    exit 1
    ;;
  esac
}

main "$@"


```