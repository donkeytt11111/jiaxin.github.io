```python
import time
import unittest
from config import BASE_URL
from selenium.webdriver.support.ui import Select
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC
from selenium import webdriver
from selenium.webdriver.chrome.service import Service
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC

class HarborTest(unittest.TestCase):
    @classmethod
    def setUpClass(cls):
        service = Service('C:\\chromedriver.exe')  # 替换为你的chromedriver路径
        options = webdriver.ChromeOptions()
        options.add_argument('--ignore-certificate-errors')
        cls.driver = webdriver.Chrome(service=service, options=options)
        cls.driver.maximize_window()

    @classmethod
    def tearDownClass(cls):
        cls.driver.quit()

    def test_harbor_operations(self):
        driver = self.driver

        # 访问并处理证书警告
        driver.get(BASE_URL)
        # advanced_button = WebDriverWait(driver, 10).until(
        #     EC.element_to_be_clickable((By.XPATH, '//*[@id="advancedButton"]'))
        # )
        # advanced_button.click()
        # proceed_button = WebDriverWait(driver, 10).until(
        #     EC.element_to_be_clickable((By.XPATH, '//*[@id="exceptionDialogButton"]'))
        # )
        # proceed_button.click()

        """
        登录
        """
        username_field = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '//*[@id="login_username"]'))
        )
        username_field.send_keys('admin')
        password_field = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '//*[@id="login_password"]'))
        )
        password_field.send_keys('Harbor12345')
        login_button = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.ID, 'log_in'))
        )
        login_button.click()
        time.sleep(5)  # 等待登录完成

        """
        仓库管理
        """
        repository_management_button = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '/html/body/harbor-app/harbor-shell/clr-main-container/div/clr-vertical-nav/div/div[1]/clr-vertical-nav-group/div[2]/clr-vertical-nav-group-children/a[3]/span'))
        )
        repository_management_button.click()
        new_button = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '//*[@id="add"]'))
        )
        new_button.click()
        destination_name_field = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '//*[@id="destination_name"]'))
        )
        destination_name_field.send_keys('test')
        destination_url_field = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '//*[@id="destination_url"]'))
        )
        destination_url_field.send_keys('https://172.16.8.201')

        usr_exit = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '//*[@id="destination_access_key"]'))
        )
        usr_exit.send_keys('admin')

        destination_pass_field = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '//*[@id="destination_password"]'))
        )
        destination_pass_field.send_keys('Harbor12345')

        uncheck_button = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '/html/body/harbor-app/harbor-shell/clr-main-container/div/div/hbr-endpoint/div/hbr-create-edit-endpoint/clr-modal/div/div[2]/div[2]/div/div[2]/div/form/clr-checkbox-container/div/clr-checkbox-wrapper/label'))
        )
        uncheck_button.click()
        submit_button = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '/html/body/harbor-app/harbor-shell/clr-main-container/div/div/hbr-endpoint/div/hbr-create-edit-endpoint/clr-modal/div/div[2]/div[2]/div/div[3]/button[3]/span'))
        )
        submit_button.click()

        # 复制管理功能
        new_replication_rule_button = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '/html/body/harbor-app/harbor-shell/clr-main-container/div/clr-vertical-nav/div/div[1]/clr-vertical-nav-group/div[2]/clr-vertical-nav-group-children/a[4]/span'))
        )
        new_replication_rule_button.click()

        new_replication_exit_button = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '//*[@id="new_replication_rule_id"]'))
        )
        new_replication_exit_button.click()

        init_button = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '/html/body/harbor-app/harbor-shell/clr-main-container/div/div/total-replication/div/hbr-replication/div/hbr-create-edit-rule/clr-modal/div/div[2]/div[2]/div/div[2]/div/form/clr-radio-container/div/clr-radio-wrapper[2]/label'))
        )
        init_button.click()

        rule_name_field = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '//*[@id="ruleName"]'))
        )
        rule_name_field.send_keys('test')
        # target_repo_dropdown = WebDriverWait(driver, 10).until(
        #     EC.element_to_be_clickable((By.XPATH, '/html/body/harbor-app/harbor-shell/clr-main-container/div/div/total-replication/div/hbr-replication/div/hbr-create-edit-rule/clr-modal/div/div[2]/div[2]/div/div[2]/div/form/div[3]/div/div[2]'))
        # )
        # target_repo_dropdown.click()
        """
        下拉1
        """

        try:
            # 等待并定位<select>元素
            select_element = WebDriverWait(driver, 20).until(
                EC.presence_of_element_located((By.ID, 'src_registry_id'))
            )
            # 创建Select对象
            select = Select(select_element)
            print("找到了下拉列表")

        except Exception as e:
            print(f"无法找到下拉列表: {e}")

        try:
            # 选择下拉列表中的第一个非空选项
            # 注意：这里的index是从0开始的，因此第一个非空选项的索引是1（假设第一个选项是占位符）
            select.select_by_index(1)
            print("选择了下拉列表的第一个非空选项")

        except Exception as e:
            print(f"无法选择下拉列表的第一个非空选项: {e}")

        """
        处理2下拉
        """
        try:
            # 点击下拉列表以展开
            dropdown_tow = WebDriverWait(driver, 20).until(
                EC.element_to_be_clickable((By.ID, 'dest_namespace_replace_count'))
            )
            dropdown_tow.click()
            print("点击了下拉列表")
            # 等待并选择“无替换”选项
            # 选择的 XPATH 必须匹配展开后的选项
            no_replace_option = WebDriverWait(driver,10).until(
                EC.element_to_be_clickable(
                    (By.XPATH, '//select[@id="dest_namespace_replace_count"]/option[@value="0"]'))
            )
            no_replace_option.click()
            print("选择了‘无替换’选项")

        except Exception as e:
            print(f"无法处理下拉列表: {e}")

        """
        保存
        """
        time.sleep(5)
        save_button = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '//*[@id="ruleBtnOk"]'))
        )
        save_button.click()
        # 提交复制功能
        replication_checkbox = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '/html/body/harbor-app/harbor-shell/clr-main-container/div/div/total-replication/div/hbr-replication/div/div[2]/hbr-list-replication-rule/div/clr-datagrid/div[1]/div/div/div/div/clr-dg-row/div/div[1]/div/clr-radio-wrapper/label'))
        )
        replication_checkbox.click()
        replicate_button = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '//*[@id="replication_exe_id"]'))
        )
        replicate_button.click()
        confirm_replicate_button = WebDriverWait(driver, 10).until(
            EC.element_to_be_clickable((By.XPATH, '/html/body/harbor-app/harbor-shell/clr-main-container/div/div/total-replication/div/hbr-replication/div/confirmation-dialog[1]/clr-modal/div/div[2]/div[2]/div/div[3]/button[2]'))
        )
        confirm_replicate_button.click()

if __name__ == '__main__':
    unittest.main()
