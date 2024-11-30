git@github.com:shell-yan/starfishhungrybot.git

cd  starfishhungrybot 
Visual studio code -m venv venv 
source venv/bin/activate 
pip3 install -r req.txt 
vim bot.py 
python3 bot.py 
#!/usr/bin/python
# -*- coding:utf-8 -*-
 
import re
import json
import urllib2
 
from PlurkAPI import PlurkAPI
 
plurk = PlurkAPI('DDoPEkyGXNd2', 'WOSxyGxr1vQbmip6hNdmBSrU78NgW8hO')
plurk.authorize('avJGJBqdfD3P', 'v34XamJ3097vMTgZEPYYJUm2f5x3lDUU')

comet = plurk.callAPI('/APP/Realtime/getUserChannel')
comet_channel = comet.get('comet_server') + "&new_offset=%d"
jsonp_re = re.compile('CometChannel.scriptCallback\((.+)\);\s*');
new_offset = -1
while True:
    plurk.callAPI('/APP/Alerts/addAllAsFriends')
    req = urllib2.urlopen(comet_channel % new_offset, timeout=80)
    rawdata = req.read()
    match = jsonp_re.match(rawdata)
    if match:
        rawdata = match.group(1)
    data = json.loads(rawdata)
    new_offset = data.get('new_offset', -1)
    msgs = data.get('data')
    if not msgs:
        continue
    for msg in msgs:
        if msg.get('type') == 'new_plurk':
            pid = msg.get('plurk_id')
            content = msg.get('content_raw')
            if content.find("hello") != -1:
                plurk.callAPI('/APP/Responses/responseAdd',
                              {'plurk_id': pid,
                               'content': 'world',
                               'qualifier': ':' })

from typing import List, ValuesView
import org.bone.splurk.bot._
import org.bone.splurk.constant._
import scala.util.Random

import random
import requests
import re
import json
import urllib.request


import time
from plurk_oauth import PlurkAPI

# 以下請替換成自己的值！
friend_list = []
headers = {'x-api-key': '{$$.env.x-api-key}'}
plurk = PlurkAPI('APP_KEY', 'APP_SECRET')
plurk.authorize('ACCEESS_TOKEN', 'ACCESS_TOKEN_SECRET')

random_list = [ "今天過得開心嗎(p-rock)", "讚讚 [emo10]", "棒棒地[emo9]"]
comet = plurk.callAPI('/APP/Realtime/getUserChannel')
comet_channel = comet.get('comet_server') + "&new_offset=%d"
jsonp_re = re.compile('CometChannel.scriptCallback\((.+)\);\s*');
new_offset = -1

def auth():
    plurk.authorize('ACCEESS_TOKEN', 'ACCESS_TOKEN_SECRET')
    time.sleep(0.1)
    comet = plurk.callAPI('/APP/Realtime/getUserChannel')
    try:
        comet_channel = comet.get('comet_server') + "&new_offset=%d"
        jsonp_re = re.compile('CometChannel.scriptCallback\((.+)\);\s*');
    except Exception as e:
        print(f"[err]auth:{e}")


def setFriendList():
    try:
        data = plurk.callAPI('/APP/FriendsFans/getCompletion')
        if data is not None:
            for user in data:
                if not user in friend_list:
                    friend_list.append(user)
    except Exception as e:
        print(f"setFriendList err: {e}")

#回噗    
def plurkResponse(pid,content):
    plurk.callAPI('/APP/Responses/responseAdd', {'plurk_id': pid,
                                                 'content': content, 'qualifier': ':'})
def initApi():
    auth()
    plurk.callAPI('/APP/Alerts/addAllAsFriends')
    setFriendList()
    req = urllib.request.urlopen(comet_channel % new_offset, timeout=80)
    rawdata = req.read()
    match = jsonp_re.match(rawdata.decode('ISO-8859-1'))
    return match


def findTargetResponse(res_list, res_id):
    for res in res_list:
        if res['id'] == res_id:
            return res['content_raw']
    return "not found"


def responseMentioned():
    plurks = plurk.callAPI('/APP/Alerts/getActive')
    if plurks is not None:
        for pu in plurks:
            if pu is not None:
                if pu['type'] == "mentioned":
                    res_id = pu['response_id']
                    pid = pu['plurk_id']
                    res_json = plurk.callAPI('/APP/Responses/get', {'plurk_id': pid})
                    if res_json is None:
                        pass
                    else:
                        res_list = res_json['responses']
                        target = findTargetResponse(res_list, res_id)
                        dealContent(pid, target, True, pu, pu['from_user']['id'])


def dealContent(pid, content, isCmd, pu, user_id):
    print(f"reply plurk id:{pid} content:{content} pu: {pu}")
    if content.find("進村") != -1 or content.find("開村") != -1:
        plurkResponse(pid, ' 進村拉～')
    if content.find("鴨鴨") != -1:
        response = requests.get(duck_url)
        if response.status_code == 200:
            plurkResponse(pid, '呱呱！' + response.json()['url'])
    else:
        if content.find("謝謝") != -1:
            plurkResponse(pid, "不客氣拉！[emo9]")
        elif content.find("喜歡") != -1 or content.find("棒") != -1:
            plurkResponse(pid, "謝謝你的喜歡！也記得要這麼喜歡自己喔！[emo9]")
        else:
            random.shuffle(random_list)
            plurkResponse(pid, random_list[0])

            
while True:
    match = initApi()
    if match:
        rawdata = match.group(1)
    data = json.loads(rawdata)
    new_offset = data.get('new_offset', -1)
    msgs = data.get('data')
    responseMentioned()
    print("Seal bot is working!")
    if not msgs:
        continue
    for msg in msgs:
        pid = msg.get('plurk_id')
        user_id = msg.get('user_id')

        if user_id is None:
            try:
                pid = msg['plurk']['plurk_id']
                user_id = msg['plurk']['user_id']
            except Exception as e:
                print("sth wrong: msg:" + str(e))
                continue

        if str(user_id) not in friend_list:
            print("Not in friend list.")
            continue

        if msg.get('type') == 'new_plurk':
            print(f"reply now user:{user_id} msg: {msg.get('content')}")
            content = msg.get('content_raw')
            if content.find("--noreply") != -1 or content.find("慎入") != -1:
                print(":p")
            else:
                dealContent(pid, content, False, "", user_id)

    val foodList  = List ("咖哩扮飯!", "珍珠粉條仙草凍", "紅酒燉牛肉", "酸辣粉", "烤肉", "披薩", "肉桂捲", "薯條","芋頭火鍋","烤肉","車輪餅","地瓜球")

    def main (): 
        ( args: Array[String]) {
        
        val plurkBot = new PlurkBot(apiKey)

        // 登入噗浪帳號
        plurkBot.Users.login (username, password)

        // 開始機器人模式
        plurkBot.startBot {
            // 如果有新噗進來
            case NewPlurk(plurk) => 
                println ("有新噗：" + plurk.contentRaw)

                // 而且有人問吃什麼
                if (plurk.qualifier == Qualifier.Asks && plurk.contentRaw.contains("海星餓餓獸","吃什麼")) {
                
                    val food  = Random.shuffle(foodList).head

                    plurkBot.Responses.responseAdd (
                        plurkID = plurk.plurkID,
                        qualifier = Qualifier.Says,
                        content = "%s" format(food)
                    )
                }

            // 如果有新的回應
            case NewResponse(plurk, user, response) => 
                println ("有新回應：" + response.contentRaw)

                // 說不想吃
                if (response.contentRaw.contains("不要") || response.contentRaw.contains("換一個")) {

                    // 那就重新挑一個奇怪的給他
                    val food  = Random.shuffle(otherList).head

                    plurkBot.Responses.responseAdd (
                        plurkID = plurk.plurkID,
                        qualifier = Qualifier.Says,
                        content = "那海星餓餓獸說想要吃ㄋㄋ"
                    )
                }

                elif (response.contentRaw.contains("屁眼")) {

                    plurkBot.Responses.responseAdd (
                        plurkID = plurk.plurkID,
                        qualifier = Qualifier.Says,
                        content = "幹幹" 
                    )
                }

        }
    }
}

# -*- coding: utf-8 -*-
from .oauth import PlurkOAuth
from .api import PlurkAPI

__all__ = (PlurkAPI, PlurkOAuth)
