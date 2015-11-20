#### exercise 2
   - 주기적으로 특정 웹페이지(http://www.airkorea.or.kr) 크롤링하여 openTSDB에 저장 (미세먼지)
   - ex1 번의 json 데이터를 추가하여 작성..metric은 변경하여 사용하길 바라(참고:dust)
   - url 중복 확인, split 한 변수가 string이므로 int로 형변환 int(c[1])

##### 참고
```sh

#!/usr/bin/python
# -*- coding: utf-8 -*- 

import urllib2 # extensible library for opening URLs
import time

# 인천 미세먼지 
url = 'http://www.airkorea.or.kr/index'


def getData(buffers):
	a = buffers.split('<tbody id="mt_mmc2_10007">')[1]
	#print a
	
	b = a.split('</tbody>')[0].replace('<tr>','').replace('</tr>','').replace('</td>','')
	#print b

	c = b.split('<td>')
	#print c[1]
	#print c[2]

if __name__ == '__main__':

	page = urllib2.urlopen(url).read()
	print page

	getData(page)

```

##### 아래 코드를 참고하여 서울과 인천의 미세먼지 수치를 openTSDB에 저장하고 웹에서 확인

```sh

#!/usr/bin/python
# -*- coding: utf-8 -*- 

import sys
import urllib2
import time
from datetime import datetime, timedelta
import json
import requests


# 인천 미세먼지 
url = 'http://www.airkorea.or.kr/index'
url_local ="http://127.0.0.1:4242/api/put"

def insert(metric_name, tag_site, value):
	data={
		"metric":metric_name,
		"timestamp":time.time(),
		"value":value,
		"tags":{
			"site":tag_site
		}
	}
	ret = requests.post(url_local, data=json.dumps(data))
	time.sleep(1)


def getData(buffers):
	a = buffers.split('<tbody id="mt_mmc2_10007">')[1]
	#print a
	
	b = a.split('</tbody>')[0].replace('<tr>','').replace('</tr>','').replace('</td>','')
	#print b

	c = b.split('<td>')
	# seoul
	print c[1]
	print c[2]

	# incheon
	print c[7]
	print c[8]

	insert('dust', 'incheon', int(c[8]))
	time.sleep(1)

if __name__ == '__main__':
	while 1 :
		t = time.localtime()
		tsec = t.tm_sec
		
		if tsec%10!=0:
			print tsec
			time.sleep(1)
		else :
			page = urllib2.urlopen(url).read()
			getData(page)

```
   
   
#### exercise 3
   - 기상청 웹페이지(http://www.kma.go.kr/weather/lifenindustry/sevice_rss.jsp) 
   - xml 형태로 리턴해줌
   - 크롤링하여 온도를 출력하고 openTSDB에 저장

##### 참고
```sh

#!/usr/bin/python
# -*- coding: utf-8 -*- 

##################################################
# 기상청
# http://www.kma.go.kr/weather/lifenindustry/sevice_rss.jsp

# RSS : 웹사이트 상의 컨텐츠를 요약하고 상호 공유할 수 있도록 만든 표준 XML을 기초로 만들어진 데이터 형식
# RSS 인천 용현동1,4
# http://www.kma.go.kr/wid/queryDFSRSS.jsp?zone=2817055500

# lxml library 설치해야함
# yum install python-lxml
##################################################

import urllib2 # extensible library for opening URLs
import time

from lxml.html import parse, fromstring # processing XML and HTML

# 인천 남구 용현동 기상상황 확인 url
url = 'http://www.kma.go.kr/wid/queryDFSRSS.jsp?zone=2823759100'
temp=[]

def temp_process(xml):
	for  elt in xml.getiterator("temp"):	# getting temp tag 
		temp_val = elt.text
		print temp_val

if __name__ == '__main__':
	page = urllib2.urlopen(url).read()
	print page
	
	# fromstring : Parses an XML document or fragment from a string. 
	# Returns the root node (or the result returned by a parser target).
	xml_raw = fromstring(page)

	# processing temperature
	temp_process(xml_raw)

```

#### exercise 3은 exercise 2를 참고하여 실행
```sh
#!/usr/bin/python
# -*- coding: utf-8 -*- 

##################################################
# 기상청
# http://www.kma.go.kr/weather/lifenindustry/sevice_rss.jsp

# RSS : 웹사이트 상의 컨텐츠를 요약하고 상호 공유할 수 있도록 만든 표준 XML을 기
초로 만들어진 데이터 형식
# RSS 인천 용현동1,4
# http://www.kma.go.kr/wid/queryDFSRSS.jsp?zone=2817055500

# lxml library 설치해야함
# yum install python-lxml
##################################################

import urllib2 # extensible library for opening URLs
import time
import sys
from datetime import datetime, timedelta
import json
import requests

from lxml.html import parse, fromstring

# 인천 남구 용현동 기상상황 확인 url
url = 'http://www.kma.go.kr/wid/queryDFSRSS.jsp?zone=2823759100'
url_local = "http://127.0.0.1:4242/api/put"
temp=[]

def insert(metric_name, tag_site, value):
    data={
        "metric":metric_name,
        "timestamp":time.time(),
        "value":value,
        "tags":{
            "site":tag_site
        }
    }
    ret = requests.post(url_local, data=json.dumps(data))
    time.sleep(1)

def temp_process(xml):
        for  elt in xml.getiterator("temp"):    # getting temp tag 
                temp_val = elt.text
        print temp_val

        insert('temperature', 'inchon', temp_val)
        insert('temperature', 'seoul', 20)



if __name__ == '__main__':
    while 1 :
        t = time.localtime()
        tsec = t.tm_sec

        if tsec%10!=0:
            print tsec
            time.sleep(1)
        else :
            page = urllib2.urlopen(url).read()
            #getData(page)

            # fromstring : Parses an XML document or fragment from a string. 
            # Returns the root node (or the result returned by a parser target).
            xml_raw = fromstring(page)

            # processing temperature
            temp_process(xml_raw)
```

#### exercise 4
   - openTSDB API 호출하고 json 형식의 데이터를 파싱
   - 호출 url
```sh
http://125.7.128.53:4242/#start=10m-ago&m=sum:gyu_RC1_thl.temperature%7Bnodeid=2454%7D&o=&key=out%20bottom%20center%20box&wxh=600x300&autoreload=15
```

   - json 형식의 데이터 호출
```sh
http://125.7.128.53:4242/api/query?start=10m-ago&m=sum:gyu_RC1_thl.temperature%7Bnodeid=2454%7D&o=&key=out%20bottom%20center%20box&wxh=600x300&autoreload=15
```


```sh

#!/usr/bin/python

import sys
import urllib2
import json
import struct
import time
from datetime import datetime, timedelta

url = 'http://125.7.128.53:4242/api/query?start=10m-ago&m=sum:gyu_RC1_thl.temperature%7Bnodeid=2454%7D&o=&key=out%20bottom%20center%20box&wxh=600x300&autoreload=15'

def dataParser(s):
        s = s.split("{")
        i = 0
        j = len(s)
        if j > 2:
                s = s[3].replace("}","")
                s = s.replace("]","")
        else:
                return '0:0,0:0'
        return s

if __name__ == '__main__':
        while 1 :
                t = time.localtime()
                tsec = t.tm_sec

                if tsec%10!=0 :
                        print tsec
                        time.sleep(0.8)
                else :
                        #endTimeUnix = time.time()
                        #startTimeUnix = endTimeUnix - 1800 

                        #startTime = datetime.fromtimestamp(startTimeUnix).strftime('%Y/%m/%d-%H:%M:%S')
                        #endTime = datetime.fromtimestamp(endTimeUnix).strftime('%Y/%m/%d-%H:%M:%S')

                        #print startTime
                        #print endTime

                        #metric = 'cc_100.test'
                        #url = 'http://127.0.0.1:4242/api/query?start=' + startTime + '&end=' + endTime + '&m=sum:' + metric
                        #print url

                        try:
                                u = urllib2.urlopen(url)
                        except:
                                raise NameError('url error')

                        data = u.read()
                        #print data
                        packets = dataParser(data)
                        packet = packets.split(',')

                        j=len(packet)

                        v_s = packet[0].split(":")
                        v_e = packet[j-1].split(":")

                        print v_s[0], v_s[1]
                        #print "cc.test %d %d" % ( endTimeUnix, v_e )
                        time.sleep(0.8)

```
