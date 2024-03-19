# FoodCrawler

### 美食杰

安装依赖包
DrissionPage为开发开源爬虫框架
![](/resource/meishijie/Pasted image 20240319154828.png)
https://gitee.com/g1879/DrissionPage
手册：
https://g1879.gitee.io/drissionpagedocs/get_start/import
``` shell
pip install DrissionPage
```

以美食杰网站为例
### 倒入包
csv用来保存菜谱
``` python
import csv
from DrissionPage import SessionPage
```
### 爬取菜系列表

获取菜系分类，默认获取全部
![](/resource/meishijie/Pasted image 20240319153553.png)
```python
def get_caixi_link(caixi_group=None, caixi_name=None):
    result = []
    page.get('https://www.meishij.net/caipufenlei/') # 获取菜系列表
    sort_wrap_list = page.eles('.sort_wrap')
    for h2 in sort_wrap_list:
        if caixi_group and caixi_group!= h2.ele('tag:h2').text:
            continue # 不等于指定的分组，跳过
        else:
            caixi_list = h2.eles('tag:li')
            for caixi in caixi_list: # 循环获取菜系列表
                caixi_a = caixi.ele('tag:a')
                if caixi_name and caixi_name!= caixi_a.text:
                    continue    # 不等于指定的菜系，跳过
                elif caixi_name == caixi_a.text:
                    result.append(dict(caixi_name=caixi_a.text, caixi_url=caixi_a.attr('href')))
    return result
```

### 爬取菜单列表
获取菜单列表，并获取分页信息
如图：
![](/resource/meishijie/Pasted image 20240319154722.png)
```python
def get_menu_items(url):
    result = []
    page.get(url)
    menu_list = page.eles('.list_s2_item')
    for menu in menu_list:
        menu_name = menu.ele('.title').text
        menu_url = menu.ele('tag:a').attr('href')
        result.append(dict(menu_name=menu_name  , menu_url=menu_url))
    # 下一页
    try:
        next_page = page.ele('#recipe_page').ele('text:下一页').attr('href')
    except Exception as e:
        print(e)
        next_page = None
    return result, next_page
```

### 获取食谱详情
爬取的数据范围主要包含：
- 菜谱名、收藏数、~~图片~~、~~做法~~
- 工艺、口味、时间、难度
- 主料、辅料
如图：
![](/resource/meishijie/Pasted image 20240319154558.png)
```python
def get_recipe_details(url):
    result = dict(main_list=[], others_list=[])
    page.get(url, retry=3, interval=1)
    result['recipe_title'] = page.ele('.recipe_title').text # 获取标题
    # print(result['recipe_title'])
    try:
        shoucang = page.ele('.info1').text # 获取收藏数
        result['shoucang'] = shoucang.split(' ')[1]
    except Exception as e:
        print(e)
    result['gongyi'] = page.ele('tag:em@text():工艺').next().text # 获取工艺后一个同级节点的文本
    result['kouwei'] = page.ele('tag:em@text():口味').next().text # 同上
    result["shijian"] = page.ele('tag:em@text():时间').next().text # 同上
    result["nandu"] = page.ele('tag:em@text():难度').next().text # 同上
    main_list = page.ele('.recipe_ingredientsw').ele('.right').eles('tag:strong') # 获取主料列表
    for main in main_list:
        tag_a = main.ele('tag:a')
        result['main_list'].append(dict(name=tag_a.text, size=main.text.split(tag_a.text)[1]))
    others_list = page.ele('.recipe_ingredients recipe_ingredients1').ele('.right').eles('tag:strong') # 获取辅料列表
    for other in others_list:
        tag_a = other.ele('tag:a')
        result['others_list'].append(dict(name=tag_a.text, size=other.text.split(tag_a.text)[1]))
    return result
```

### 保存到CSV文件
参考手册：
```python
def save_to_csv(data):
	with open('recipe.csv', 'a', newline='') as f:
		writer = csv.DictWriter(f, data.keys())
		writer.writerow(data)
```

### 入口
```python
if __name__ == '__main__':
    caixi_list = get_caixi_link('中华菜系', '川菜')
    menu_list = []
    for item in caixi_list: # 循环获取菜系列表
        caixi_name = item['caixi_name']
        caixi_url = item['caixi_url']
        menu_list, next_page = get_menu_items(caixi_url)
        while next_page:    # 循环获取菜系列表下的菜谱列表
            for menu in menu_list:  # 循环获取菜谱列表
                try:
                    details = get_recipe_details(menu['menu_url'])
                    save_to_csv(details) # 保存到csv
                    print(f'save {menu["menu_name"]}')
                except Exception as e:
                    print(f'error {e}, {menu["menu_name"]}')
            menu_list, next_page = get_menu_items(next_page)
```

### 最终导出格式
![](/resource/meishijie/Pasted image 20240319154257.png)


