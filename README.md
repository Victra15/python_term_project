# python_term_project
웹 크롤링을 이용한 장바구니 최저가 탐색
프로젝트 목표 및 내용

웹 크롤링 작업을 활용해서 내가 원하는 물품들을 최저의 가격으로 살 수 있는 조건을 찾아주는 프로그램 제작

주제 선정 이유 또는 이 프로젝트의 필요성

인터넷을 통해 상품을 구매할 때, 같은 상품을 구매하는 경우에 네이버나 다른 쇼핑몰 사이트 등에서 찾은 상품의 가격이 다르고,
네이버 쇼핑을 통해 찾아낸 물품의 최저가를 보고 링크를 타고 들어가 봤더니 네이버 쇼핑에 표기된 최저가와 다른 경우도 있기도 하고,
또 네이버 쇼핑에서 링크를 통해 쇼핑몰 사이트에 들어갔으나, 쇼핑몰 사이트 내부에서 세부 옵션을 선택해야지만 진짜 가격이 나오는 경우도 있다.
그리고 동시에 여러 개의 상품을 구매하는 경우에, 같은 판매자로부터 구매하는 상품은 하나의 배송비로 묶여서 배송이 되기도 한다.
하지만 이런 모든 조건들을 고려하여 물품을 구매하는 것은 소비자에게 있어서 상당한 시간을 소모하는 일이기 때문에, 이러한 소비자의 수고를 덜기 위한 프로그램을 제작해 보고자 하였고,
파이썬 웹 크롤링 모듈인 beautifulsoup4, selenium 등의 사용법을 익혀 실용적인 프로그램을 만들어 보고자 이러한 주제를 선정하게 되었다.

데이터 획득

selenium을 활용하여 네이버 쇼핑 검색결과를 가져오고 가져온 상품들 각각의 상세페이지에서

상품에 대한 상세정보(상품명, 가격, 배송비, 상품옵션, URL, 묶음배송상품정보URL) 등을

가져옴

구현 내용

프로그램 구현에 필요한 모듈인 beautifulsoup, selenium module을 설치한 후,

!pip install BeautifulSoup
!pip install selenium

프로그램 구현에 필요한 selenium, beautifulsoup, time, request, re module을 각각 import한다.

import requests
import re
import time
import urllib.request

from selenium import webdriver
from selenium.webdriver.common.keys import Keys
from selenium.common.exceptions import NoSuchElementException
from bs4 import BeautifulSoup

상품 상세페이지의 정보를 클래스 형태로 저장하기 위해 Item 클래스를 만든다.

class Item:
    """
    상품 상세페이지의 정보를 저장하는 클래스
    """
    def __init__(self):
        self.name = ""
        self.price = 0
        self.option = []
        self.bundle_shipping_item_page_URL = ""
        self.URL = ""
        self.shipping_charge = 0

    def setName(self,givenName):
        self.name = givenName
    def setPrice(self,givenPrice):
        self.price = givenPrice
    def setOption(self,givenOption):
        self.option = givenOption
    def setBundleItemPageURL(self,givenBundleURL):
        self.bundle_shipping_item_page_URL = givenBundleURL
    def setURL(self,givenURL):
        self.URL = givenURL
    def setShippingCharge(self,givenShippingCharge):
        self.shipping_charge = givenShippingCharge

    def getName(self):
        return self.name
    def getPrice(self):
        return self.price
    def getOption(self):
        return self.option
    def getBundleItemPageURL(self):
        return self.bundle_shipping_item_page_URL
    def getURL(self):
        return self.URL
    def getShippingCharge(self):
        return self.shipping_charge

    def __str__(self):
        msg = "이름: {}\n가격: {}\n배송비: {}\n옵션정보: {}\n묶음배송상품정보URL: {}\n상품URL: {}".format(self.name,self.price,self.shipping_charge,self.option,self.bundle_shipping_item_page_URL,self.URL)
        return msg

우선, 찾고자하는 정확한 상품명들을 입력받아 list 형태로 저장한 후,

네이버 쇼핑 검색창에 차례대로 입력하는 코드를 만들고, 검색 결과를 수집한다.

이때, 네이버 쇼핑 검색 사이트는 스크롤을 내리지 않으면 모든 정보를 보여주지 않으므로

selenium의 webdriver모듈을 이용해서 스크롤을 내리는 기능을 구현한 후,

현재 페이지의 html문서를 가져온 후, beautifulsoup모듈로 해석한다.

이때, 상품에 대한 정보는 {"검색어":Item 객체 List} 의 dictionary로 저장한다.

# 찾고자 하는 상품명들을 정확히 입력 후, 검색 결과를 가져옴
path = "C:/Users/82106/Downloads/chromedriver.exe"
driver = webdriver.Chrome(path)
search_keyword_list = []
while(True):
    search_keyword = input(prompt = "찾을 품목의 정확한 이름을 입력하세요.(종료하려면 0) : ")
    if(search_keyword == "0"):
        break
    search_keyword_list.append(search_keyword)

search_item_list = {}

for search_keyword in search_keyword_list:
    driver.get("https://shopping.naver.com/")
    search_box = driver.find_element_by_name("query")

    search_box.send_keys(search_keyword)
    search_box.send_keys(Keys.RETURN)

    scroll_down_page()
    search_item_list[search_keyword] = find_item_info_from_naver_shopping_search()

구매 옵션, 묶음상품정보가 있는 URL을 가져오기 위해 selenium을 이용해서

각각의 상품 상세페이지에 들어가 동적으로 데이터를 수집해야할 필요가 있으므로

정보를 볼 상품의 수를 입력받은 후, 입력받은 숫자만큼 상세 페이지에서 구매 옵션, 묶음상품정보 페이지URL을 가져온다.

# 검색된 상품 중 일부 상품의 상세페이지에 입장해서 상세정보 조회
input_num = int(input(prompt="정보를 볼 상품의 수를 입력하세요: "))

for search_keyword in search_keyword_list:
    for idx in range(0,input_num):
        URL = search_item_list[search_keyword][idx].getURL()
        item = search_item_list[search_keyword][idx]
        driver.get(URL)
        if("smartstore" in driver.current_url):
            find_item_info_from_naver_smartstore(driver, item)
#         elif("auction" in drive.current_url):
#             find_item_info_from_auction(driver, item)
#         elif("gmarket" in drive.current_url):
#             find_item_info_from_gmarket(driver, item)
#         elif("interpark" in drive.current_url):
#             find_item_info_from_interpark(driver, item)
#         elif("tmon" in drive.current_url):
#             find_item_info_from_tmon(driver, item)
#         elif("lotteon" in drive.current_url):
#             find_item_info_from_lotteon(driver, item)
#         elif("wemakeprice" in drive.current_url):
#             find_item_info_from_wemakeprice(driver, item)
#         elif("11st" in drive.current_url):
#             find_item_info_from_11st(driver, item)
#         elif("coupang" in drive.current_url):
#             find_item_info_from_coupang(driver, item)

상세 정보를 조회한 데이터들을 구매 결정 후보군으로 입력하고,

각각의 검색 키워드별로 구매 결정 후보군에서 배송비와 가격의 합이 가장 작은 Item객체를

{"검색 키워드":Item객체} 의 dictionary 형태로 모든 검색 키워드에 대해서

먼저 저장하고 -- (1) 아래 코드의 recommand_best_purchasing()함수를 사용해서

구매 결정 후보군에서 묶음배송상품을 고려했을때, (1)번에서 저장한 dictionary의

Item 가격의 총합보다 작은 dictionary를 만들 수 있으면 그것을 list로 만든 후 보여준다.

# 상품 정보를 조회한 상품들을 구매 결정 후보군으로 입력함
# 구매 결정 후보군의 묶음배송상품 정보를 조회한 후, 최종적으로 최저가로 구매 가능한 조합을 추천

purchase_candidate_list = {}
for search_keyword in search_keyword_list:
    purchase_candidate_list[search_keyword] = search_item_list[search_keyword][0:input_num]

recommanded_purchase_list_candidate = recommand_best_purchasing(purchase_candidate_list)




구현을 위해 별도로 만든 함수들은 다음과 같다.

def scroll_down_page():
def move_next_page_naver_shopping_search():
def find_item_info_from_naver_shopping_search():
def check_exists_by_class_name(driver, class_name):
def find_item_info_from_naver_smartstore(drive, item):
def find_item_info_from_auction(drive, item)
def find_item_info_from_gmarket(drive, item)
def find_item_info_from_interpark(drive, item)
def find_item_info_from_tmon(drive, item)
def find_item_info_from_lotteon(drive, item)
def find_item_info_from_wemakeprice(drive, item)
def find_item_info_from_11st(drive, item)
def find_item_info_from_coupang(drive, item)
def find_bundle_shipping_item_info_from_naver_smartstore(driver, new_recommanded_purchase_list, non_bundled_purchase_list)
def compare_total_price(item1, item2):
def find_minimum_price_item(item_list):
def calc_total_price(recommand_list):
def recommand_best_purchasing(purchase_candidate_list):

각 함수에 대한 간략한 설명은 구현 코드에 정리되어 있다.

구현 결과

시험 구동을 위해서

[신라면, 햇반, 데체코 파스타면, 허니버터칩] 이 네개의 검색 키워드를 입력 후,

검색 결과를 가져옴

찾을 품목의 정확한 이름을 입력하세요.(종료하려면 0) : 신라면

찾을 품목의 정확한 이름을 입력하세요.(종료하려면 0) : 햇반

찾을 품목의 정확한 이름을 입력하세요.(종료하려면 0) : 데체코 파스타면

찾을 품목의 정확한 이름을 입력하세요.(종료하려면 0) : 허니버터칩

찾을 품목의 정확한 이름을 입력하세요.(종료하려면 0) : 0

이후 검색 결과가 정상적으로 크롤링 되었는지 print()함수를 이용해 확인한 후,

상세페이지에서 상세정보를 조회할 데이터의 크기를 입력함,

정보를 볼 상품의 수를 입력하세요: 10

최종적으로 묶음배송상품 정보를 고려해 더 저렴하게 살 수 있는 정보들을 보여줌

list  1

이름: 농심 신라면 봉지 120g
가격: 490
배송비: 3000
옵션정보: ['농심 신라면 봉지 120g']
묶음배송상품정보URL: https://smartstore.naver.com/plan88/bundle/20904313?prevProdId=4852268901
상품URL: https://cr.shopping.naver.com/adcr.nhn?x=mYITTcPazqKoL6qXEpjW2f%2F%2F%2Fw%3D%3DsMOkpFw7SBRsZA%2F%2BR4wZUL8k14rQVQP2TMQKPsFAR027sxnCeUdoJKwdm6dccG%2B8JckFNyAWU9sAYFuaaFcVrHUDzxJH2kXM%2BZO%2Bc9R1zVHqMmg8e3fyYqu%2Fmyi%2BD%2FAJNb2Ay4H9kAJIceRWCy88UT0S%2BortKzfQDWBF5GV%2FKfqEhJv3ljJo79VVIpj6%2Fm3WO4g5cmqoNway0KGtbsWWa8M2HYWYcQlZP02BVDdc68f1WEXeXwSaubPu0qUaxQfPPa%2BpHa6ewb0%2FDpcGez9%2B2MszRsUfNrHrMFzKVIupmN966lTZluAoM%2B%2FfISgLZPwCwVIBheONaGO%2FLj6NdOjj8V68z6W%2BL084AhZ0WquZHS%2BaJzp6k6d%2FmX2FIAug3P8irXXw%2Byg0cs9wsP3ZQy5x0mym4TEQPhrcCSdiSwqFMbTCLx6sB9UXluBYtA0JpFp6%2BU%2F3%2FM98dCBsmwBMraIHUHhlOzFskaCyn0%2FpeDgLpbwAUCbq93rnvxiWBHn8P1Y2k2oJDXx9uDahl9WuMZaPEYZSPjR5%2BPfZ1n55jLIm1GYGf8msIlx3X6IrdTpGZ2GunTBFWmC%2Fe%2BUWOZkhWtfc2dO70DqsmGdJT9uS76snzkOY%3D&nvMid=82396792200&catId=50002385

이름: CJ 햇반 200g
가격: 710
배송비: 0
옵션정보: []
묶음배송상품정보URL: 
상품URL: https://smartstore.naver.com/plan88/products/5093208841

이름: 데체코 스파게티니 11호 1kg 스파게티 파스타
가격: 3190
배송비: 2500
옵션정보: []
묶음배송상품정보URL: 
상품URL: https://cr.shopping.naver.com/adcr.nhn?x=Gi8jVqFnwQp2i3AgqxZ%2Fsv%2F%2F%2Fw%3D%3Dsggv%2FWh1VugdK8x2Jl2kIIML7GW%2Bpy0NM3zqkJsCLhHT6HjDSbOthmPZKIm79xy3irQv35oOaKFBwsI3gf6TQqkgwWBqxfotKZa%2B%2B0y7QHYFZzKcxqzxQDmJoD%2BH85fzGYKnoEaM0p%2B8vlH3sXxW%2BoQHV0j8OFIb%2B8FY1qmBefUK28FN0Ov4xXtRrN3ou87DsiuwR%2BJD5dvGHrlzyJHCXUQbKapXDMDfB1vYKuD1%2FYotokdOfztY1eIB7pBHiqFm8FAf4ge8T8pp%2BU%2BqMBvriJFz9NBBeqgTWjmp2ixpxdQ7It9ELSj9ZgOwWGT6cXiS3A4ngWLwD1eOq0sUfCk5avzbkiGtcBklHzXhBd4YxyeG73eFezMGjFxwNkdK3g9hasCvAlcFT0kuNQSfoqqEY4b7kqXz3WwNt%2BUk1l%2BPbLrU4LUmvh5hbP3sHwizOcmtyRFadIoK8rzHqu58MsZGpJ00%2Fem11SwCQLmLBUicJHZyBYkv8YrjrLdvFgz7EDtEhF7%2Fl9sW8L9r9KQvg2LCdxXf%2FiyShskMkXyxueGBPvh3pZ55jAGMLxMI6%2FLdbIjY5H9X4KUAEAHo%2FTc3ezX%2Fp34biYwSHZQdAMrVc0CRb%2BEzWuEPNVotwRBzmGNdKc7DtWAWVMoU97bUMucIGoKbszyBiw8eR93Hxkm8iHKUaEQYxtZkJRhKnMmfAN6NeKOnX&nvMid=9272401042&catId=50002389

이름: 해태제과 허니버터칩 60g
가격: 680
배송비: 2500
옵션정보: []
묶음배송상품정보URL: 
상품URL: https://cr.shopping.naver.com/adcr.nhn?x=MAhb5hCW7V%2FcLABL16BKAv%2F%2F%2Fw%3D%3DshXFLNwY1znsmz8CZMZTVJHH5NdJSQjr9ob%2Fm%2BjBGvk5eoJVWuWAiswwZ0vcLZJvPdCrJ683G8DUl0C2KjZ6q1XNGQENQ5vBuQ3xaoJqXuRyXDpm0XqfWHnRrjIr8tJtn7yKBcpDBrn4Rd0TquNIx9axUTfizNciQK%2B%2FJMo5kHrlWrxaEGYqB0DQc4FejQVgOZoNgu9yW1jjri7kMaD3fE78YbuxiKQ2YbMTawsbPs%2BzYlNi9h%2FvafBXu%2FJBLTdEaw6ZjbtkbN7RD61dGUzg7G441ZK8P3rauGK1tOXUp%2FMxwG8kKjfXzoVPhcPqcKqXHSm15Qi4MAGLgDc7o6JVo%2BzIv2dRDg8nl%2B99plxhOITk39MV4VOCZHMYEwZKEV4QzCoy%2BzT4S3YVy%2BDwGyg3L1nYy7rW3gNpg3MV4Gyu%2FqTtH7tinJmSX31i6Pda1uY4w69UJmxswHmtXada2iFu948s2hprdNWr38UNyr3SSTNcwbA0Hw92CknM7sxXW9vIns0LpLNpm6gwqb%2FHtDD%2Bxs5hz%2FypdDnr73yoeZcVlYBcLrOsamngGmhJsfT13%2FKU5am%2BAGKKN%2F2WhB8LwfYJM7imjsFSH5hSSXButiq1ukCYldIWWx6olv4P3LW5p1MTu&nvMid=82602283496&catId=50001998

가격 총합 :  13070

list  2

이름: CJ 햇반 200g
가격: 560
배송비: 3000
옵션정보: ['cj 햇반200g']
묶음배송상품정보URL: https://smartstore.naver.com/plan88/bundle/20904313?prevProdId=5093208841
상품URL: https://cr.shopping.naver.com/adcr.nhn?x=6no1id5HTgUjjIIsJIUKGf%2F%2F%2Fw%3D%3DsHYq1hHWGHFy4VhIAvr4m1PVVUjd5iqM4RUTzEM0ikF%2B%2BMed5MxUhQuU9%2Fdvk8SXBSZe3kyXMqPpIbdF5AiFcI2rsxll5TVpBY3Qe%2BVw%2BOab8hMLqKzv4zOnPJn%2BTibkEMBbAUTLhEczrfo6o%2BFCYJ9koDZhZTyE2PqJzLGA4wO6CNGPg14xKLCfbXBkYNS8Ku7G5sDaUhbXVnYYHHbMO435hkZWKqBmlEbUVsPbEovDffisv5tHfoGFh7RH%2BtSaODg1f9vTf3Iez1Rjn58zRmqK4sAS8pf5foRuTSmIdbY6IfkfvyBhc%2BFxvBYB0PoSDTJ9IQKY7nJXSUZEKS4cX5KqUGokz%2F%2FRFskJQiz%2B2BppcG753OTh0q0ZpdQt58PSB2FjrC5flfok6BYqeoW0IJA7bOBkxyuu11j7tW8mMn0iSa1hG3dFTd9Rb%2FonECvgFmc4lOZ9f7%2FKVqDWlXBnPmEn0reKCX5nV9FAeHFCFizByvgU0Vm27n0wARWRxdqgciDTfUcGpRvY2mDclp%2FyW8vRcZC2yHm9Sy4lZYCcSIQ2WBw2R%2FOJXosmyQRcC%2B7z5&nvMid=82637730629&catId=50001891

이름: 농심 신라면 봉지 120g
가격: 810
배송비: 0
옵션정보: []
묶음배송상품정보URL: 
상품URL: https://smartstore.naver.com/plan88/products/4852268901

이름: 데체코 스파게티니 11호 1kg 스파게티 파스타
가격: 3190
배송비: 2500
옵션정보: []
묶음배송상품정보URL: 
상품URL: https://cr.shopping.naver.com/adcr.nhn?x=Gi8jVqFnwQp2i3AgqxZ%2Fsv%2F%2F%2Fw%3D%3Dsggv%2FWh1VugdK8x2Jl2kIIML7GW%2Bpy0NM3zqkJsCLhHT6HjDSbOthmPZKIm79xy3irQv35oOaKFBwsI3gf6TQqkgwWBqxfotKZa%2B%2B0y7QHYFZzKcxqzxQDmJoD%2BH85fzGYKnoEaM0p%2B8vlH3sXxW%2BoQHV0j8OFIb%2B8FY1qmBefUK28FN0Ov4xXtRrN3ou87DsiuwR%2BJD5dvGHrlzyJHCXUQbKapXDMDfB1vYKuD1%2FYotokdOfztY1eIB7pBHiqFm8FAf4ge8T8pp%2BU%2BqMBvriJFz9NBBeqgTWjmp2ixpxdQ7It9ELSj9ZgOwWGT6cXiS3A4ngWLwD1eOq0sUfCk5avzbkiGtcBklHzXhBd4YxyeG73eFezMGjFxwNkdK3g9hasCvAlcFT0kuNQSfoqqEY4b7kqXz3WwNt%2BUk1l%2BPbLrU4LUmvh5hbP3sHwizOcmtyRFadIoK8rzHqu58MsZGpJ00%2Fem11SwCQLmLBUicJHZyBYkv8YrjrLdvFgz7EDtEhF7%2Fl9sW8L9r9KQvg2LCdxXf%2FiyShskMkXyxueGBPvh3pZ55jAGMLxMI6%2FLdbIjY5H9X4KUAEAHo%2FTc3ezX%2Fp34biYwSHZQdAMrVc0CRb%2BEzWuEPNVotwRBzmGNdKc7DtWAWVMoU97bUMucIGoKbszyBiw8eR93Hxkm8iHKUaEQYxtZkJRhKnMmfAN6NeKOnX&nvMid=9272401042&catId=50002389

이름: 해태제과 허니버터칩 60g
가격: 680
배송비: 2500
옵션정보: []
묶음배송상품정보URL: 
상품URL: https://cr.shopping.naver.com/adcr.nhn?x=MAhb5hCW7V%2FcLABL16BKAv%2F%2F%2Fw%3D%3DshXFLNwY1znsmz8CZMZTVJHH5NdJSQjr9ob%2Fm%2BjBGvk5eoJVWuWAiswwZ0vcLZJvPdCrJ683G8DUl0C2KjZ6q1XNGQENQ5vBuQ3xaoJqXuRyXDpm0XqfWHnRrjIr8tJtn7yKBcpDBrn4Rd0TquNIx9axUTfizNciQK%2B%2FJMo5kHrlWrxaEGYqB0DQc4FejQVgOZoNgu9yW1jjri7kMaD3fE78YbuxiKQ2YbMTawsbPs%2BzYlNi9h%2FvafBXu%2FJBLTdEaw6ZjbtkbN7RD61dGUzg7G441ZK8P3rauGK1tOXUp%2FMxwG8kKjfXzoVPhcPqcKqXHSm15Qi4MAGLgDc7o6JVo%2BzIv2dRDg8nl%2B99plxhOITk39MV4VOCZHMYEwZKEV4QzCoy%2BzT4S3YVy%2BDwGyg3L1nYy7rW3gNpg3MV4Gyu%2FqTtH7tinJmSX31i6Pda1uY4w69UJmxswHmtXada2iFu948s2hprdNWr38UNyr3SSTNcwbA0Hw92CknM7sxXW9vIns0LpLNpm6gwqb%2FHtDD%2Bxs5hz%2FypdDnr73yoeZcVlYBcLrOsamngGmhJsfT13%2FKU5am%2BAGKKN%2F2WhB8LwfYJM7imjsFSH5hSSXButiq1ukCYldIWWx6olv4P3LW5p1MTu&nvMid=82602283496&catId=50001998

가격 총합 :  13240
결론

본 프로젝트를 진행하면서, 쇼핑몰 사이트별로 데이터를 수집 할 때, 변수가 너무 많다는 것을

알게 되었다. 예를 들면, 같은 상품을 취급하는 상품 상세 페이지인데도 A 페이지는 상품명에

상품의 수량과 이름이 모두 명시되어있고, B 페이지는 상품명에는 상품 이름만 기재해놓고,

수량에 대한 정보는 구매 옵션으로 지정해 놓고, C 페이지는 상품명에 비슷한 제품군에

해당하는 모든 제품들을 기재해 놓은뒤(ex.인스턴트 라면 모음전) 세부 옵션을 통해 수량과

원하는 상품에 대한 가격 정보를 알 수 있게 해놓은 경우가 있었다. 그리고 쇼핑몰 사이트별로

묶음배송에 대한 정책도 다르고, 구현해놓은 사이트의 구조가 각기 달라서 쇼핑몰 사이트별로

대응하는 코드를 만드는 데에는 시간적, 기술적인 한계가 있었다. 그래서 본 프로젝트는

네이버쇼핑과 네이버 스마트스토어 사이트에 대해서만 동작하도록 구현하였고, 또한 크롤링을

할 때, 모든 변수에 대응해서 정확한 검색 결과를 가져올 수가 없었으므로, 검색 결과를 통해

나온 데이터 하나하나가 사용자가 원하는 정확한 상품이라고 가정하고, 크롤링을 통해 얻은

쇼핑몰 검색 결과에서 가장 낮은 가격의 조합을 찾는 데에 초점을 맞추고 코드를 구성하였다.

그리고 마지막에 묶음배송 상품 정보를 고려하는 부분에서 당초의 목적은 모든 묶음배송 상품

정보를 확인 한 후에 그 정보들을 바탕으로 가장 저렴하게 검색 상품들을 살 수 있는 조건을

제일 저렴한 순으로 구하는 코드를 만드려고 했지만 알고리즘을 구현하는데에 있어 복잡한

부분이 있어서 하나의 묶음 배송 정보 페이지에서 각 상품의 최저가를 합친 가격보다 싸게

살 수 있는 조건을 발견하면 가져와서 list에 저장하여 모두 보여주고, 사용자가 이 정보를

활용하게 만드는 방식으로 바꾸었다. 이렇게 함으로써 사용자의 편의성은 조금 떨어졌지만

사용자가 좀 더 나은 구매 조건을 알 수 있다는 점에서는 원하는 결과를 얻었다.

그리고 최종적으로 나온 결과를 통해 웹페이지의 변수에 대응해서 사용자가 원하는 정보를

정확히 골라 크롤링 할 수 있다면 좀 더 나은 프로그램을 만들수 있을거라 예측이 되었다.

오픈소스 활용한 부분

거의 모든 함수와 대부분의 코드에 selenium 라이브러리와 beautifulsoup라이브러리,

request 라이브러리를 활용했다.

참고 문헌

python beautifulsoup4 모듈 사용법에 대한 유튜브 강의

https://www.youtube.com/watch?v=yQ20jZwDjTE&list=PLR1BBBjlA9ZC94RtCbMnZ6meY_WzOqmiV&index=8&t=6217s

selenium documentation

https://www.selenium.dev/documentation/en/

selenium scroll down 하는 방법

https://hello-bryan.tistory.com/194

how to check element exists in selenium

https://stackoverflow.com/questions/45695874/check-if-element-exists-python-selenium/51534196
