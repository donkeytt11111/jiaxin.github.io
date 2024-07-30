```python
import openpyxl
import dns.resolver
import telnetlib
import logging

# 日志配置
logging.basicConfig(filename='dns_telnet.log', level=logging.INFO)

def resolve_dns(domain, nameserver='114.114.114.114'):
    resolver = dns.resolver.Resolver(configure=False)
    resolver.nameservers = [nameserver]

    try:
        a_records = resolver.resolve(domain, 'A')
        aaaa_records = resolver.resolve(domain, 'AAAA')

        print(f"正在解析 {domain} 的DNS记录...")
        print(f"A 记录: {[str(rdata) for rdata in a_records]}")
        print(f"AAAA 记录: {[str(rdata) for rdata in aaaa_records]}")

        return {'A': [str(rdata) for rdata in a_records],
                'AAAA': [str(rdata) for rdata in aaaa_records]}
    except Exception as e:
        logging.error(f"解析 {domain} 失败: {e}")
        print(f"解析 {domain} 失败: {e}")
        return {}

def check_port(ip, port=80):
    print(f"正在检查 IP {ip} 的端口 {port} 是否开放...")
    try:
        tn = telnetlib.Telnet(ip, port, timeout=5)
        tn.close()
        print(f"IP {ip} 的端口 {port} 是开放的。")
        return True
    except Exception as e:
        logging.error(f"连接到 {ip}:{port} 失败: {e}")
        print(f"连接到 {ip}:{port} 失败: {e}")
        return False

def update_excel(excel_file):
    wb = openpyxl.load_workbook(excel_file)
    sheet = wb.active

    for row in sheet.iter_rows(min_row=2, values_only=True):
        url = row[2]
        parsed_url = urlparse(url)
        domain = parsed_url.hostname
        port = parsed_url.port or 80

        records = resolve_dns(domain)
        ipv4_status = "否"
        ipv6_status = "否"

        for ip in records.get('A', []):
            if check_port(ip, port):
                ipv4_status = "是"
                break

        for ip in records.get('AAAA', []):
            if check_port(ip, port):
                ipv6_status = "是"
                break

        # 更新Excel文件的结果
        row_index = row[0]
        sheet.cell(row=row_index, column=4).value = ipv4_status
        sheet.cell(row=row_index, column=5).value = ipv6_status

    wb.save('new_' + excel_file)

# 添加对urlparse的导入
from urllib.parse import urlparse

# 调用函数
update_excel('test.xlsx')