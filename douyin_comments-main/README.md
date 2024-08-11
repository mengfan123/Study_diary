# 使用方式：运行douyin_comments.py文件，当页面出现弹窗验证时要快速手动验证，等待页面自行滑动，完成后保存在文件夹中，以下是可供调节的参数
```python
#想要爬取的个人主页
account_home_page= 'https://www.douyin.com/user/MS4wLjABAAAAKouSmCULyRPvwO2ECzsUljHEmlAxvRIJSy3Q30VEuu0'
#想要爬取的视频数量
video_num = 10
#翻阅的最大视频数量，为了控制程序运行时间
max_video_num = 100
#想要爬取的评论数量
comment_num = 10
#翻阅的最大评论数量，为了控制程序运行时间
max_comment_num = 50

```

# 如果出现错误可以看看issues，里面介绍了程序运行步骤。

# if there is an error, look at issues, which describes the procedure for running the program.



## 导入所需的模块

```python
import os
from selenium import webdriver
from selenium.webdriver.common.by import By
import time
```



## 抖音用户主页链接和需要获取评论的视频数量

```python
account_home_page= 'https://www.douyin.com/user/MS4wLjABAAAAABJTNtdE9bZKmIZfL_pR15F8X0VNK591ffRA9pXXZsw'
video_num = 10
comment_num = 20
video_or_comment = "video"
```



## 下拉函数，用于滚动网页并加载更多内容


```python
def drop_down(video_or_comment):
    prev_height = 0
    if video_or_comment == "video":
        curr_num = len(driver.find_elements(By.CSS_SELECTOR, '.Eie04v01'))
        curr_height = driver.execute_script("return document.documentElement.scrollHeight")
        driver.execute_script("window.scrollTo(0, document.documentElement.scrollHeight);")
        time.sleep(3)
        new_height = driver.execute_script("return document.documentElement.scrollHeight")
        new_num = len(driver.find_elements(By.CSS_SELECTOR, '.Eie04v01'))
        max_num = 10 * video_num


    elif video_or_comment == "comment":
        curr_num = len(driver.find_elements(By.XPATH, '//*[@id="douyin-right-container"]/div[2]/div/div[1]/div[5]/div/div/div[4]/*/div/div[2]/div'))
        curr_height = driver.execute_script("return document.documentElement.scrollHeight")
        driver.execute_script("window.scrollTo(0, document.documentElement.scrollHeight);")
        time.sleep(3)
        new_height = driver.execute_script("return document.documentElement.scrollHeight")
        new_num = len(driver.find_elements(By.XPATH, '//*[@id="douyin-right-container"]/div[2]/div/div[1]/div[5]/div/div/div[4]/*/div/div[2]/div'))
        max_num = 10 * comment_num
    else:
        raise ValueError
    
    ele_num = new_num - curr_num
    
    while True:
        # Get current scroll height
        curr_height = driver.execute_script("return document.documentElement.scrollHeight")
        
        # Scroll down the page
        driver.execute_script("window.scrollTo(0, document.documentElement.scrollHeight);")
        
        # Wait for some time to let the page load
        time.sleep(3)
        
        # Get the new scroll height
        new_height = driver.execute_script("return document.documentElement.scrollHeight")

        new_num = new_num + ele_num
        
        # Break if we have reached the bottom of the page
        if (new_height == curr_height == prev_height) or (new_num >= max_num):
            break
        
        # Update the previous scroll height
        prev_height = curr_height
```


## 将评论中的点赞数字符串转化为数字

```python
def str2num(s):
    try:
        if s[-1] == '万':
            n = int(float(s[:-1]) * 10000)
        else:
            n = int(s)
    except:
        n = 0
    return n
```



## 提取评论中的文本和点赞数

```python
def extract(comment_splitted):
    try:
        text = comment_splitted[1]
        if comment_splitted[-1] == '回复':
            like = str2num(comment_splitted[-3])
        else:
            like = str2num(comment_splitted[-4])
        
    except:
        text = ""
        like = 0
    return text, like
```

​        

## 创建 Chrome 浏览器实例并打开用户主页

```python
driver = webdriver.Chrome()
driver.get(account_home_page)
time.sleep(10)
drop_down("video")
driver.implicitly_wait(10)
```



## 获取用户 ID 并创建以其命名的文件夹，用于存储评论

```python
account_id = driver.find_element(By.XPATH, '//*[@id="douyin-right-container"]/div[2]/div/div/div[1]/div[2]/div[1]/h1/span/span/span/span/span/span').text
account_path = './' + account_id
if not os.path.exists(account_path):
    os.mkdir(account_path)
```




## 获取热门视频的链接和点赞数，并按照点赞数排序

```python
lis = driver.find_elements(By.CSS_SELECTOR, '.Eie04v01')
url_likes = []
for li in lis:
    url = li.find_element(By.CSS_SELECTOR, 'a').get_attribute('href')
    video_like = str2num(li.find_element(By.XPATH, './div/a/div/span/span').text)
    url_likes.append([url, video_like])

url_likes.sort(key=lambda x : x[-1], reverse=True)
```



## 遍历热门视频的链接，滚动网页加载评论，创建文件写入评论
```python
num = 0
for url, video_like in url_likes[:min(video_num, len(url_likes))]:
    num += 1
    driver.get(url)
    time.sleep(3)
    drop_down("comment")

    comments = driver.find_elements(By.XPATH, '//*[@id="douyin-right-container"]/div[2]/div/div[1]/div[5]/div/div/div[4]/*/div/div[2]/div')
    text_likes = []
    for comment in comments:
        comment_splitted = comment.text.split(sep='\n')
        print(comment_splitted)
        text_likes.append(extract(comment_splitted))

    text_likes.sort(key= lambda x : x[-1], reverse=True)
    try:
        video_name = driver.find_element(By.XPATH, '//*[@id="douyin-right-container"]/div[2]/div/div[1]/div[3]/div/div[1]/div/h2/span').text.split(sep=' ')[0]
    except:
        video_name = 'unknown video'
    comment_path = account_path + '/' + str(num) + '.txt'
    f = open(comment_path, 'w',encoding='utf-8')
    f.write('视频名称：' + video_name + '\n')
    f.write('视频点赞数：' + str(video_like) + '\n')
    f.write('视频地址：' + url + '\n')
    f.write('评论：' + '\n')
    for i in range(min(comment_num, len(text_likes))):
        f.write('点赞数：' + str(text_likes[i][-1]) + '  ' + text_likes[i][0] + '\n')
    f.close()
```

