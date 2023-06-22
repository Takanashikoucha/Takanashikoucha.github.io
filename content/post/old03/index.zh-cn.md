---
title: "第一个完整的python程序"
slug: ""
description: "直接用tkinter撸GUI，我觉得也是有点过于懒"
date: 2023-06-23T02:39:36+08:00
lastmod: 2023-06-23T02:39:36+08:00
draft: false
toc: true
weight: false
image: ""
categories: ["old"]
tags: ["old"]
---
稳,从萌萌那里接了个GUI外包.

算是自己第一个完整的python程序.

下面是源代码,顺便也测试一下博客的代码显示效果:

```python
# coding=utf-8
"""
Created on 2017-4-9
"""


# import
import os
import tkinter
from tkinter import ttk
from tkinter import scrolledtext
from tkinter.filedialog import askdirectory
from queue import Queue
import re
import requests
import xlwt
from bs4 import BeautifulSoup
from threading import Thread
from xml.etree import ElementTree as ET
import logging


# set logging
logging.basicConfig(level=logging.INFO,
                    format='%(asctime)s %(levelname)s %(message)s',
                    datefmt='%d %b %Y %H:%M:%S',
                    filename='boxlog.log',
                    filemode='a+')
con = logging.StreamHandler()
con.setLevel(logging.INFO)
formater = logging.Formatter('%(levelname)-8s %(message)s')
con.setFormatter(formater)
logging.getLogger('').addHandler(con)


# global set
global SCHE
global oknum
global savep
global KEYPOOL
savep = os.getcwd()
oknum = 0
SCHE = 0
HEADERS = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64)'}
FAILEDURL = []

# spyder


class DataWorker(Thread):
    def __init__(self, univ, queue):
        Thread.__init__(self)
        self.univ = univ
        self.queue = queue

    def run(self):
        while 1:
            URL = self.queue.get()
            self.univ.save_data(URL)
            self.queue.task_done()


class AddWorker(Thread):
    def __init__(self, univ, queue):
        Thread.__init__(self)
        self.univ = univ
        self.queue = queue

    def run(self):
        while 1:
            IDX = self.queue.get()
            self.univ.get_address(IDX)
            self.queue.task_done()
            global SCHE
            global oknum
            SCHE = (oknum / (len(self.univ.stu_data) * 2) * 100)
            P1.config(value=SCHE)
            oknum = oknum + 1


class PCWorker(Thread):
    def __init__(self, univ, queue):
        Thread.__init__(self)
        self.univ = univ
        self.queue = queue

    def run(self):
        while 1:
            IDX = self.queue.get()
            self.univ.get_pc(IDX)
            self.queue.task_done()
            global SCHE
            global oknum
            SCHE = (oknum / (len(self.univ.stu_data) * 2) * 100)
            P1.config(value=SCHE)
            oknum = oknum + 1


class Spider(object):
    def __init__(self, url, stu_data=[], page=0.1, university=None):
        self.url = url
        self.stu_data = stu_data
        self.page = page
        self.university = university

    def initialization(self):
        REQ = requests.get(self.url,
                           headers=HEADERS,
                           verify=False)
        REQ.encoding = 'utf8'
        HTML = REQ.text
        SOUP = BeautifulSoup(HTML, 'lxml')

        STU_INF = SOUP.body.contents[3].table.find_all('tr')[1:]
        PATTERN = re.compile('<td>(.+?)</td>')
        for INF in STU_INF:
            self.stu_data.append(re.findall(PATTERN, str(INF)))
        # 获取总页数
        P = SOUP.body.contents[3].div.div.form
        PATTERN = re.compile('共.+?>(\d+?)</')
        self.page = int(re.findall(PATTERN, str(P))[0])
        # 获取大学名字
        U = SOUP.body.contents[3].h3
        PATTERN = re.compile('>\r\n\t\t(.+?)\r\n\t\t名单公示')
        self.university = re.findall(PATTERN, str(U))[0]
    # 生成所有页面的url池

    def get_urlpool(self):
        URLPOOL = []
        for i in range(2, self.page + 1):
            URLPOOL.append(self.url + '&start=%d' % ((i - 1) * 30))
        return URLPOOL

    def get_data(self, urlpool):
        QUEUE = Queue()
        for url in urlpool:
            QUEUE.put(url)
        for _ in range(6):
            worker = DataWorker(self, QUEUE)
            worker.daemon = True
            worker.start()
        QUEUE.join()

    def save_data(self, url):
        try:
            REQ = requests.get(url,
                               headers=HEADERS,
                               verify=False,
                               timeout=25)
        except:
            FAILEDURL.append(url)
            logging.warning(url + '\nfailed to download.')
            T1.insert(tkinter.END, '\n' + url + '\nfailed to download.','red')
            T1.see(tkinter.END)
            return
        REQ.encoding = 'utf8'
        HTML = REQ.text
        SOUP = BeautifulSoup(HTML, 'lxml')

        STU_INF = SOUP.body.contents[3].table.find_all('tr')[1:]
        PATTERN = re.compile('<td>(.+?)</td>')
        for INF in STU_INF:
            self.stu_data.append(re.findall(PATTERN, str(INF)))
        logging.info('complete')
        T1.insert(tkinter.END, '\n' + 'complete','green')
        T1.see(tkinter.END)

    def get_address(self, idx):
        school = self.stu_data[idx][2]
        try:
            while 1:
                global KEYPOOL
                KEY = KEYPOOL[-1]
                URL = ('http://restapi.amap.com/v3/place/text?&output=XML' +
                       '&key=%s&keywords=%s' % (KEY, school))
                REQ = requests.get(URL, timeout=25)
                HTML = REQ.text
                tree = ET.fromstring(HTML)
                INFO = tree.find('info').text
                logging.info(INFO)
                T1.insert(tkinter.END, '\n' + 'INFO          OK','green')
                T1.see(tkinter.END)
                if INFO == 'OK':
                    break
                logging.warning('\nkey: ' + KEY + ' runs up to daily limit')
                T1.insert(tkinter.END, '\n' + '\nkey: ' +
                          KEY + ' runs up to daily limit','blue')
                T1.see(tkinter.END)
                try:
                    KEYPOOL.remove(KEY)
                except:
                    logging.info('\nkey: ' + KEY + ' was removed')
                    T1.insert(tkinter.END, '\n' + '\nkey: ' +
                              KEY + ' was removed','blue')
                    T1.see(tkinter.END)
                logging.info('\nnow switching to key: ' + KEYPOOL[-1])
                T1.insert(tkinter.END, '\n' +
                          '\nnow switching to key: ' + KEYPOOL[-1],'blue')
                T1.see(tkinter.END)
            ADDS = tree.find('pois')[0].find('address').text
            CITY = tree.find('pois')[0].find('cityname').text
            AREA = tree.find('pois')[0].find('adname').text
            # print(ADDS, CITY, AREA)
            if ADDS == None:
                ADDS = 'Unknown'
            AREA = CITY + AREA
        except:
            AREA = 'Unknown'
            ADDS = 'Unknown'
        self.stu_data[idx] += [AREA, ADDS]
        # print(school+'\n'+AREA+' '+ADDS)

    def add_address(self):
        QUEUE = Queue()
        for idx in range(len(self.stu_data)):
            QUEUE.put(idx)
        for _ in range(10):
            worker = AddWorker(self, QUEUE)
            worker.daemon = True
            worker.start()
        QUEUE.join()

    def get_pc(self, idx):
        stu = self.stu_data[idx]
        if stu[4] == 'Unknown':
            self.stu_data[idx].append('Unknown')
            return
        try:
            AD = str(stu[4].encode('gb2312')).replace('\\x', '%').lstrip('b\'')
            URL = 'http://opendata.baidu.com/post/s?wd=%s&rn=20' % AD
            REQ = requests.get(URL, headers=HEADERS, timeout=25)
            REQ.encoding = 'gb2312'
            HTML = REQ.text
            SOUP = BeautifulSoup(HTML, 'lxml')
            PC = str(SOUP.body.section.table.find_all('tr')[1])
            PATTERN = re.compile('td>(\d+?)</td')
            PC = re.findall(PATTERN, PC)[0]
        except:
            PC = 'Unknown'
        self.stu_data[idx].append(PC)
        logging.info(PC)
        T1.insert(tkinter.END, '\n' + PC,'green')
        T1.see(tkinter.END)

    def add_pc(self):
        QUEUE = Queue()
        for idx in range(len(self.stu_data)):
            QUEUE.put(idx)
        for _ in range(10):
            worker = PCWorker(self, QUEUE)
            worker.daemon = True
            worker.start()
        QUEUE.join()


def exl(univ):
    # 初始化文档
    data = univ.stu_data
    university = univ.university
    workbook = xlwt.Workbook(encoding='ascii')
    worksheet = workbook.add_sheet('%s自招通过名单' % university)
    worksheet.write(0, 0, label='姓名')
    worksheet.write(0, 1, label='性别')
    worksheet.write(0, 2, label='中学')
    worksheet.write(0, 3, label='省份')
    worksheet.write(0, 4, label='城市')
    worksheet.write(0, 5, label='地址')
    worksheet.write(0, 6, label='邮编')
    # 设置列宽
    worksheet.col(2).width = 256 * 40
    worksheet.col(3).width = 256 * 15
    worksheet.col(4).width = 256 * 25
    worksheet.col(5).width = 256 * 35
    # 写入数据
    for idx in range(len(data)):
        for j in range(7):
            worksheet.write(idx + 1, j, label=data[idx][j])
    # 保存
    workbook.save('%s自招通过名单.xls' % university)
    logging.info(u'共有%s个数据' % (len(data)))
    T1.insert(tkinter.END, '\n' + u'共有%s个数据' % (len(data)),'green')
    T1.see(tkinter.END)


def main(url, keylist, sche):
    download.set('下载进度:')
    L4.config(fg = 'black')
    URL = url
    global SCHE
    global KEYPOOL
    KEYPOOL = keylist
    UNIV = Spider(URL)
    try:
        UNIV.initialization()
        logging.info(u'初始化完成,开始创建url池')
        T1.insert(tkinter.END, '\n' + u'初始化完成,开始创建url池','green')
        T1.see(tkinter.END)
    except:
        logging.warning('url error, pls check the url and try again.')
        T1.insert(tkinter.END, '\n' +
                  'url error, pls check the url and try again.','red')
        T1.see(tkinter.END)
        return
    UNIV.get_data(UNIV.get_urlpool())
    logging.info(u'开始获取地址')
    T1.insert(tkinter.END, '\n' + u'开始获取地址','green')
    T1.see(tkinter.END)
    UNIV.add_address()
    logging.info(u'开始获取邮编')
    T1.insert(tkinter.END, '\n' + u'开始获取邮编','green')
    T1.see(tkinter.END)
    print(SCHE)
    UNIV.add_pc()
    exl(UNIV)
    SCHE = 100
    if len(FAILEDURL) == 0:
        logging.info(u'所有信息下载完毕')
        T1.insert(tkinter.END, '\n' + u'所有信息下载完毕','green')
        T1.see(tkinter.END)
    else:
        s = ''
        for url in FAILEDURL:
            s += '\n' + url
        logging.warning(u'有%d个页面的信息未被下载，分别是:%s' % (len(FAILEDURL), s))
        T1.insert(tkinter.END, '\n' + u'有%d个页面的信息未被下载，分别是:%s' %
                  (len(FAILEDURL), s),'red')
        T1.see(tkinter.END)
    logging.info(u'已保存')
    T1.insert(tkinter.END, '\n' + u'已保存','green')
    T1.see(tkinter.END)
    SCHE = 100
    P1.config(value=SCHE)
    download.set('下载完成')
    L4.config(fg = 'green')


# GUI

root = tkinter.Tk()
root.geometry('745x345+500+250')
root.resizable(False, False)
root.title('box_EDU')

L1 = tkinter.Label(root, text="URL地址:")
L2 = tkinter.Label(root, text="保存位置:")
L1.grid(row=0)
L2.grid(row=1)

e1 = tkinter.Entry(root, width=60)
path = tkinter.StringVar()
e2 = tkinter.Entry(root, width=60, textvariable=path)
e1.grid(row=0, column=1, columnspan=2, pady=2)
e2.grid(row=1, column=1, columnspan=2, pady=2)

b1 = tkinter.Button(root, text="开始", height=1, width=25)

b2 = tkinter.Button(root, text="选择", height=1, width=25)
b1.grid(row=0, column=3, columnspan=2, padx=2, pady=2)
b2.grid(row=1, column=3, columnspan=2, padx=2, pady=2)

T1 = scrolledtext.ScrolledText(root, height=16, width=60)
T1.tag_config('red',foreground = 'red')
T1.tag_config('green',foreground = 'green')
T1.tag_config('blue',foreground = 'blue')
T1.grid(row=3, rowspan=5, columnspan=2, padx=20)

L3 = tkinter.Label(root, text="KEY池")
L3.grid(row=3, column=3, columnspan=2)

e3 = tkinter.Entry(root,  width=35)
e3.grid(row=4, column=3, columnspan=2)

lis1 = tkinter.Listbox(root, width=35)
lis1.grid(row=5, column=3, rowspan=2, columnspan=2, padx=2, pady=2)

b3 = tkinter.Button(root, text="添加", height=1, width=8)
b4 = tkinter.Button(root, text="删除", height=1, width=8)
b3.grid(row=7, rowspan=2, column=3, padx=2, pady=2)
b4.grid(row=7, rowspan=2, column=4, padx=2, pady=2)

download = tkinter.StringVar()
download.set('下载进度:')
L4 = tkinter.Label(root, text="下载进度:",textvariable = download)
L4.grid(row=8, pady=2)

P1 = ttk.Progressbar(root, length=400, mode="determinate",
                     maximum=100, value=SCHE)
P1.grid(row=8, column=1, columnspan=2, pady=2)


def wf(fn):
    KEYpool = list(set(keypool))
    global KEYPOOL
    KEYPOOL = KEYpool
    fw = open(fn, 'w+')
    fw.truncate()
    for key in KEYPOOL:
        fw.write(key + '\n')
    fw.close()


def upbox():
    KEYpool = list(set(keypool))
    global KEYPOOL
    KEYPOOL = KEYpool
    lis1.delete(0, tkinter.END)
    for i in KEYPOOL:
        lis1.insert(tkinter.END, i)
    wf('keypool.txt')


def urlpath(event):
    global URL
    global savep
    URL = e1.get()
    savep = e2.get()
    os.chdir(savep)
    logging.info(URL)
    t = Thread(target=main, args=(URL, KEYPOOL, SCHE))
    t.setDaemon(True)
    t.start()


def selectPath(event):
    path_ = askdirectory()
    path.set(path_)


def keypooladd(event):
    try:
        pattern = re.compile('^\w{32}$')
        key = re.findall(pattern, e3.get())
        if len(key) == 1:
            keypool.append(e3.get())
            upbox()
        else:
            logging.warning('兄弟key输错了啊...')
            T1.insert(tkinter.END, '\n' + '兄弟key输错了啊...','red')
            T1.see(tkinter.END)
    except:
        # 虽然我觉得并不会出错
        logging.warning('有一个你知道的问题发生了')
        T1.insert(tkinter.END, '\n' + '有一个你知道的问题发生了','red')
        T1.see(tkinter.END)


def keypoolsub(event):
    try:
        keypool.remove(lis1.get(lis1.curselection()))
        upbox()
    except:
        logging.warning('有一个你知道的错误发生了')
        T1.insert(tkinter.END, '\n' + '有一个你知道的错误发生了','red')
        T1.see(tkinter.END)


# bind
b1.bind("<Button-1>", urlpath)
b2.bind("<ButtonRelease-1>", selectPath)
b3.bind("<Button-1>", keypooladd)
b4.bind("<Button-1>", keypoolsub)

keypool = []
path.set(savep)
try:
    keypoolf = open('keypool.txt', 'r+')
    for line in keypoolf:
        keypool.append(line.replace('\n', ''))
    KEYpool = list(set(keypool))
    KEYPOOL = KEYpool
    for i in KEYpool:
        lis1.insert(tkinter.END, i)
    T1.insert(tkinter.END,'初始化成功','green')
    T1.see(tkinter.END)
except:
    keypoolf = open('keypool.txt', 'w+')
    for line in keypoolf:
           keypool.append(line.replace('\n', ''))
    KEYpool = list(set(keypool))
    KEYPOOL = KEYpool
    for i in KEYpool:
        lis1.insert(tkinter.END, i)
    T1.insert(tkinter.END,'神他喵的根本没有key池','red')
    T1.see(tkinter.END)
root.mainloop()
```

----------
给我来瓶冰阔落？

![Code](alipay.jpg)