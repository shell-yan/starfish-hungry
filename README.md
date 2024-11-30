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
