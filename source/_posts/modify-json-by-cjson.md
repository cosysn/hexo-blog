---
title: 使用cJSON修改json的值
date: 2018-01-07 02:21:45
tags:
---

## 0. 背景
将json中包含的关键信息替换成字符'*'，这样可以保护json中的数据安全。

## 1. 安装cJSON
下载[cJSON](https://github.com/DaveGamble/cJSON)的源码。
然后执行下面的命令编译cJSON：
```
mkdir build
cd build
cmake ..
make
make insatll
```
这样就将cJSON的动态库安装成功了，那么使用下面的demo来测试是否可以使用cJSON了。
```
#include <stdio.h>
#include <stdlib.h>
#include <cjson/cJSON.h>

int main()
{
    char *url = "{\"ak\":\"sjdhjfdsfjd\"}";
    cJSON *url_json = cJSON_Parse(url);
    if (NULL == url_json)
    {
        const char *error_ptr = cJSON_GetErrorPtr();
        if (error_ptr != NULL)
        {
            fprintf(stderr, "Error before: %s\n", error_ptr);
        }
    }
    cJSON_Delete(url_json);

    return 0;
}
```

将上述的demo编译成可执行程序，需要执行下面的命令：
```
gcc cjson_demo.c -lcjson -o cjson_demo
export LD_LIBRARY_PATH=${LD_LIBRARY_PATH}:/usr/local/lib64/
```

## 2. 修改json的值
```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <cjson/cJSON.h>

#define ERR_INVALID_URL (-10)
#define ERR              (-1)

int filter_single_sensitive_word(const cJSON *item, char *name)
{
	cJSON *subitem = NULL;
	char *tmp = NULL;

	if (NULL == item)
	{
		fprintf(stderr, "Parameter is invalid\n");
		return ERR;
	}

	subitem = cJSON_GetObjectItem(item, name);
	if (cJSON_IsString(subitem) && (subitem->valuestring != NULL))
	{
		for (tmp = subitem->valuestring; *tmp; tmp++)
		{
			*tmp = '*';
		}
	}

	return 0;
}

int filter_sensitive_word(char *url, int size)
{
	int ret = 0;
	char *filter_str = NULL;

	/*json字符串非法，解析url失败*/
	cJSON *url_json = cJSON_Parse(url);
	if (NULL == url_json)
	{
		const char *error_ptr = cJSON_GetErrorPtr();
        if (error_ptr != NULL)
        {
            fprintf(stderr, "Error before: %s\n", error_ptr);
        }
        ret = ERR_INVALID_URL;
        return ret;
	}

	/**/
	ret = filter_single_sensitive_word(url_json, "abc");
	if (0 != ret)
	{
		fprintf(stderr, "filter single sensitive word fail\n");
		return ret;
	}

	filter_str = cJSON_PrintUnformatted(url_json);
	strncpy(url, filter_str, size);
	url[size-1] = '\0';

	cJSON_Delete(url_json);
	return 0;
}

int main()
{
	int ret = 0;
    char url[] = "{\"abc\":\"ssdj\",\"ad\":  \"sjdhjfdsfjd\",\"tk\":\"abc\",\"sd\":\"hsdfjdshd\"}";
    ret = filter_sensitive_word(url, sizeof(url));
    if ((ret == 0) || (ret == ERR_INVALID_URL))
    {
    	printf("%s\n", url);
    }
    else
    {
    	printf("filter sensitive word fail, errno[%d]", ret);
    }
    return 0;
}
```
## 3. 参考文献
1. [Ultralightweight JSON parser in ANSI C](https://github.com/DaveGamble/cJSON)