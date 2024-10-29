```python
import time
from selenium import webdriver
from selenium.webdriver.common.by import By
import base64
import requests

# 配置参数
url = "http://172.16.8.252:30001/"  # web测试项目的url
username = "admin"  # 登录账号
password = "Passw0rd@MY"  # 登录密码
image_path = "D:/captcha_image/captcha.png"  # 验证码保存路径
api_url = "http://localhost:11434/api/generate"  # AI大模型请求接口地址

def send_captcha_request(image_path, api_url, model="minicpm-v:8b-2.6-q8_0",
                        prompt="Please extract the letters accurately. Please show only uppercase letters. Do not show numbers, special punctuation marks or other characters"):
    """
    发送验证码图片到AI模型进行识别，并返回识别结果
    """
    with open(image_path, "rb") as image_file:
        base64_image = base64.b64encode(image_file.read()).decode('utf-8')
    
    data = {
        "model": model,
        "prompt": prompt,
        "stream": False,
        "images": [base64_image]
    }
    
    while True:
        try:
            response = requests.post(api_url, json=data)
            if response.status_code == 200:
                data = response.json()
                if 'response' in data and data['response'].strip():
                    extracted_text = data['response'].replace(" ", "")
                    print(f"识别到的验证码: {extracted_text}")
                    return extracted_text
                else:
                    print("AI模型未返回有效的验证码，请重试。")
            else:
                print(f"错误: 接收到状态码 {response.status_code}")
        except Exception as e:
            print(f"请求出错: {e}")
        
        time.sleep(2)  # 等待2秒后重试

def login(url, username, password):
    """
    使用Selenium自动化登录，并在验证码识别失败时不断重试
    """
    # 初始化浏览器
    options = webdriver.ChromeOptions()
    options.add_argument("--start-maximized")
    driver = webdriver.Chrome(options=options)
    
    try:
        driver.get(url)
        time.sleep(5)  # 等待页面加载
        
        while True:
            # 输入用户名
            username_field = driver.find_element(By.XPATH, '//*[@id="app"]/div/div/div[1]/div[2]/div[2]/form/div[1]/div/div[1]/input')
            username_field.clear()
            username_field.send_keys(username)
            time.sleep(1)
            
            # 输入密码
            password_field = driver.find_element(By.XPATH, '//*[@id="app"]/div/div/div[1]/div[2]/div[2]/form/div[2]/div/div/input')
            password_field.clear()
            password_field.send_keys(password)
            time.sleep(1)
            
            # 获取并保存验证码图片
            captcha_element = driver.find_element(By.XPATH, '//*[@id="app"]/div/div/div[1]/div[2]/div[2]/form/div[3]/div/div[2]/img')
            captcha_element.screenshot(image_path)
            time.sleep(1)
            
            # 发送验证码识别请求
            captcha_text = send_captcha_request(image_path, api_url)
            
            # 输入验证码
            captcha_input = driver.find_element(By.XPATH, '//*[@id="app"]/div/div/div[1]/div[2]/div[2]/form/div[3]/div/div[1]/input')
            captcha_input.clear()
            captcha_input.send_keys(captcha_text)
            time.sleep(1)
            
            # 点击登录按钮
            login_button = driver.find_element(By.XPATH, '//*[@id="app"]/div/div/div[1]/div[2]/div[2]/form/button')
            login_button.click()
            time.sleep(5)  # 等待登录结果
            
            # 判断是否登录成功
            # 这里需要根据实际情况修改，比如检查URL变化或特定元素是否存在
            page_title = driver.title
            if  page_title == '概要 - Ming Yang':
                print("登录成功！")
                break  # 退出循环
            else:
                print("登录失败，可能是验证码识别错误，正在刷新验证码并重试...")
                # 点击验证码图片刷新
                captcha_element.click()
                time.sleep(2)  # 等待验证码刷新
    except Exception as e:
        print(f"发生错误: {e}")
    finally:
        # 保持浏览器打开，便于查看结果
        # 若不需要，可以取消下面的注释以在登录成功后关闭浏览器
        # driver.quit()
        pass

if __name__ == "__main__":
    login(url, username, password)
    time.sleep(10)

```python