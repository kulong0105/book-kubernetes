# Introduction

Kubernetes作为业界容器云平台的分布式架构方案，跟我一起来学习

```bash
#!/bin/bash
ls
echo
df

manager_services()
{
    local nodes_info="$1"
    local services="$2"

    # https://unix.stackexchange.com/questions/336149/print-all-words-on-lines-of-a-file-in-reverse-order
    [[ $ACTION = "stop" ]] && services=$(echo $services | awk '{for(i=NF;i>=1;i--) printf "%s ", $i;print ""}')

    local node_info
    local service

    for node_info in $nodes_info; do
    ¦   for service in $services; do
    ¦   ¦   if $SSH $node_info "systemctl status $service 2>&1 | grep -q \"Unit $service.* could not be found\""; then
    ¦   ¦   ¦   log_warn "cannot find $service service in $node_info"
    ¦   ¦   ¦   continue
    ¦   ¦   fi

    ¦   ¦   if $SSH $node_info "timeout 300 systemctl $ACTION $service" &>/dev/null; then
    ¦   ¦   ¦   log_info "$service: $node_info: OK"
    ¦   ¦   else
    ¦   ¦   ¦   log_error "$service: $node_info: Failure"
    ¦   ¦   fi
    ¦   done
    done
}

export -f log
export -f log_info
export -f log_debug
export -f log_warn
export -f log_error
export -f check_files_exist
export -f parse_deploy_config
export -f check_dirs_exist
export -f get_ip_info
export -f check_ip_config
export -f check_passwd
export -f check_node_status
export -f parse_ip_info
export -f parse_app_info
export -f convert_to_ib
export -f parse_ib_info

[[ "$SCRIPT_NAME" ]] && ACTION="run-script-$(echo "$SCRIPT_PARAM" | tr -s -t " " "-")"
[[ "$COMMANDS" ]] && ACTION="run-command"
[[ -z "$ACTION" ]] && ACTION="unknown"

config_logfile || exit
check_user_id || exit
check_install_commands || exit
update_ssh_key || exit
parse_param || exit
parse_deploy_config || exit
load_service_info || exit

export SKYDISCOVERY_SRC
export SCOPE
export ACTION
export PSSH
export PSCP
export SSH
export SCP

case $ACTION in
    check)
    ¦   $DEBUG_PREFIX "${SKYDISCOVERY_SRC}"/bin/check "$OTHERS_PARAM" ;;
    install | uninstall | add | remove | upgrade | degrade)
    ¦   $DEBUG_PREFIX "${SKYDISCOVERY_SRC}"/bin/deploy ;;
    test)
    ¦   $DEBUG_PREFIX "${SKYDISCOVERY_SRC}"/bin/test "$OTHERS_PARAM" ;;
    monitor)
    ¦   $DEBUG_PREFIX "${SKYDISCOVERY_SRC}"/bin/monitor ;;
    status | is-active | stop | start | restart)
    ¦   $DEBUG_PREFIX "${SKYDISCOVERY_SRC}"/bin/manager "$OTHERS_PARAM" ;;
    init)
    ¦   $DEBUG_PREFIX "${SKYDISCOVERY_SRC}"/bin/init ;;
    *)
    ¦   if [[ "${COMMANDS}" ]]; then
    ¦   ¦   $DEBUG_PREFIX "${SKYDISCOVERY_SRC}"/bin/cmd "${COMMANDS}"
    ¦   elif [[ "${SCRIPT_NAME}" ]]; then
    ¦   ¦   if [[ -x "${SKYDISCOVERY_SRC}/bin/script/${SCRIPT_NAME}" ]]; then
    ¦   ¦   ¦   $DEBUG_PREFIX "${SKYDISCOVERY_SRC}"/bin/script/"${SCRIPT_NAME}" "${SCRIPT_PARAM}"
    ¦   ¦   else
    ¦   ¦   ¦   log_error "cannot find executable script ${SCRIPT_NAME} under ${SKYDISCOVERY_SRC}/bin/script"
    ¦   ¦   ¦   exit 1
    ¦   ¦   fi
    ¦   else
    ¦   ¦   log_error "cannot support \"$ACTION\" operation"
    ¦   ¦   usage
    ¦   fi ;;
esac
```

