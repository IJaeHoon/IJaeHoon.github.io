 

# [AI] 8/3 실습

복습

에드커밋푸시

elem.send_keys("자전거") elem.send_keys(Keys.RETURN)

키입력

d.switch_to.frame(iframe)

따로 들어가는키

from selenium.webdriver.common.keys import Keys

- 데이터분석은 함수몇개쓰면됨

- 집어넣은 데이터가 뭐냐가 결국 결과를 크게 결정함
    
    [![](HTML%20import/Attachments/Untitled.png)](Untitled.png)
    
    동작
    
    [![](HTML%20import/Attachments/Untitled%201.png)](Untitled%201.png)
    

액션체인

Actionhains(d)

#마우스 움직이기, 위치기억 > 클릭 이벤트

buttons = d.find_elements(By.CSS_SELECTOR, ".xvy4d1p") search_button = buttons[2

```
ac = ActionChains(d)
ac.move_to_element(search_button)
		ac.click() # 클릭
			ac.perform() # 실행

```

### 실습

```
1. singer 테이블 만들기

    - id(Auto increment)

    - name(가수이름)

    - follower(팔로우 수)

2. song 테이블 만들기

    - id(Auto increment)

    - singer_id(FK)

    - title(제목)

    - album(앨범)

3. 멜론 TOP 100을 수집

    - 멜론 TOP 100에 존재하는 가수들을 singer 테이블에 저장

        - top100내의 가수목록을 selenium 등으로 수집해서 singer에 넣기

        - 가수의 중복을 없애야한다


    - 각 가수들의 곡은 song 테이블에 저장

    - 단, singer 테이블에 follower는 0으로 저장

4. 인스타그램에서 가수명으로 검색한 후 인증마크 달려있는 계정의 follower가져와서 데이터베이스의 follower 값 업데이트

5. pandas
```

1. singer 테이블에 가수 목록 중복제거해서 넣기

```
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.common.keys import Keys
from selenium.webdriver.common.action_chains import ActionChains
from openpyxl import Workbook
import pymysql
import requests
import time

# INSERT INTO 테이블2 (singer_id, song, title)
# SELECT id, '노래명', '제목' --
# FROM 테이블1;

# DB
db = pymysql.connect(host="localhost", port=3306, user="root", password="jen401018&", db="sba")
cursor = db.cursor()

# webdriver options
options = webdriver.ChromeOptions()
options.add_argument("--user-data-dir=/Users/sopung/Desktop/MyChrome")

# path
path = "/Users/sopung/Downloads/chromedriver_mac_arm64/chromedriver"
service = Service(executable_path=path)
d = webdriver.Chrome(service=service, options=options)

try:
    # 멜론 차트 top100
    d.get("https://www.melon.com/chart/index.htm")

    # 차트 top100영역
    area = d.find_element(By.CSS_SELECTOR, ".service_list_song")
    elem = area.find_elements(By.CSS_SELECTOR, "tbody > tr")
    singer_list = []

    # 가수 목록
    for e in elem:
        singer_list.append(e.find_element(By.CSS_SELECTOR, ".ellipsis.rank02").text.strip())
        title = e.find_element(By.CSS_SELECTOR, ".ellipsis.rank01").text.strip()
        album = e.find_element(By.CSS_SELECTOR, ".ellipsis.rank03").text.strip()
        # print(album)
        # print(title)
    # 가수 중복 제거
    singer_list = list(set(singer_list))

    # 리스트에서 가수를 SQL로 담아 DB에 삽입
    for singer in singer_list:
        sql = f"""
            insert into singer
            values(NULL, '{singer}', NULL)
        """
        cursor.execute(sql)
except Exception as e:
    print(e)

finally:
    d.close()
    d.quit()
    db.commit()
    db.close()
```

1. song 테이블에 singer 테이블의 id에 해당하는 singer_id, title, album 삽입하기

`from selenium import webdriver from selenium.webdriver.chrome.service import Service from selenium.webdriver.common.by import By from selenium.webdriver.common.keys import Keys from selenium.webdriver.common.action_chains import ActionChains from openpyxl import Workbook import pymysql import requests import time import traceback # 에러 정보를 가지고있는 라이브러리 # INSERT INTO 테이블2 (singer_id, song, title) # SELECT id, '노래명', '제목' -- # FROM 테이블1; # DB db = pymysql.connect(host="localhost", port=3306, user="root", password="jen401018&", db="sba") cursor = db.cursor() # webdriver options options = webdriver.ChromeOptions() options.add_argument("--user-data-dir=/Users/sopung/Desktop/MyChrome") # path path = "/Users/sopung/Downloads/chromedriver_mac_arm64/chromedriver" service = Service(executable_path=path) d = webdriver.Chrome(service=service, options=options) try: # 멜론 차트 top100 d.get("https://www.melon.com/chart/index.htm") # 차트 top100영역 area = d.find_element(By.CSS_SELECTOR, ".service_list_song") elem = area.find_elements(By.CSS_SELECTOR, "tbody > tr") singer_list = [] # 가수 목록 for e in elem: singer = e.find_element(By.CSS_SELECTOR, ".ellipsis.rank02").text.strip() title = e.find_element(By.CSS_SELECTOR, ".ellipsis.rank01").text.strip().replace("'", '"') album = e.find_element(By.CSS_SELECTOR, ".ellipsis.rank03").text.strip().replace("'", '"') # singer id에 해당하는 제목, 앨범 넣기 sql = f""" insert into song values(NULL, (select id from singer where name = '{singer}'), '{title}', '{album}'); """ cursor.execute(sql) except Exception as e: traceback.print_exc() finally: d.close() d.quit() db.commit() db.close()`

```
1. singer 테이블 만들기

    - id(Auto increment)

    - name(가수이름)

    - follower(팔로우 수)

2. song 테이블 만들기

    - id(Auto increment)

    - singer_id(FK)

    - title(제목)

    - album(앨범)

3. 멜론 TOP 100을 수집

    - 멜론 TOP 100에 존재하는 가수들을 singer 테이블에 저장

        - top100내의 가수목록을 selenium 등으로 수집해서 singer에 넣기

        - 가수의 중복을 없애야한다


    - 각 가수들의 곡은 song 테이블에 저장

    - 단, singer 테이블에 follower는 0으로 저장

4. 인스타그램에서 가수명으로 검색한 후 인증마크 달려있는 계정의 follower가져와서 데이터베이스의 follower 값 업데이트

5. pandas
```