# 20151106
수업강의내용 
#### exercise 1
   - 주기적으로 증가하는 데이터를 출력

```sh
vim test.py
```

```sh

#!/usr/bin/python
# -*- coding: utf-8 -*- 

count = 0

if __name__ == '__main__':

	while 1 :
		t = time.localtime()
		tsec = t.tm_sec
		
		if tsec%10!=0 :
			print tsec
			time.sleep(0.8)
		else :
			print count
	
```
   
#### exercise 2
   - 주기적으로 증가한는 데이터를 openTSDB에 저장
   
#### exercise 3
   - 주기적으로 특정 웹페이지(http://www.airkorea.or.kr) 크롤링하여 출력 (미세먼지)

##### 참고
```sh

#!/usr/bin/python
# -*- coding: utf-8 -*- 

import urllib2 # extensible library for opening URLs
import time

# 인천지역 미세먼지 넣기 
url = 'http://www.airkorea.or.kr/index'

def time_print():
	now_unix = time.time()	# unix time
	now = time.localtime()	# local time

	now_year = now.tm_year	# year
	now_mon = now.tm_mon	# month
	now_day = now.tm_mday	# day
	now_hour = now.tm_hour	# hour 
	now_min = now.tm_min	# min
	now_sec = now.tm_sec	# sec

	print "======================="
	print now_unix	# unix time print
	print now
	print "======================="
	print now_year, now_mon, now_day, now_hour, now_min, now_sec	# time parsing
	print "======================="

def getData(buffers):
	a = buffers.split('<tbody id="mt_mmc2_10007">')[1]
	print a
	
	b = a.split('</tbody>')[0].replace('<tr>','').replace('</tr>','').replace('</td>','')
	print b

	c = b.split('<td>')
	print c[1]
	print c[2]

if __name__ == '__main__':

	time_print()
	page = urllib2.urlopen(url).read()
	#print page

	getData(page)

```
   
   
#### exercise 4
   - 주기적으로 특정 웹페이지(http://www.airkorea.or.kr) 클로링하여 openTSDB에 저장 (미세먼지)
   
#### exercise 5
   - 기상청 웹페이지(http://www.kma.go.kr/weather/lifenindustry/sevice_rss.jsp) 
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

# install lxml library
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

def time_print():
	now_unix = time.time()	# unix time
	now = time.localtime()	# local time

	now_year = now.tm_year	# year
	now_mon = now.tm_mon	# month
	now_day = now.tm_mday	# day
	now_hour = now.tm_hour	# hour 
	now_min = now.tm_min	# min
	now_sec = now.tm_sec	# sec

	print "======================="
	print now_unix	# unix time print
	print now
	print "======================="
	print now_year, now_mon, now_day, now_hour, now_min, now_sec	# time parsing
	print "======================="

if __name__ == '__main__':

	# time print function
	time_print()

	page = urllib2.urlopen(url).read()

	# fromstring : Parses an XML document or fragment from a string. 
	# Returns the root node (or the result returned by a parser target).
	xml_raw = fromstring(page)

	# processing temperature
	temp_process(xml_raw)

```

#### exercise 6
   - openTSDB API 호출하고 json 형식의 데이터를 파싱
```sh

#!/usr/bin/python

import sys
import urllib2
import json
import struct
import time
from datetime import datetime, timedelta

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

while 1 :
    t = time.localtime()
    tsec = t.tm_sec
    if tsec%10!=0 :
    	print tsec
    	time.sleep(0.8)
        
    else :
        endTimeUnix = time.time()
        startTimeUnix = endTimeUnix - 1800 

        startTime = datetime.fromtimestamp(startTimeUnix).strftime('%Y/%m/%d-%H:%M:%S')
        endTime = datetime.fromtimestamp(endTimeUnix).strftime('%Y/%m/%d-%H:%M:%S')

        #print startTime
        #print endTime
	metric = 'cc_100.test'
        url = 'http://127.0.0.1:4242/api/query?start=' + startTime + '&end=' + endTime + '&m=sum:' + metric
        #print url

        try:
            u = urllib2.urlopen(url)
        except:
	    raise NameError('url error')

        data = u.read()
	print data
        packets = dataParser(data)
        packet = packets.split(',')

        j=len(packet)

        v_s = packet[0].split(":")
        v_e = packet[j-1].split(":")

	print v_s[0], v_s[1]
        #print "cc.test %d %d" % ( endTimeUnix, v_e )
    	time.sleep(0.8)


```

#### Exercise 7
##### Restful 방식
