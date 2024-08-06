import os,ddddocr
from time import sleep
from selenium import webdriver
from selenium.webdriver import Keys
from selenium.webdriver.common.by import By
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.action_chains import ActionChains
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC 
from PIL import Image

def login(): 
    driver_path = os.path.join(os.getcwd(), 'chromedriver.exe')
    service = Service(driver_path)
    chrome_options = webdriver.ChromeOptions()
    # 设置下载目录
    project_directory = os.path.abspath(os.path.dirname(__file__))  # 获取当前脚本所在目录的绝对路径
    download_directory = os.path.join(project_directory, 'downloads')  # 将下载目录设置为项目目录中的downloads文件夹
    if not os.path.exists(download_directory):
        os.makedirs(download_directory)
    
    # 设置更详细的User-Agent
    chrome_options.add_argument('--user-agent=Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3')
    # 下载设置
    prefs = {
        "download.default_directory": download_directory,  # 设置下载目录
        "download.prompt_for_download": False,  # 关闭下载提示
        "download.directory_upgrade": True,
        "safebrowsing.enabled": True  # 禁用安全浏览器检查
    }
    # 隐藏Selenium特征
    chrome_options.add_experimental_option('excludeSwitches', ['enable-automation'])
    chrome_options.add_experimental_option('useAutomationExtension', False)
    chrome_options.add_experimental_option('prefs', prefs)
    
    driver = webdriver.Chrome(service=service, options=chrome_options)
    driver.execute_cdp_cmd("Page.addScriptToEvaluateOnNewDocument", {
        "source": """
            Object.defineProperty(navigator, 'webdriver', {
                get: () => undefined
            })
        """
    })
    #这里的login_url来判断是否完成了登录的,验证码输错就再来一遍
    login_url = 'https://www.cnvd.org.cn/user/login'
    #真正是向real_url发起请求，然后会跳转到login_url，登陆后跳回real_url
    real_url='https://www.cnvd.org.cn/shareData/list'
    #验证码错误后跳转的url
    error_url='https://www.cnvd.org.cn/user/doLogin/loginForm'
    # 模拟用户行为
    action = ActionChains(driver)
    action.move_by_offset(100, 100).perform()  # 随意移动鼠标
    #浏览器登录real_url
    
    #用循环防止验证码输入错误
    while True:
        driver.get(real_url)
        sleep(5)
        #获取屏幕截图并得到验证码的位置
        driver.save_screenshot('full_screen.png')
        img = driver.find_element(By.ID, 'codeSpan')
        left = img.location['x']
        top = img.location['y']
        right = img.location['x'] + img.size['width']
        bottom = img.location['y'] + img.size['height']
        photo = Image.open('full_screen.png')
        photo = photo.crop((left, top, right, bottom))
        photo.save('Verification_code.png')
        # 输入登录信息
        code = VerificationCode()
        email_field = driver.find_element(By.NAME, 'email')
        password_field = driver.find_element(By.ID, 'password')
        Code_field = driver.find_element(By.NAME, 'myCode')
        email_field.send_keys('your_email_address')#your email address 你的账号
        password_field.send_keys('your_password')#your password 你的密码
        Code_field.send_keys(code, Keys.ENTER)
        WebDriverWait(driver, 10).until(EC.url_changes(login_url))
        current_url = driver.current_url
        if current_url == real_url:
            break  # 成功登录，跳出循环

    driver.implicitly_wait(3)
    #一页就10个 每天都获取的话就不需要翻页
    for i in range(1,11):
        #该cnvd点击下载的xpath
        download_xpath="//*[@id='patchList']/table/tbody/tr["+str(i)+"]/td[1]/a"
        #寻找时间，然后与上一次上传时间进行对比，如果时间在上次上传时间之后，就上传当前的cnvd
        download_element=driver.find_element(By.XPATH,(download_xpath))
        download_element.click()
        sleep(3)

def VerificationCode():
    '''
    得到验证码
    '''
    ocr = ddddocr.DdddOcr(show_ad=False)
    image = open("Verification_code.png", "rb").read()
    result = ocr.classification(image)
    return result
