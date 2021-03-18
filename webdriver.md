---
title: "爬取matplotlib官网所有例子"
author: "Kai Zheng"
tags: ["webdriver"]
date: 2021-03-14T11:15:31+08:00
draft: true
---

爬取matplotlib官网所有例子

```diff
from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait 
import os
import urllib.request,re
import requests
PATH=lambda p:os.path.abspath(os.path.join(
    os.path.dirname(__file__), p))
class downMatplotlibDemo():
    def __init__(self):
        self.urlList=self.getUrlList()
        self.driver=webdriver.Chrome() 
        self.driver.maximize_window() 
        self.GetDemoDownload()
        self.driver.close()   
        
    def getUrlList(self):
        # try:
        url="http://matplotlib.org/devdocs/api/_as_gen/matplotlib.pyplot.subplots.html#matplotlib.pyplot.subplots"    
        matutl="http://matplotlib.org/devdocs/gallery/"
        pageContent=urllib.request.urlopen(url).read().decode('utf-8')
        if pageContent:
            linkList=re.findall('class="reference internal" href="../../gallery/(.*?)"><span class="std std-ref">(.*?)</span></a>', pageContent, re.S) 
            charList=[matutl+var[0] for var in linkList if len(linkList)>0]
            return charList
        # except Exception:
        #     print ("Create UrlList Error:") #,e
               
    def GetDemoDownload(self):
        count=0     
        for url in self.urlList:     
            self.driver.get(url)
            js="var q=document.body.scrollTop=200000"
            self.driver.execute_script(js)
            try:
                downLoadBtnList=WebDriverWait(self.driver,5).until(lambda driver:driver.find_elements_by_partial_link_text('Download'))
            except Exception:
                print ("Download not exist:") #,e
            
            try:
                if len(downLoadBtnList)>0:
                    for downLoad in downLoadBtnList:  
                        downurl=downLoad.get_attribute("href")
                        if downurl:
                            fileName=downurl.split("/")[-1]
                            if fileName:
                                filePath=PATH('./sourceCode/')
                                if os.path.exists(filePath):
                                    pass
                                else:
                                    os.mkdir(filePath)
                                fileWithPath=PATH(filePath+'\\'+fileName)
                                if not os.path.exists(fileWithPath):
                                    with open(fileWithPath,"wb") as FH:                   
                                        pageConet=requests.get(downurl)  # pageConet=urllib.urlopen(downurl).read() 
                                        FH.write(pageConet.content)  # FH.write(pageConet)
                                else:
                                    print( "the file with path is exists....")
                            else:
                                print ("The file name is null!")
                        else:
                            print ("the download url is null!")
            except Exception:
                print ("Download List:") #,e              
            count+=1      
            print (count,"\t url=",downurl)
                                               
  
if __name__=="__main__":
    downMatplotlibDemo()
```


![avatar](https://gimg2.baidu.com/image_search/src=http%3A%2F%2Fgss0.baidu.com%2F7Po3dSag_xI4khGko9WTAnF6hhy%2Fzhidao%2Fpic%2Fitem%2F78310a55b319ebc4881ef8e68026cffc1e17166e.jpg&refer=http%3A%2F%2Fgss0.baidu.com&app=2002&size=f9999,10000&q=a80&n=0&g=0n&fmt=jpeg?sec=1618024180&t=f358d1d62dd0b89c0cf2670b7f596ce7)

Reference：https://zhuanlan.zhihu.com/p/161510012

