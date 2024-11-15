# anime-press-release-websites-scraper
python file to scrap some anime press release websites scraper

### https://akiba-souken.com
```python
# 要删除的关键词
!huggingface-cli login --token hf_fXye……
!wget -P /content https://github.com/Map987/image/releases/download/ojjj/links_sorted.13.txt
keyword = '599999' #@param {type:"string"}

# 要处理的文件路径
file_path = '/content/links_sorted.13.txt'

# 读取文件，并保存包含关键词的行之后的行
with open(file_path, 'r') as file:
    lines = file.readlines()

# 找到包含关键词的行索引
keyword_index = next((i for i, line in enumerate(lines) if keyword in line), None)

# 如果找到关键词，则删除该行以及之前的所有行
if keyword_index is not None:
    # 从包含关键词的行之后开始的所有行
    remaining_lines = lines[keyword_index+1:]

    # 将剩余的行写回文件
    with open(file_path, 'w') as file:
        file.writelines(remaining_lines)
```

```python
import re
from huggingface_hub import HfApi
import requests

# Initialize the Hugging Face API
api = HfApi()

# Specify the repository
repo_id = "haibaraconan/akiba"
repo_type = "dataset"

# List files in the repository
files_info = api.list_repo_files(repo_id=repo_id, repo_type=repo_type)
file_list = files_info

# Extracting the list of files starting with 'html/60/' from the provided list of files
html_60_files = [file for file in file_list if file.startswith('html/60/')]

# Extracting the filenames and converting them to integers
numbers = []
for file in html_60_files:
    match = re.search(r'(\d+)(?=\.[^.]+$)', file)
    if match:
        number = int(match.group())
        numbers.append(number)

# Sorting the numbers
numbers.sort()

# Extracting the maximum number
max_number = numbers[-1]
min_number = numbers[0]

# Finding the missing natural numbers
missing_numbers = [num for num in range(min_number + 1, max_number) if num not in numbers]

# Prepare to save missing files and log errors
log_entries = []

# Function to save files and log errors
def save_file(number):
    url = f'https://akiba-souken.com/article/{number}/'
    headers = {
        'Host': 'akiba-souken.com',
        'User-Agent': 'Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/58.0.3029.110 Safari/537.3',
        'Accept': '*/*',
        'Referer': 'https://websniffer.com/',
        'Accept-Encoding': 'gzip',
        'Connection': 'keep-alive'
    }

    try:
        print(number)
        response = requests.get(url, headers=headers)  # Use GET instead of POST
        if response.status_code == 200:
            with open(f'/content/{number}.html', 'w', encoding='utf-8') as file:
                file.write(response.text)
        else:
            log_entries.append(f"Failed to retrieve {number}. Status code: {response.status_code}")
    except Exception as e:
        log_entries.append(f"Error retrieving {number}. Exception: {str(e)}")

# Process each missing number
for num in missing_numbers:
    save_file(num)

# Save log entries
if log_entries:
    with open('/content/log.txt', 'w', encoding='utf-8') as log_file:
        for entry in log_entries:
            log_file.write(entry + '\n')

max_number, min_number, missing_numbers, len(log_entries)  # Return max, min, missing numbers and log entry count
```

```python
import requests
import re

# 文件下载函数
def download_file(url):
    response = requests.get(url)
    response.raise_for_status()  # 确保请求成功
    return response.text

# 解析文件并提取图片文件名中的数字
def extract_numbers_from_text(text):
    # 正则表达式已更新，以匹配文件名中的首个数字序列，直到遇到第一个点号（.）
    return set(map(int, re.findall(r'(\d+)[.][a-zA-Z]+', text)))
def format_url_final_correct(url):
    url = str(url)
    # 确保输入的是数字字符串
    if not url.isdigit():
        raise ValueError("URL must be a numeric string")

    # 计算百万位、千位到10万位和剩余数字
    num_length = len(url)
    millions = url[:-6].zfill(3) if num_length > 6 else '000'

    # 对于长度大于等于6的数字，取千位到10万位的数字；否则，取整个数字
    if num_length >= 6:
        tens_of_thousands = url[-6:-3]
    else:
        tens_of_thousands = url
    rest = url[-4:]

    # 构造格式化的URL
    formatted_url = f"https://image-org-s.akiba-souken.com/assets/images/article/{millions}/{tens_of_thousands.zfill(3)}/{url}.jpg"

    return formatted_url

# 测试函数


# 主函数
def main():
    # 下载并解析指定的文本文件
    akiba_files = [
        f"https://huggingface.co/datasets/haibaraconan/akiba/raw/main/{i}-{i+49999}.txt" for i in range(700000, 1050000, 50000)
    ]
    akiba_files += [
        "https://huggingface.co/datasets/haibaraconan/akiba/raw/main/65000-699999.tarfile_list.txt",
        "https://huggingface.co/datasets/haibaraconan/akiba/raw/main/1000000-1499999.txt",
        "https://huggingface.co/datasets/haibaraconan/akiba/raw/main/600000-649999.txt",
        "https://huggingface.co/datasets/haibaraconan/akiba/raw/main/500000-999999.txt",
        "https://huggingface.co/datasets/haibaraconan/akiba/raw/main/1000000-1499999%E8%A1%A5.txt",
        "https://huggingface.co/datasets/haibaraconan/akiba/raw/main/500000-999999%E8%A1%A5.txt",
        "https://huggingface.co/datasets/haibaraconan/akiba/raw/main/500000-999999%E8%A1%A5%E8%A1%A5.txt",
        "https://huggingface.co/datasets/haibaraconan/akiba/raw/main/1000000-1499999%E8%A1%A5%E8%A1%A5.txt"
    ]
    all_numbers = set()
    min_num = None
    max_num = None

    for file_url in akiba_files:
        try:
            text = download_file(file_url)
            numbers = extract_numbers_from_text(text)
            all_numbers.update(numbers)

            # Check if this file contains the new minimum number
            if min_num is None or (numbers and min(numbers) < min_num):
                min_num = min(numbers)

            # Check if this file contains the new maximum number
            if max_num is None or (numbers and max(numbers) > max_num):
                max_num = max(numbers)
        except requests.HTTPError as e:
            print(f"Failed to download {file_url}: {e}")

    # 查找在akiba_files集合中不存在的自然数
    missing_numbers = sorted(set(range(min_num, max_num + 1)) - all_numbers)

    # 输出所有自然数到txt中
    with open('/content/mi.txt', 'w') as f:
        print(max_num, min_num)
        for number in missing_numbers:
            print(number)
            number = format_url_final_correct(number)
            f.write(f"{number}\n")

# 执行主函数
if __name__ == "__main__":
    main()
```
html文件夹中找图片链接

```python
##### ed78.ipynb


import os
import re
from concurrent.futures import ThreadPoolExecutor

# 设置文件夹路径
folder_path = '/content/0-9999'

# 设置输出文件的路径
output_file_path = '/content/links_sorted.txt'

# 正则表达式来匹配 <!--関連画像start--> 到 <!--関連画像end--> 的内容
related_images_pattern = re.compile(r'<!--関連画像start-->(.*?)<!--関連画像end-->', re.DOTALL)

# 正则表达式来匹配 img src，允许数字部分和后缀变化
img_src_pattern = re.compile(r'src="https://akiba-souken.k-img.com/images/article/(\d{3,})/(\d+)/t120_(\d{6,7})\.(\w+)"')

# 定义一个函数来处理单个文件
def process_file(file_name):
    file_path = os.path.join(folder_path, file_name)
    with open(file_path, 'r', encoding='utf-8') as file:
        content = file.read()
        related_images_matches = related_images_pattern.findall(content)
        links = []
        for related_images in related_images_matches:
            img_src_matches = img_src_pattern.findall(related_images)
            for match in img_src_matches:
                link = f'https://akiba-souken.k-img.com/images/article/{match[0]}/{match[1]}/t120_{match[2]}.{match[3]}'
                links.append(link)
        return links

# 使用线程池来处理所有文件
links = []
with ThreadPoolExecutor() as executor:
    # 提交任务到线程池
    futures = [executor.submit(process_file, file_name) for file_name in os.listdir(folder_path) if file_name.endswith('.html')]

    # 收集结果
    for future in futures:
        links.extend(future.result())

# 按照链接中的数字进行排序
links.sort(key=lambda x: int(re.search(r't120_(\d{6,7})\.', x).group(1)))

# 读取现有文件内容
try:
    with open(output_file_path, 'r', encoding='utf-8') as file:
        existing_links = file.readlines()
except FileNotFoundError:
    existing_links = []

# 移除每行末尾的换行符，并合并两个列表
existing_links = [link.strip() for link in existing_links]  # 移除换行符
all_links = list(set(existing_links + links))  # 合并并去重

# 按照链接中的数字进行排序
all_links.sort(key=lambda x: int(re.search(r't120_(\d{6,7})\.', x).group(1)))

# 将合并后的链接列表写入到文件中
with open(output_file_path, 'w', encoding='utf-8') as output_file:
    output_file.writelines(f"{link}\n" for link in all_links)

print(f'链接已提取并更新到 {output_file_path}')    

```

```python
import os
import threading
import requests
from queue import Queue

# 下载进度记录文件
PROGRESS_FILE = 'download_progress.txt'

# 每个文件夹处理的文件数量
FILES_PER_FOLDER = 500000

# 线程数
NUM_THREADS = 128 # 根据你的需求和环境调整线程数

# 队列用于存放下载任务
queue = Queue()

# 线程锁，用于同步线程
thread_lock = threading.Lock()

# 已下载的文件集合
downloaded = set()

# 文件夹下载状态跟踪
folder_download_status = {}
def format_url_final_correct(url):
    url = str(url)
    # 确保输入的是数字字符串
    if not url.isdigit():
        raise ValueError("URL must be a numeric string")

    # 计算百万位、千位到10万位和剩余数字
    num_length = len(url)
    millions = url[:-6].zfill(3) if num_length > 6 else '000'
    
    # 对于长度大于等于6的数字，取千位到10万位的数字；否则，取整个数字
    if num_length >= 6:
        tens_of_thousands = url[-6:-3]
    else:
        tens_of_thousands = url
    rest = url[-4:]
    
    # 构造格式化的URL
    formatted_url = f"https://image-org-s.akiba-souken.com/assets/images/article/{millions}/{tens_of_thousands.zfill(3)}/{url}.JPG" #jpg #png #JPF
    
    return formatted_url    
    
# 读取进度
def read_progress():
    try:
        with open(PROGRESS_FILE, 'r') as f:
            return set(f.read().splitlines())
    except FileNotFoundError:
        return set()

# 保存进度
def save_progress(downloaded):
    with open(PROGRESS_FILE, 'w') as f:
        for item in downloaded:
            f.write(item + '\n')

# 下载文件
def download_file():
    while True:
        with thread_lock:
            if queue.empty():
                break

            url, folder, file_index = queue.get()

        if url in downloaded:
            continue

        try:
            url = format_url_final_correct(url)
            filename = url.split('/')[-1].replace('t120_', '')
            url = url.replace('t120_', '')
            response = requests.get(url)
            response.raise_for_status()
            os.makedirs(folder, exist_ok=True)
            filepath = os.path.join(folder, filename)
            with open(filepath, 'wb') as f:
                f.write(response.content)
            downloaded.add(url)

            # 更新文件夹下载状态
            with thread_lock:
                if folder not in folder_download_status:
                    folder_download_status[folder] = 1
                else:
                    folder_download_status[folder] += 1

                # 检查文件夹是否下载完成
                if folder_download_status[folder] == FILES_PER_FOLDER:
                    print(f"Download completed for folder: {folder}")
                    folder_download_status[folder] = 0  # 重置计数器

        except requests.RequestException as e:
            pass
        finally:
            queue.task_done()

# 主函数
def main():
    global downloaded
    downloaded = read_progress()

    # 填充队列
    with open('/content/mi.txt', 'r') as f:
        for line in f:
            line = line.strip()
            if not line:
                continue
            filename = line.split('/')[-1].replace('t120_', '')
            number = int(filename.split('.')[0])
            folder = f'{number//FILES_PER_FOLDER*FILES_PER_FOLDER}-{(number//FILES_PER_FOLDER+1)*FILES_PER_FOLDER-1}'
            queue.put((line, folder, number))

    # 启动线程池
    threads = []
    for _ in range(NUM_THREADS):
        thread = threading.Thread(target=download_file)
        thread.start()
        threads.append(thread)

    # 等待所有线程完成
    for thread in threads:
        thread.join()

    # 保存下载进度
    save_progress(downloaded)

if __name__ == '__main__':
    main()
```
