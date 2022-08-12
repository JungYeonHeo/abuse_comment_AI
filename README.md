<div align="center">

  # 🤬 abuse_comment_AI
  **`악성 댓글을 탐지 AI 웹 서비스`**
</div> 

<br/>

## Crawring
![사이트선정](https://user-images.githubusercontent.com/94504613/184283819-50396137-5768-4123-bec0-e494ea2869e1.jpg)

![디씨](https://user-images.githubusercontent.com/94504613/184283816-cf209947-b621-4cd0-b5ec-23af91e19a20.jpg)

- 위 통계 표를 바탕으로 우리나라 커뮤니티 사이트 중 가장 큰 커뮤니티 사이트인 **`dcinside`** 선정하였다.
- **`hit 갤러리`** 와 **`실시간 베스트`** 글에 대한 크롤링 진행하였다.
- 아래 코드를 바탕으로 **`크롤링`** 을 하여 **`csv`** 파일에 저장하였다.

```python
from bs4 import BeautifulSoup
from urllib.request import Request, urlopen
import pandas as pd
import time
from selenium import webdriver

class Service:
    
    def getComment(self):       
        comment_list = []
        
        for page in range(1, 16):
            url = 'https://gall.dcinside.com/board/lists/?id=hit'
            url += '&page=' + str(page)
            print('>> page: ' + str(page))     
            
            try: 
                headers = {'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/83.0.4103.116 Safari/537.36'}
                request = Request(url, headers=headers) 
                response = urlopen(request)
                html = response.read()
                soup = BeautifulSoup(html, 'lxml')
                links = soup.select('#container > section.left_content > article:nth-child(3) > div.gall_listwrap.list > table > tbody > tr')  
                
                for l in links:
                    a_link = l.select_one('td.gall_tit.ub-word > a:nth-child(1)').get('href')  
                    
                    if a_link != 'javascript:;':
                        baseurl = 'https://gall.dcinside.com'
                        url2 = baseurl + a_link
                        print('>> ' + url2)

                        try: 
                            options = webdriver.ChromeOptions() 
                            # options.add_argument('headless') 
                            options.add_argument('disable-gpu') 
                            options.add_experimental_option("excludeSwitches", ["enable-logging"])
                            options.add_argument('user-agent=Mozilla/5.0 (Windows NT 6.3; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/99.0.4692.99 Safari/537.36, Referrer-Policy=no-referrer, strict-origin-when-cross-origin')
                            driver = webdriver.Chrome('chromedriver', options=options)
                            driver.get(url2)
                            time.sleep(5)
                            soup2 = BeautifulSoup(driver.page_source, 'lxml')
                            comments = soup2.select('#focus_cmt > div.comment_wrap > div.comment_box > ul > li')
                            
                            for c in comments:
                                try: 
                                    text = c.find('p', attrs={'class': 'usertxt'}).text
                                    if 'dc App' in text:
                                        text = text.split('-')[0].strip('')
                                    print(text)
                                    comment_list.append(text) 
                                except Exception as e:
                                    pass
                                
                        except Exception as e:
                            print(e)
                            pass
                        finally:
                            driver.quit()

            except Exception as e:
                print(e)
                pass
        
        data = pd.DataFrame(comment_list, columns=['댓글'])
        data.to_csv('댓글.csv', index = False, encoding='utf-8-sig')


if __name__ == "__main__": 
    s = Service()
    res = s.getComment()
```

<br>

## Labeling
인기 있는 게시물의 댓글을 크롤링하여 욕설인 것과 아닌 것을 손수 라벨링하였다.
https://user-images.githubusercontent.com/94504613/184284590-94da7a90-e01f-454f-8263-646161fb80d4.mp4

## Preprocessing
데이터 전처리로 다음과 같은 도구를 사용하였다.
  <p>
    <img src="https://img.shields.io/badge/KONLPY-d80000?style=flat&logo=Konlpy&logoColor=white"/>
    <img src="https://img.shields.io/badge/TensorFlow-f7811a?style=flat&logo=TensorFlow&logoColor=white"/>
  </p>

<br>

전처리 과정은 아래와 같이 진행하였다.
![전처리](https://user-images.githubusercontent.com/94504613/184285280-a55f88cc-aec4-4adf-b37f-b4cf14b0cfca.png)
<br> 

## Model
모델 도구로 keras의 tensorflow를 사용하였다.
![모델도구](https://user-images.githubusercontent.com/94504613/184285570-ec4834dc-c361-471f-a0dc-b5c6fe098dbf.jpg)

자연어 처리가 들어가야 해서 **`LSTM`** 을 사용하였고 모델 구조는 다음과 같이 선정하였다.
![모델구조](https://user-images.githubusercontent.com/94504613/184285627-457f8150-6c9d-43da-8a88-97dac02d2579.png)