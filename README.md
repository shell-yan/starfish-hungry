git@github.com:shell-yan/bot.git

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
from typing import List, ValuesView
import scala.util.Random
 
from PlurkAPI import PlurkAPI
 
plurk = PlurkAPI('DDoPEkyGXNd2', 'WOSxyGxr1vQbmip6hNdmBSrU78NgW8hO')
plurk.authorize('avJGJBqdfD3P', 'v34XamJ3097vMTgZEPYYJUm2f5x3lDUU')
 
comet = plurk.callAPI('/APP/Realtime/getUserChannel')
comet_channel = comet.get('comet_server') + "&new_offset=%d"
jsonp_re = re.compile('CometChannel.scriptCallback\((.+)\);\s*');
new_offset = -1

foodList  = List ("咖哩扮飯!", "珍珠粉條仙草凍", "紅酒燉牛肉", "酸辣粉", "烤肉", "披薩", "肉桂捲", "薯條","芋頭火鍋","烤肉","車輪餅","地瓜球")
otherList = List ("答辯","那海星餓餓獸想要吃ㄋㄋ","闢眼")
def main (): 
        ( args: Array[String]) {
        
        val plurkBot = new PlurkBot(apiKey)

        // 登入噗浪帳號
        plurkBot.Users.login (username, password)

        // 開始機器人模式
        plurkBot.startBot {

                // 而且有人問吃什麼
                if (plurk.qualifier == Qualifier.Asks && plurk.contentRaw.contains()) {
                
                    val food  = Random.shuffle(foodList).head                   

        }
    }
}
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
            if content.find("海星餓餓獸","吃什麼") != -1:
                plurk.callAPI('/APP/Responses/responseAdd',
                              {'plurk_id': pid,
                               'content': foodList,
                               'qualifier': ':' })

            if (response.contentRaw.contains("不要") || response.contentRaw.contains("換一個"))   
                     {       
                    // 那就重新挑一個奇怪的給他
                    val food  = Random.shuffle(otherList).head

            plurkBot.Responses.responseAdd (
                        plurkID = plurk.plurkID,
                        qualifier = Qualifier.Says,
                        content = otherList
                    )}
                }

# -*- coding: utf-8 -*-
from .oauth import PlurkOAuth
from .api import PlurkAPI

__all__ = (PlurkAPI, PlurkOAuth)

# -*- coding: utf-8 -*-
from oauth2 import (
    Client, Consumer, Request,
    SignatureMethod_HMAC_SHA1, Token,
)
import requests

# compatible python3
import sys
if sys.version_info >= (3, 0, 0):
    from urllib.parse import parse_qsl, parse_qs
    from urllib.parse import urlencode
else:
    from urlparse import parse_qsl
    from urllib import urlencode
    input = raw_input


class PlurkOAuth:
    def __init__(self, customer_key=None, customer_secret=None):
        self.base_url = 'https://www.plurk.com'
        self.request_token_url = '/OAuth/request_token'
        self.authorization_url = '/OAuth/authorize'
        self.access_token_url = '/OAuth/access_token'
        self.customer_key = customer_key
        self.customer_secret = customer_secret
        self.sign_method = SignatureMethod_HMAC_SHA1()
        self.consumer = None
        self.token = None
        self.oauth_token = {}
        if self.customer_key and self.customer_secret:
            self.consumer = Consumer(self.customer_key, self.customer_secret)

    def __unicode__(self):
        return self.base_url

    def _dump(self, data):
        # import pprint
        # pprint.pprint(data)
        pass

    def authorize(self, access_token_key=None, access_token_secret=None):
        while not self.consumer:
            self.get_consumer_token()
        if access_token_key and access_token_secret:
            self.oauth_token['oauth_token'] = access_token_key
            self.oauth_token['oauth_token_secret'] = access_token_secret
        else:
            self.get_request_token()
            verifier = self.get_verifier()
            self.get_access_token(verifier)

    def request(self, url, params={}, data={}, files={}):
        """ Return: status code, json object, status reason """

        # Setup
        if self.oauth_token:
            self.token = Token(self.oauth_token['oauth_token'],
                               self.oauth_token['oauth_token_secret'])
        req = self._make_request(self.base_url + url, data)

        req_files = {}
        try:
            if files:
                for (name, fpath) in files.items():
                    req_files[name] = open(fpath, 'rb')

            r = requests.post(
                self.base_url + url,
                headers=req.to_header(),
                data=data,
                files=req_files if req_files else None
            )

        except requests.RequestException as ex:
            print >> sys.stderr, ex
            sys.exit(1)
        except:
            print >> sys.stderr
            sys.exit(1)
        finally:
            for name, ofile in req_files.items():
                 ofile.close()

        #print(r.text)

        try:
            return r.status_code, r.json(), r.reason
        except:
            return r.status_code, r.text, r.reason

    def get_consumer_token(self):

        # Setup
        print("Prepare the CONSUMER Info")

        verified = 'n'
        while verified.lower() != 'y':
            key = input('Input the CONSUMER_KEY: ')
            secret = input('Input the CONSUMER_SECRET: ')
            print('Consumer Key: %s' % str(key))
            print('Consumer Secret: %s' % str(secret))
            verified = input('Are you sure? (y/N) ')
        self.customer_key = key
        self.customer_secret = secret
        self.consumer = Consumer(self.customer_key, self.customer_secret)

    def _make_request(self, requestURL, param=None):
        request = Request.from_consumer_and_token(consumer=self.consumer,
                                                  token=self.token,
                                                  http_method='POST',
                                                  http_url=requestURL,
                                                  parameters=param,
                                                  is_form_encoded=True)
        request.sign_request(self.sign_method, self.consumer, self.token)
        return request

    def _has_pending_oauth_token(self):
        # TODO we dont know this is request or access token, may ambiguous
        return self.oauth_token \
            and 'oauth_token' in self.oauth_token \
            and 'oauth_token_secret' in self.oauth_token

    def get_request_token(self):

        if self._has_pending_oauth_token():
            # Already has a request/access token
            return

        # Get Token Key/Secret
        status, content, reason = self.request(self.request_token_url)
        if str(status) != '200':
            # TODO Declare an exception
            raise Exception(reason)
        self.oauth_token = dict(parse_qsl(content))
        self._dump(self.oauth_token)
        # print('Token Key: %s' % str(token['oauth_token']))
        # print('Token Secret: %s' % str(token['oauth_token_secret']))

    def get_verifier(self):

        # Setup
        print("Open the following URL and authorize it")
        print("%s?oauth_token=%s" % (self.base_url + self.authorization_url,
                                     self.oauth_token['oauth_token']))

        verified = 'n'
        while verified.lower() == 'n':
            verifier = input('Input the verification number: ')
            verified = input('Are you sure? (y/n) ')
        return verifier

    def get_verifier_url(self):

        if not self.oauth_token or 'oauth_token' not in self.oauth_token:
            # TODO Declare an exception
            raise Exception('Please request a token first')
        return '{0}{1}?oauth_token={2}'.format(self.base_url,
                                               self.authorization_url,
                                               self.oauth_token['oauth_token'])

    def get_access_token(self, verifier):

        status, content, reason = self.request(self.access_token_url, data={
            'oauth_token_secret': self.oauth_token['oauth_token_secret'],
            'oauth_verifier': verifier,
        })
        if str(status) != '200':
            # TODO Declare an exception
            raise Exception(reason)
        # Get Token Key/Secret
        self.oauth_token = dict(parse_qsl(content))
        self._dump(self.oauth_token)
        # print('Access Key: %s' % str(self.oauth_token['oauth_token']))
        # print('Access Secret: %s' % str(self.oauth_token['oauth_token_secret']))
