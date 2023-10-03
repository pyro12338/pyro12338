import websockets
import json
import asyncio
import re
from hangul_utils import join_jamos
import requests

clients = []
devWebSocket = None
names=[]
devs = [
  "ffmgctic",
  "SACFP107",
  "saha_1412"
]
sentclients = 0

cons = {'r':'ㄱ', 'R':'ㄲ', 's':'ㄴ', 'e':'ㄷ', 'E':'ㄸ', 'f':'ㄹ', 'a':'ㅁ', 'q':'ㅂ', 'Q':'ㅃ', 't':'ㅅ', 'T':'ㅆ',
           'd':'ㅇ', 'w':'ㅈ', 'W':'ㅉ', 'c':'ㅊ', 'z':'ㅋ', 'x':'ㅌ', 'v':'ㅍ', 'g':'ㅎ'}
vowels = {'k':'ㅏ', "K":"ㅏ", 'o':'ㅐ', "I":"ㅑ", 'i':'ㅑ', 'O':'ㅒ', 'j':'ㅓ', "J":"ㅓ", 'p':'ㅔ', 'u':'ㅕ', 'U':"ㅕ", 'P':'ㅖ', 'h':'ㅗ', "H":"ㅗ", 'hk':'ㅘ', 'ho':'ㅙ', 'hl':'ㅚ',
           'y':'ㅛ', "Y":"ㅛ", 'N':'ㅜ', 'n':'ㅜ', 'nj':'ㅝ', 'np':'ㅞ', 'nl':'ㅟ', 'Nj':'ㅝ', 'Np':'ㅞ', 'Nl':'ㅟ', 'b':'ㅠ', 'B':'ㅠ', 'M':'ㅡ', 'm':'ㅡ', 'ml':'ㅢ', 'l':'ㅣ', 'L':'ㅣ',' ':' '}
cons_double = {'rt':'ㄳ', 'sw':'ㄵ', 'sg':'ㄶ', 'fr':'ㄺ', 'fa':'ㄻ', 'fq':'ㄼ', 'ft':'ㄽ', 'fx':'ㄾ', 'fv':'ㄿ', 'fg':'ㅀ', 'qt':'ㅄ'}

def engkor(text):
    engs = re.findall('"([^"]*)"', text)
    for i in range(len(engs)):
        text = text.replace(f'"{engs[i]}"', f'[{(int(i)+1)*"№"}]')
    result = ''
    vc = '' 
    for t in text:
        if t in cons :
            vc+='c'
        elif t in vowels:
            vc+='v'
        else:
            vc+='!'
    vc = vc.replace('cvv', 'fVV').replace('cv', 'fv').replace('cc', 'dd')
    i = 0
    while i < len(text):
        v = vc[i]
        t = text[i]
        j = 1
        try:
            if v == 'f' or v == 'c':
                result+=cons[t]
            elif v == 'V':
                result+=vowels[text[i:i+2]]
                j+=1
            elif v == 'v':
                result+=vowels[t]
            elif v == 'd':
                result+=cons_double[text[i:i+2]]
                j+=1
            else:
                try:
                    result+=vowels[t]
                except KeyError:
                    result+=t
        except:
            if v in cons:
                result+=cons[t]
            elif v in vowels:
                result+=vowels[t]
            else:
                try:
                    result+=vowels[t]
                except KeyError:
                    result+=t
        i += j
    result = join_jamos(result)
    for i in range(len(engs)):
        result = result.replace(f'[{(int(i)+1)*"№"}]', f'{engs[i]}')
    return result

async def accept(websocket, path):
    clients.append(websocket)
    print("Connect User")
    while True:
        try:
            data = json.loads(str(await websocket.recv()))
            if data:
                if data["Type"] == "SetDev" and data["UserName"] in devs:
                    global devWebSocket
                    devWebSocket = websocket
                if data["Type"] == "E2KC":
                    await websocket.send(json.JSONEncoder().encode({
                        "Type":"Chat",
                        "Kor":engkor(data["Eng"])
                    }))
                if data["Type"] == "E2KV":
                    await websocket.send(json.JSONEncoder().encode({
                        "Type":"View",
                        "Kor":engkor(data["Eng"])
                    }))
                if data["Type"] == "Verify":
                    global names
                    global sentclients
                    names.append(data["UserName"])
                    sentclients += 1
                    if sentclients == len(clients):
                        await devWebSocket.send(json.JSONEncoder().encode({
                            "Type":"SendUsers",
                            "Users":names
                        }))
                        names = []
                        sentclients = 0
                if data["Type"] == "request":
                    if data["Method"] == "POST":
                        requests.post(data["Url"], headers=data["Headers"] or {}, json=data["Body"] or {})
                    elif data["Method"] == "GET":
                        res = requests.get()
                        await websocket.send(res.json())
                    elif data["Method"] == "DELETE":
                        requests.delete()
                    elif data["Method"] == "FETCH":
                        requests.fetch()
                    else:
                        await websocket.send("Error Method")
        except Exception as e:
            clients.remove(websocket)
            print("Disconnect User")
            break
    return

async def main():
  async with websockets.serve(accept, "0.0.0.0", 25567):
    print("Loaded Websocket Server")
    await asyncio.Future()

asyncio.run(main())
