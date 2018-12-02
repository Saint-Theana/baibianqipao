# baibianqipao
#百变气泡
#!/usr/bin/bash

ids="11 12 46 60 63 110 114 398"

qrcodepath="/sdcard/1.png"

cookiepath="/data/data/io.neoterm/files/home/1.cookie"

hash33() {

e=0;

s=$1;

n=${#s};

i=0

while [[ $i -lt $n ]];do

g=$((e *32 &2147483647))

j="${s:i:1}"

o=$(printf "%d" \'$j)

e=$((e+g+o &2147483647))

((i=i+1))

done

echo "$e"

}

g_tk() {

e=5381;

s=$1;

n=${#s};

i=0

while [[ $i -lt $n ]];do

g=$((e *32 &2147483647))

j="${s:i:1}"

o=$(printf "%d" \'$j)

e=$((e+g+o &2147483647))

((i=i+1))

done

echo "$e"

}

get_qrcode(){

curl -s -L -k  "https://ui.ptlogin2.qq.com/cgi-bin/login?daid=164&target=self&style=16&mibao_css=m_webqq&appid=501004106&enable_qlogin=0&no_verifyimg=1&s_url=http%3A%2F%2Fw.qq.com%2Fproxy.html&f_url=loginerroralert&strong_login=1&login_state=10&t=20131024001" -c $cookiepath >/dev/null

curl -s -L -k "https://ssl.ptlogin2.qq.com/ptqrshow?appid=501004106&e=0&l=M&s=5&d=72&v=4&t=0.9142399367333609" -H "Referer: https://ui.ptlogin2.qq.com/cgi-bin/login" -b $cookiepath -c $cookiepath >$qrcodepath

qrsig="$(cat 1.cookie|grep -Po "qrsig[\t]*.*"|sed 's/qrsig\t//g')"

ptqrtoken="$(hash33 "$qrsig")"

}

main(){

get_qrcode

                

while true;do

Message=$(curl -s -L -k "https://ssl.ptlogin2.qq.com/ptqrlogin?ptqrtoken=$ptqrtoken&webqq_type=10&remember_uin=1&login2qq=1&aid=501004106&u1=http%3A%2F%2Fw.qq.com%2Fproxy.html%3Flogin2qq%3D1%26webqq_type%3D10&ptredirect=0&ptlang=2052&daid=164&from_ui=1&pttype=1&dumy=&fp=loginerroralert&action=0-0-1283996.7948407694&mibao_css=m_webqq&t=undefined&g=1&js_type=0&js_ver=10141&login_sig=&pt_randsalt=0" -H "Referer: https://ui.ptlogin2.qq.com/cgi-bin/login?daid=164&target=self&style=16&mibao_css=m_webqq&appid=501004106&enable_qlogin=0&no_verifyimg=1&s_url=http%3A%2F%2Fw.qq.com%2Fproxy.html&f_url=loginerroralert&strong_login=1&login_state=10&t=20131024001" -b $cookiepath -c $cookiepath)

if [[ $Message =~ "登录成功" ]];then

    echo "已经扫码，正在登录"

    verify_url=$(echo $Message|tr "'," "\n"|grep -Po "http:.*")

    break

fi

if [[ $Message =~ "已失效" ]];then

    get_qrcode

    echo "重新获取二维码"

fi

echo "等待扫码登陆"

sleep 2

done

curl -s -k "$verify_url" -b $cookiepath -c $cookiepath

ptwebqq="$(cat 1.cookie|grep -Po "ptwebqq[\t]*.*"|sed 's/ptwebqq\t//g')"

Message=$(curl -s -k "http://s.web2.qq.com/api/getvfwebqq?ptwebqq=$ptwebqq&clientid=53999199&psessionid=&t=1488053293431" -H "Referer: http://s.web2.qq.com/proxy.html?v=20130916001&callback=1&id=1" -b $cookiepath -c $cookiepath)

vfwebqq=$(echo $Message|tr "{}" "\n"|grep -Po "vfwebqq:.*"|sed 's/vfwebqq://g')

data="r={\"ptwebqq\": \"$ptwebqq\", \"clientid\": 53999199, \"psessionid\": \"\", \"status\": \"online\"}"

Message=$(curl -L -s -k "http://d1.web2.qq.com/channel/login2" -H "Referer: http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2" -H "Origin: http://d1.web2.qq.com" -d "$data" -b $cookiepath -c $cookiepath)

qqnumber=$(echo $Message|tr ",{}" "\n"|grep -Po "uin\":.*"|sed 's/uin\"://g'|tr -d "\"")

psessionid=$(echo $Message|tr ",{}" "\n"|grep -Po "psessionid\":\".*"|sed 's/psessionid\":\"//g'|tr -d "\"")

vfwebqq=$(echo $Message|tr ",{}" "\n"|grep -Po "vfwebqq\":\".*"|sed 's/vfwebqq\":\"//g'|tr -d "\"")

curl -s -L -k "http://d1.web2.qq.com/channel/get_online_buddies2?vfwebqq=$vfwebqq&clientid=53999199&psessionid=$psessionid&t=1488268527333" -b $cookiepath -c $cookiepath -H "Host: d1.web2.qq.com" -H "Referer: http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2" >/dev/null

data="r={\"ptwebqq\": \"$ptwebqq\", \"clientid\": 53999199, \"psessionid\": \"$psessionid\", \"status\": \"online\"}"

while true;do

curl -s -L -k "http://d1.web2.qq.com/channel/poll2" -H "Referer: http://d1.web2.qq.com/proxy.html?v=20151105001&callback=1&id=2" -H "Origin: http://d1.web2.qq.com" -b $cookiepath -d "$data" >/dev/null

sleep 5

done &

skey="$(cat 1.cookie|grep -v p_skey|grep -Po "[\t]*skey[\t]*.*"|sed 's/skey\t//g'|tr -d "\t")"

gtk=$(g_tk "$skey")

while true;do

time=$(date +%s)000

for id in $ids;do

Message=$(curl -s -L -k -A "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0; BOIE9;ZHCN)" "http://logic.content.qq.com/bubble/bubbleSetup?id=$id&platformId=3&uin=$qqnumber&sid=&format=jsonp&t=$time&&g_tk=$gtk&callback=jsonp2" -H "Referer: http://logic.content.qq.com/bubble/bubbleSetup?id=$id&platformId=3&uin=$qqnumber&sid=&format=jsonp&t=$time&&g_tk=$gtk&callback=jsonp2" -H "Content-Type: application/x-www-form-urlencoded" -H "Connection: Keep-Alive" -b $cookiepath)

if [[ $Message =~ "ok" ]];then

echo "成功:   id:$id";fi

sleep 5

done

done &

}

main

