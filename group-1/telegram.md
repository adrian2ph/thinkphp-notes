# telegram

## telegram bot



use this shell function to send message after create a bot from @BotFather.



```sh
#!/bin/bash

# telegram bot token
# @BotFather
BOT_TOKEN=""

# Replace 'YOUR_CHAT_ID' with the chat ID you want to send the message to
DEFAULT_CHAT_ID=""

# telegram机器人发消息到一个窗口
# eamxple: botSendMsg $msg $chatId
# botSendMsg "how are you" 1834657982 && echo 'send to person done'
# botSendMsg "how's it going" && echo 'send to group chat done'
function botSendMsg() {
    local MESSAGE="$1"
    local CHAT_ID=$DEFAULT_CHAT_ID
    if [ -n "$2" ]; then
        CHAT_ID="$2"
    fi

    # URL for sending messages via Telegram Bot API
    TELEGRAM_API="https://api.telegram.org/bot$BOT_TOKEN/sendMessage"
    # Sending a message to the specified chat ID
    RESULT=`curl -s -X POST "$TELEGRAM_API" -d chat_id="$CHAT_ID" -d text="$MESSAGE"`
    if [ -z "${RESULT##*\"ok\"\:true*}" ]; then
        return 0
    else
        return -1
    fi
}

function botUpdates() {
    local UPDATE_ID=0
    if [ -n "$1" ]; then
        UPDATE_ID="$1"
    fi

    # URL for getting updates from Telegram Bot API
    TELEGRAM_API="https://api.telegram.org/bot$BOT_TOKEN/getUpdates?offset=$UPDATE_ID"
    # Getting updates
    curl -s $TELEGRAM_API
}


##############################################
#Description: 监控cpu、磁盘、内存使用率
##############################################
function checkSysUsage(){

    #获取报警时间
    now_time=`date '+%F %T'`

    #获取cpu使用率
    cpuUsage=`top -b -n5 | fgrep "Cpu(s)" | tail -1 | awk -F'id,' '{split($1, vs, ","); v=vs[length(vs)]; sub(/\s+/, "", v);sub(/\s+/, "", v); printf "%d", 100-v;}'`
    #统计内存使用率
    mem_used_persent=`free -m | awk -F '[ :]+' 'NR==2{printf "%d", ($3)/$2*100}'`

    HOSTNAME=`hostname`

    function buildMsg() {
        cat <<EOF
hostname: ${HOSTNAME}
时间：${now_time}
CPU使用率：${cpuUsage}%
内存使用率：${mem_used_persent}%
EOF
    }

    MESSAGE=`buildMsg`
    if [[ "$cpuUsage" > 80 ]] || [[ "$diskUsage" > 80 ]] || [[ "$mem_used_persent" > 50 ]];then
        botSendMsg "报警信息 $MESSAGE"
    else
        botSendMsg "普通信息 $MESSAGE"
    fi
}


```

