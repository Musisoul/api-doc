## 图片生成(i2i,t2i) 接口

### 获取token

请求地址

> POST   [https://miaohua.sensetime.com/api/v1b/get_token](https://miaohua.sensetime.com/api/v1b/get_token)


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| email | string | 是 | "[xxx@example.com](mailto:xxx@example.com)
" | tob用户的用户邮箱，需先在miaohua.sensetime.com注册 |
| password | string | 是 | "password" | tob用户密码 |


请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/get_token'
data = {
   "email": "xxx@example.com",
   "password": "password",
}

response = requests.post(url, json=data)

print(response.status_code)
print(response.text)
```

返回示例

```json
{
    "token": "xxxxxxxxxxxxxxxxxx", # string 当前用户token
}
```

### 任务提交

请求地址

> POST   [https://miaohua.sensetime.com/api/v1b/task_submit](https://miaohua.sensetime.com/api/v1b/task_submit)


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| token | string | 是 | 无，通过get_token获得 | 通过get_token获取的token |
| model_name | string | 否 | Artist v0.3.0 Beta | 🔴❗Warning❗🔴 模型名称(可选值: 可通过/api/v1b/models/base获得) 参数存在隐患，现阶段支持只提交model_name，versionid为空，将在202405删除对model_name字段支持|
| versionid | string | 是 | sgl_artist_v0.3.5_0925 | 新增字段，模型版本id( 可通过/api/v1b/models/base获得对应versionid) versionid必须提交|
| prompt | string | 是 | "" | 用于生成图片的特征描述，如："one girl,beautiful" |
| neg_prompt | string | 否 | "" | 特征的反向描述，一般无需指定 |
| n_images | int | 是 | 2 | 生成图片数量 |
| scale | int | 是 | 7 | 文本控制力度(1-20), 同Prompt weight |
| strength | float | 是 | 0.6 | 图片控制力度(0-1), 同Image weight |
| ddim_steps | int | 否 | 50 | 迭代步数 |
| select_seed | int | 否 | -1 | 随机数种子 |
| output_size | string | 文生图是,图生图否 | 960x960 | 图片的输出尺寸，如："960x960",长度范围可选640-6144，宽度范围可选640-6144|
| init_img | string | 否 | "" | 输入图片，url形式，若有即为i2i(图生图)，无即为t2i(文生图)。必须是通过/api/v1b/upload_img或 /api/v1b/upload_imgs 上传后的图片地址 |
| add_prompt | bool | 否 | false | 是否使用gpt进行描述词优化, 开启优化生图速度会有一定影响 |
| cn_configs | array | 否 | null | controlnet配置，包括权重、图片，见示例. 支持的cn从 /api/v1b/cn/artist 或 /api/v1b/cn/opensource中获取. 权重范围为 0~2, 图片为通过upload_img/upload_imgs接口上传的图片地址，最多支持传入3个cn |
| lora_configs | array | 否 | null | lora配置， 包括lora模型、权重， 见示例. versionid从 /api/v1b/models/private 或 /api/v1b/models/lora中获取. 权重范围为 0~2 |


> **_注意_**：
>  
> 1. 开源和自研模型的controlnet支持不一样，可以通过 /api/v1b/cn/artist 和 /api/v1b/cn/opensource获取自研和开源模型支持的cn
> 2. 只有基模型才能作为初始模型，可以通过 /api/v1b/models/base获取基模型， 目前支持一层lora，多层不生效


请求示例

**curl 示例**

```
curl https://miaohua.sensetime.com/api/v1b/task_submit \
  -H "Content-Type: application/json" \
  -d '{
    "versionid": "sgl_artist_v0.3.5_0925",
    "prompt": "one girl, beautiful",
    "neg_prompt": "",
    "n_images": 2,
    "scale": 7,
    "output_size": "960x960",
    "select_seed": -1,
    "token": "",
    "init_img": "",
    "ddim_steps": 7,
    "strength": 0.82,
    "lora_configs": [
      {"versionid": "", "merge_weight": 1}
    ],
    "cn_configs": {
      {"cn": "openpose", "img": "https://ossxxxxxx.png", "weight": 1}
    }
  }'
```

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/task_submit'
data = {
    "versionid": "sgl_artist_v0.3.5_0925", # string 用到的模型versionid（规定范围内）
    "prompt": "one girl, beautiful", # 正向描述词
    "neg_prompt": "", # 反向描述词
    "n_images": 2, # int 生成图片的数量
    "ddim_steps": 50, # int 迭代步数
    "scale": 7, # int 文本控制力度
    "strength": 0.6, # float 图片控制力度
    "output_size": "960x960", # string 生成的图片尺寸
    "select_seed": -1, # int 随机数种子。如果是-1的话代表不指定
    "init_img": "", # img url,若为空为文生图，否则为图生图
    "add_prompt": False, # 是否使用gpt进行描述词优化
    "token": "",  # get_token获取的token
    "lora_configs": [
      {"versionid": "aaa", "merge_weight": 1}  # versionid从 /api/v1b/models/private 或 /api/v1b/models/lora中获取. 权重范围为 0~2
    ],
    "cn_configs": [
      {"cn": "openpose", "img": "https://ossxxxxxx.png", "weight": 1}  # 支持的cn从 /api/v1b/cn/artist 或 /api/v1b/cn/opensource中获取. 权重范围为 0~2, 图片为通过upload_img/upload_imgs接口上传的图片地址
    ]
}

response = requests.post(url, json=data)

print(response.status_code)
print(response.text)
```
#### 

**controlnet 参数示例**
```json
{
        "cn": "tile", //['canny','lineart',..]
        "weight": 0.3,
        "guidance_start": 0.1, //  0-1 默认0
        "guidance_end": 0.9, //  0-1 默认1
        "img": "https://bkmk.oss-accelerate.aliyuncs.com/35c82400-52c8-11ee-817a-1e719ed4d673.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317054673034&Signature=X1A%2Bhnro0l0TmLOWKknlBckJhok%3D",
        "config": {//config可选，针对特定cn支持配置
            "control_processor": "tile_resample",
            "down_sample_rate": 1 // , tile需要 浮点数两位小数,默认1,范围1-8
        }
}
```
**以下为cn现在支持的内置config 参数, 存在差异，按照文档调用**
```json
{
        "cn": "canny",
        "weight": 0.3,
        "guidance_start": 0.1, //  0-1 默认0
        "guidance_end": 0.9, //  0-1 默认1
        "img": "https://bkmk.oss-accelerate.aliyuncs.com/35c82400-52c8-11ee-817a-1e719ed4d673.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317054673034&Signature=X1A%2Bhnro0l0TmLOWKknlBckJhok%3D",
        "config": {
            "control_processor": "canny",//canny invert
            "low_threshold":1,// 1-255,
            "high_threshold": 1,// 1-255,
        }
}
{
        "cn": "lineart",
        "weight": 0.3,
        "guidance_start": 0.1, //  0-1 默认0
        "guidance_end": 0.9, //  0-1 默认1
        "img": "https://bkmk.oss-accelerate.aliyuncs.com/35c82400-52c8-11ee-817a-1e719ed4d673.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317054673034&Signature=X1A%2Bhnro0l0TmLOWKknlBckJhok%3D",
        "config": {
            "control_processor": "lineart_realistic",//'lineart_realistic','lineart_coarse','lineart_standard','invert'
        }
}
{
        "cn": "lineart_anime",
        "weight": 0.3,
        "guidance_start": 0.1, //  0-1 默认0
        "guidance_end": 0.9, //  0-1 默认1
        "img": "https://bkmk.oss-accelerate.aliyuncs.com/35c82400-52c8-11ee-817a-1e719ed4d673.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317054673034&Signature=X1A%2Bhnro0l0TmLOWKknlBckJhok%3D",
        "config": {
            "control_processor": "lineart_anime" or "lineart_anime_denoise",
        }
}
{
        "cn": "openpose",
        "weight": 0.3,
        "guidance_start": 0.1, //  0-1 默认0
        "guidance_end": 0.9, //  0-1 默认1
        "img": "https://bkmk.oss-accelerate.aliyuncs.com/35c82400-52c8-11ee-817a-1e719ed4d673.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317054673034&Signature=X1A%2Bhnro0l0TmLOWKknlBckJhok%3D",
        "config": {
            "control_processor": "dw_openpose_full,
            /*['dw_openpose_full',
                'openpose',
                'openpose_face',
                'openpose_faceonly',
                'openpose_full',
                'openpose_hand',]*/
        }
}
{
        "cn": "tile",
        "weight": 0.3,
        "guidance_start": 0.1, //  0-1 默认0
        "guidance_end": 0.9, //  0-1 默认1
        "img": "https://bkmk.oss-accelerate.aliyuncs.com/35c82400-52c8-11ee-817a-1e719ed4d673.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317054673034&Signature=X1A%2Bhnro0l0TmLOWKknlBckJhok%3D",
        "config": {
            "control_processor": "tile_colorfix,
            /*['tile_colorfix',
              'tile_colorfix_sharp',
              'tile_resample',]*/
            "down_sample_rate":1,//1-8

        }
}
```


**多cn 示例**
```json
{
    "token": "",
    "lora_configs": [],
    "model_name": "Anything V3_fp16_train",
    "prompt": "t1#girl, room, red hair, red clothes",
    "neg_prompt": "",
    "n_images": 2,
    "scale": 7,
    "select_seed": -1,
    "init_img": "https://bkmk.oss-accelerate.aliyuncs.com/2c0001d0-4004-11ee-b30c-00163e005161.jpg?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317052609716&Signature=ZalLV8FSCsTU3DjPtKCqOn2qXr0%3D",
    "add_prompt": false,
    "ddim_steps": 50,
    "output_size": "2748x4096",
    "cn_configs": [{
        "cn": "tile",
        "weight": 0.3,
        "guidance_start": 0.1, //  0-1 默认0
        "guidance_end": 0.9, //  0-1 默认1
        "img": "https://bkmk.oss-accelerate.aliyuncs.com/35c82400-52c8-11ee-817a-1e719ed4d673.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317054673034&Signature=X1A%2Bhnro0l0TmLOWKknlBckJhok%3D",
        "config": {
            "control_processor": "tile_resample",
            "down_sample_rate": 1 // , tile需要 浮点数两位小数,默认1,范围1-8
        }
    }, {
        "cn": "brightness",
        "weight": 0.4,
        "guidance_start": 0.2, //  0-1 默认0
        "guidance_end": 0.8, //  0-1 默认1
        "img": "https://bkmk.oss-accelerate.aliyuncs.com/35c82400-52c8-11ee-817a-1e719ed4d673.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317054673034&Signature=X1A%2Bhnro0l0TmLOWKknlBckJhok%3D",
      
    }]
}
{
    "token": "",
    "lora_configs": [],
    "model_name": "Dalcefo",
    "prompt": "t1#girl, room, red hair, red clothes",
    "neg_prompt": "",
    "n_images": 2,
    "scale": 7,
    "select_seed": -1,
    "init_img": "https://bkmk.oss-accelerate.aliyuncs.com/8804c8b2-52c9-11ee-aa0d-ea71f9947415.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317054673602&Signature=hguT%2BR3VxxSvZLxBg%2FMi6vanyA8%3D",
    "add_prompt": false,
    "ddim_steps": 50,
    // "output_size": "2748x4096",
    "output_size": "2256x2252",
    "cn_configs": [{
        "cn": "canny",
        "weight": 0.9,
        "guidance_start": 0, //  0-1 默认0
        "guidance_end": 1, //  0-1 默认1
        "img": "https://bkmk.oss-accelerate.aliyuncs.com/aaa2d68e-52c4-11ee-817a-1e719ed4d673.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317054671512&Signature=hUkZUXxJ1eHCwgQ4m0UiWz%2BaC6o%3D",
        "config": {
            "control_processor": "canny",
            "low_threshold": 100, // , canny需要 范围1-255，默认100
            "high_threshold": 200 // , canny需要 范围1-255，默认200
        }
    	}]
}
```

返回示例

```json
{
    "task_id": "f3e5b59c-7416-11ed-a160-00163e025c94", # string 任务id
}
```

### 结果查询

请求地址

> POST       [https://miaohua.sensetime.com/api/v1b/task_result](https://miaohua.sensetime.com/api/v1b/task_result)


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| task_id | string | 是 | 无 | 由任务创建接口返回的任务id |
| token | string | 是 | 无 | 由get_token获取的token |
| sensitive_limit_value | float | 否 | 0 | 0-1，设置敏感图片鉴定阈值，越大代表越严格，0采用系统默认标准|


请求示例

**curl 示例**

```
curl https://miaohua.sensetime.com/api/v1b/task_result \
  -H "Content-Type: application/json" \
  -d '{
        "task_id": "f3e5b59c-7416-11ed-a160-00163e025c94",
        "token": "",
  }'
```

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/task_result'
data = {
    "task_id": "f3e5b59c-7416-11ed-a160-00163e025c94", # string 任务id
    "token": "", # get_token获取的token
}

response = requests.post(url, json=data)

print(response.status_code)
print(response.text)
```

返回示例

```json
{
    "state": "done", # string 任务状态，未完成会是 "pending"
    'images': [
        {
            'large': "https://pic.bakamaka.io/txt2img-result/2022-12-10/31e17f66-788b-11ed-98e2-00163e025c94_00000.jpg",
            'middle': "https://pic.bakamaka.io/txt2img-result/2022-12-10/31e17f66-788b-11ed-98e2-00163e025c94_00000.mid.jpg",
            'save_index': 0,
            'share': false,
            'shareable': true,
        },
        {
            'large': "https://pic.bakamaka.io/txt2img-result/2022-12-10/31e17f66-788b-11ed-98e2-00163e025c94_00001.jpg",
            'middle': "https://pic.bakamaka.io/txt2img-result/2022-12-10/31e17f66-788b-11ed-98e2-00163e025c94_00001.mid.jpg",
            'save_index': 1,
            'share': false,
            'shareable': true,
        },
        {
            'large': "https://pic.bakamaka.io/txt2img-result/2022-12-10/31e17f66-788b-11ed-98e2-00163e025c94_00002.jpg",
            'middle': "https://pic.bakamaka.io/txt2img-result/2022-12-10/31e17f66-788b-11ed-98e2-00163e025c94_00002.mid.jpg",
            'save_index': 2,
            'share': false,
            'shareable': true,
        },
        {
            'large': "https://pic.bakamaka.io/txt2img-result/2022-12-10/31e17f66-788b-11ed-98e2-00163e025c94_00003.jpg",
            'middle': "https://pic.bakamaka.io/txt2img-result/2022-12-10/31e17f66-788b-11ed-98e2-00163e025c94_00003.mid.jpg",
            'save_index': 3,
            'share': false,
            'shareable': true,
        },
    ], # list 图片路径
    "select_seed": 8206464,#种子数
      
    "error_msg": "Intern error", # string 错误信息
}
```

### 批量结果查询

请求地址

> POST       [https://miaohua.sensetime.com/api/v1b/task_results](https://miaohua.sensetime.com/api/v1b/task_results)


请求参数

- 一次可获取多个任务的结果，一次最多不超过50个任务
- 接口上限为6次/秒，根据实际情况调用

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| task_ids | list | 是 | 无 | 由任务创建接口返回的任务id数组 |
| token | string | 是 | 无 | 由get_token获取的token |
| sensitive_limit_value | float | 否 | 0 | 0-1，设置敏感图片鉴定阈值，越大代表越严格, 0采用系统默认标准|


请求示例

**curl 示例**

```
curl https://miaohua.sensetime.com/api/v1b/task_results \
  -H "Content-Type: application/json" \
  -d '{
        "task_ids": [
          "f3e5b59c-7416-11ed-a160-00163e025c94"
        ],
        "token": "",
  }'
```

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/task_results'
data = {
    "task_ids": ["f3e5b59c-7416-11ed-a160-00163e025c94"], # string 任务id
    "token": "", # get_token获取的token
}

response = requests.post(url, json=data)

print(response.status_code)
print(response.text)
```

返回示例

```json
{
    "code": 0,
    "info": [
      {
        "state": "done", // string 任务状态，未完成会是 "pending"
        "error_msg": "Intern error", // string 错误信息
        "images": [
            {
                "task_id": "f3e5b59c-7416-11ed-a160-00163e025c94",
                "task_type": "txt2img",
                "large": "https://pic.bakamaka.io/txt2img-result/2022-12-10/31e17f66-788b-11ed-98e2-00163e025c94_00000.jpg",
                "middle": "https://pic.bakamaka.io/txt2img-result/2022-12-10/31e17f66-788b-11ed-98e2-00163e025c94_00000.mid.jpg",
                "small": "https://pic.bakamaka.io/txt2img-result/2022-12-10/31e17f66-788b-11ed-98e2-00163e025c94_00000.mid.jpg",
                "save_index": 0
            }
        ],
        "select_seed": 8206464,#种子数
      },
      {...}
    ]
}
```

### 图生文 img2txt (同步接口)

请求地址

> POST       [https://miaohua.sensetime.com/api/v1b/img2txt](https://miaohua.sensetime.com/api/v1b/img2txt)


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| init_img | string | 是 | 无 | 初始图片路径 |
| token | string | 是 | 无 | 由get_token获取的token |


请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/img2txt'
data = {
    "init_img": "https://bkmk.oss-accelerate.aliyuncs.com/900ea63e-e1dd-11ed-bef5-00163e005161?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317042257726&Signature=x7nUVN6xpDustx4K02WOTjuty4Q%3D", # 初始图片url
    "token": "", # get_token获取的token
}

response = requests.post(url, json=data)

print(response.json())
```

返回示例

```json
{
  "code": 0,
  "info": "a very large clock tower with many other buildings behind it", # 图生文返回信息
  "msg": ""
}
```

### 获取生成表单

> GET       [https://miaohua.sensetime.com/api/v1b/get_generation_form](https://miaohua.sensetime.com/api/v1b/get_generation_form)


请求参数

无

请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/get_generation_form'

response = requests.get(url)

print(response.status_code)
print(response.text)
```

返回示例

```json
{
  "code": 0,
  "info": [
    {
      "choices": [
        "Artist v0.3.0 Beta",
      ],
      "default": "Artist v0.3.0 Beta",
      "description": "model name",
      "name": "model_name",
      "type": "list"
    },
    {
      "default": "",
      "description": "prompt",
      "name": "prompt",
      "type": "string"
    },
    {
      "default": "",
      "description": "negative prompt",
      "name": "neg_prompt",
      "type": "string"
    },
    {
      "default": 2,
      "description": "number of images",
      "max": 8,
      "min": 1,
      "name": "n_images",
      "type": "int"
    },
    {
      "default": 7,
      "description": "Guidance Scale",
      "max": 20,
      "min": 1,
      "name": "scale",
      "type": "int"
    },
    {
      "default": -1,
      "description": "seed",
      "max": 9999999,
      "min": 0,
      "name": "select_seed",
      "type": "int"
    },
    {
      "choices": {
        "1:1": [
          "512x512",
          "640x640",
          "768x768",
          "1024x1024"
        ],
        "2:3": [
          "512x768",
          "640x960",
          "768x1152",
          "1024x1536"
        ],
        "3:2": [
          "768x512",
          "960x640",
          "1152x768",
          "1536x1024"
        ]
      },
      "default": "512x512",
      "description": "resolution",
      "name": "resolution",
      "type": "dict"
    },
    {
      "default": "",
      "description": "init image url",
      "name": "init_img",
      "type": "str"
    },
    {
      "default": "",
      "description": "controlnet model",
      "name": "controlnet_model",
      "type": "str",
      "choices": ["", "canny"],
    },
  ],
  "msg": ""
}
```

### 获取模型：基模型

获取公开的基模型

请求地址

> POST       [https://miaohua.sensetime.com/api/v1b/models/base](https://miaohua.sensetime.com/api/v1b/models/base)


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| token | string | 是 | 无，通过get_token获得 | 通过get_token获取的token |
| page | int | 否 | 1，第几页 | 第几页 |
| per_page | int | 否 | 20，每页数量 | 每页数量 |
| query | string | 否 | 无，查询 | 通过模型名、tag等过滤 |


请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/models/base'

response = requests.post(url, json={"token": "xxx"})

print(response.status_code)
print(response.text)
```

返回示例

```json
{
    "code": 0,
    "info": {
        "has_next": true,
        "models": [
            {
                "author": "xxx",
                "base_model": "xxxx",
                "demo_picture": "https://bkmk.oss-accelerate.aliyuncs.com/8b53e958-374e-11ee-9f88-00163e253f9a_00001_XvYNa9vW_raw.jpeg?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317051652108&Signature=4NyJBRoZk86xleogKqp9ud3M60U%3D",
                "description": "",
                "is_opensource": true,
                "model_id": xx,
                "model_name": "meinamix_meinaV11",
                "tag": "upload",
                "versionid": "xxx"
            }
        ],
        "page": 1,
        "per_page": 1
    },
    "msg": ""
}
```

### 获取模型：LoRA模型

获取公开的LoRA模型, 可跟基模型lora merge

请求地址

> POST       [https://miaohua.sensetime.com/api/v1b/models/lora](https://miaohua.sensetime.com/api/v1b/models/lora)


**_注意： 目前数据正在补齐中，自定义的lora可以同 api/v1b/models/private中获取_**

请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| token | string | 是 | 无，通过get_token获得 | 通过get_token获取的token |
| page | int | 否 | 1，第几页 | 第几页 |
| per_page | int | 否 | 20，每页数量 | 每页数量 |
| query | string | 否 | 无，查询 | 通过模型名、tag等过滤 |


请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/models/lora'

response = requests.post(url, json={"token": "xxx"})

print(response.status_code)
print(response.text)
```

返回示例

```json
{
    "code": 0,
    "info": {
        "has_next": true,
        "models": [
            {
                "author": "xxx",
                "base_model": "",
                "demo_picture": "https://bkmk.oss-accelerate.aliyuncs.com/8b53e958-374e-11ee-9f88-00163e253f9a_00001_XvYNa9vW_raw.jpeg?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317051652108&Signature=4NyJBRoZk86xleogKqp9ud3M60U%3D",
                "description": "",
                "is_opensource": true,
                "model_id": 5,
                "model_name": "Baby Roy盲盒",
                "tag": "LORA",
                "versionid": "xxx"
            }
        ],
        "page": 1,
        "per_page": 1
    },
    "msg": ""
}
```

### 获取模型：私有模型(用户上传/训练)

获取自己上传的基模型/LoRA模型、训练的LoRA模型, 可跟基模型lora merge

请求地址

> POST       [https://miaohua.sensetime.com/api/v1b/models/private](https://miaohua.sensetime.com/api/v1b/models/private)


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| token | string | 是 | 无，通过get_token获得 | 通过get_token获取的token |
| page | int | 否 | 1，第几页 | 第几页 |
| per_page | int | 否 | 20，每页数量 | 每页数量 |
| query | string | 否 | 无，查询 | 通过模型名、tag等过滤 |


请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/models/private'

response = requests.post(url, json={"token": "xxx"})

print(response.status_code)
print(response.text)
```

返回示例

```json
{
    "code": 0,
    "info": {
        "has_next": true,
        "models": [
            {
                "author": "xxx",
                "base_model": "",
                "demo_picture": "https://bkmk.oss-accelerate.aliyuncs.com/8b53e958-374e-11ee-9f88-00163e253f9a_00001_XvYNa9vW_raw.jpeg?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317051652108&Signature=4NyJBRoZk86xleogKqp9ud3M60U%3D",
                "description": "",
                "is_opensource": true,
                "model_id": 5,
                "model_name": "Baby Roy盲盒",
                "tag": "LORA",
                "versionid": "xxx"
            }
        ],
        "page": 1,
        "per_page": 1
    },
    "msg": ""
}
```

### 获取Controlnet列表: Artist

获取自研模型支持的controlnet列表， 可用于推理生图控制

请求地址

> POST       [https://miaohua.sensetime.com/api/v1b/cn/artist](https://miaohua.sensetime.com/api/v1b/cn/artist)


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| token | string | 是 | 无，通过get_token获得 | 通过get_token获取的token |


请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/cn/artist'

response = requests.post(url, json={"token": "xxx"})

print(response.status_code)
print(response.text)
```

返回示例

```json
{
    "code": 0,
    "info": [
        {
            "demo_picture": "https://bkmk.oss-accelerate.aliyuncs.com/75ccd522-3fd7-11ee-b072-00163e253f9a.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317052590512&Signature=UFmUGW1nalt2Ni2HlTkikgV%2Byk4%3D",
            "demo_preview": "https://bkmk.oss-accelerate.aliyuncs.com/b9a6b330-3fd7-11ee-b072-00163e253f9a.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317052590627&Signature=XCcGGQCm%2BmxcDS2pajjBHABjgWo%3D",
            "demo_result": "https://bkmk.oss-accelerate.aliyuncs.com/0e88ca3c-3fd8-11ee-abdf-00163e253f9a.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317052590768&Signature=rvY7Y%2Bd9fgRg5aUvo4WtZC4IJyw%3D",
            "desc": "Onepose: Extracting Character Pose/Pose Recognition can use the figure uploaded by the user as a reference picture for AI painting, extract the pose and movement of the character, and generate a picture of the same pose, with accurate and stunning results.",
            "local_name": "Pose",
            "name": "openpose"
        }
    ],
    "msg": ""
}
```

### 获取Controlnet列表: 开源

获取开源模型支持的controlnet列表， 可用于推理生图控制

请求地址

> POST       [https://miaohua.sensetime.com/api/v1b/cn/opensource](https://miaohua.sensetime.com/api/v1b/cn/opensource)


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| token | string | 是 | 无，通过get_token获得 | 通过get_token获取的token |


请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/cn/opensource'

response = requests.post(url, json={"token": "xxx"})

print(response.status_code)
print(response.text)
```

返回示例

```json
{
    "code": 0,
    "info": [
        {
            "demo_picture": "https://bkmk.oss-accelerate.aliyuncs.com/75ccd522-3fd7-11ee-b072-00163e253f9a.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317052590512&Signature=UFmUGW1nalt2Ni2HlTkikgV%2Byk4%3D",
            "demo_preview": "https://bkmk.oss-accelerate.aliyuncs.com/b9a6b330-3fd7-11ee-b072-00163e253f9a.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317052590627&Signature=XCcGGQCm%2BmxcDS2pajjBHABjgWo%3D",
            "demo_result": "https://bkmk.oss-accelerate.aliyuncs.com/0e88ca3c-3fd8-11ee-abdf-00163e253f9a.png?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317052590768&Signature=rvY7Y%2Bd9fgRg5aUvo4WtZC4IJyw%3D",
            "desc": "Onepose: Extracting Character Pose/Pose Recognition can use the figure uploaded by the user as a reference picture for AI painting, extract the pose and movement of the character, and generate a picture of the same pose, with accurate and stunning results.",
            "local_name": "Pose",
            "name": "openpose"
        }
    ],
    "msg": ""
}
```

## 以下为训练相关

### 上传图片

请求地址

> POST       [https://miaohua.sensetime.com/api/v1b/upload_img](https://miaohua.sensetime.com/api/v1b/upload_img)
Authorization: Bearer token_string


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| init_img | file | 是 | 无 | 图片文件,为单个，以'init_img'为key |


请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/upload_imgs'
files={'init_img': open('/local/pic1.png','rb')}
headers = {"Authorization": "Bearer MYREALLYLONGTOKENIGOT"}

response = requests.post(url, files=files, headers=headers)

print(response.status_code)
print(response.json())
```

返回示例

```json
{
  "code": 0,
  "info": [
    "https://bkmk.oss-accelerate.aliyuncs.com/900ea63e-e1dd-11ed-bef5-00163e005161?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317042257726&Signature=x7nUVN6xpDustx4K02WOTjuty4Q%3D"
  ],
  "msg": ""
}
```

### 批量上传图片

请求地址

**20230809更新：header中添加Authorization 以识别身份, 在未来版本中将会移除file token**

> POST       [https://miaohua.sensetime.com/api/v1b/upload_imgs](https://miaohua.sensetime.com/api/v1b/upload_imgs)
Authorization: Bearer token_string


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| init_img | file | 是 | 无 | 图片文件,可以多个，都以'init_img'为key |


请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/upload_imgs'
# files的列表格式，以init_img为key
img_paths = ['/local/pic1.png', '/local/pic2.png',]
files = [('init_img', open(img_path, 'rb')) for img_path in img_paths]
headers = {"Authorization": "Bearer MYREALLYLONGTOKENIGOT"}


response = requests.post(url, files=files, headers=headers)

print(response.status_code)
print(response.json())
```

返回示例

```json
{
  "code": 0,
  "info": [
    "https://bkmk.oss-accelerate.aliyuncs.com/900ea63e-e1dd-11ed-bef5-00163e005161?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317042257726&Signature=x7nUVN6xpDustx4K02WOTjuty4Q%3D",
    "https://bkmk.oss-accelerate.aliyuncs.com/91d5e7d4-e1dd-11ed-bef5-00163e005161?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317042257730&Signature=d1weIkwFwVK9nbCrPNdkeVm1wQA%3D"
  ],
  "msg": ""
}
```

### 创建数据集

请求地址

> POST       [https://miaohua.sensetime.com/api/v1b/dataset](https://miaohua.sensetime.com/api/v1b/dataset)


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| token | string | 是 | 无，通过get_token获得 | 通过get_token获取的token |
| name | string | 是 | 无 | 数据集名称 |
| description | string | 否 | "" | 数据集描述 |
| images | list | 是 | 无 | 图片列表, url list，可通过批量上传图片接口/api/v1b/upload_imgs获取 |


请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/dataset'
data = {
  "token": "xxxx", # get_token获取到的token
  "name": "dataset1", # 数据集名称
  "description": "dataset1 description",  # 数据集描述
  "images": [
    "https://bkmk.oss-accelerate.aliyuncs.com/900ea63e-e1dd-11ed-bef5-00163e005161?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317042257726&Signature=x7nUVN6xpDustx4K02WOTjuty4Q%3D",
    "https://bkmk.oss-accelerate.aliyuncs.com/91d5e7d4-e1dd-11ed-bef5-00163e005161?OSSAccessKeyId=LTAI5tPynodLHeacT1J5SmWh&Expires=317042257730&Signature=d1weIkwFwVK9nbCrPNdkeVm1wQA%3D"
  ] # 图片列表，url list
}

response = requests.post(url, json=data)

print(response.status_code)
print(response.text)
```

返回示例

```json
{'code': 0, 'info': {'dataset_id': 11}, 'msg': ''}
```

### 获取已创建的全部数据集

请求地址

> POST       [https://miaohua.sensetime.com/api/v1b/dataset_all?page=1](https://miaohua.sensetime.com/api/v1b/dataset_all?page=1)


url参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| page | string | 否 |  | 第几页 |


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| token | string | 是 | 无，通过get_token获得 | 通过get_token获取的token |


请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/dataset_all'
data = {
  "token": "xxx", # get_token获取到的token
}

response = requests.post(url, json=data)

print(response.status_code)
print(response.text)
```

返回示例

```json
{
  "items": [
    {
      "id": 1,   # dataset id
      "name": "dataset1",  # dataset 名称
      "description": "dataset1 description",  # dataset 描述
      "demo_picture": "url",   # 封面图
      "create_time": "2023-04-23 00:00:00",  # 创建时间
      "update_time": "2023-04-23 00:00:00",  # 修改时间
      "images": [
        "url1",
        "url2"
      ], # 数据集图片列表
      "images_num": 2 # 数据集图片数量
    },
    {
      "id": 2,
      "name": "dataset2",
      "description": "dataset2 description",
      "demo_picture": "url",
      "create_time": "2023-04-23 00:00:00",
      "update_time": "2023-04-23 00:00:00",
      "images": [
        "url3"
      ],
      "images_num": 1
    }
  ],
  "page": 1,
  "per_page": 20,
  "has_next": false,
  "has_prev": false,
  "totalPages": 1
}
```

### 数据集中添加图片

请求地址

> POST       [https://miaohua.sensetime.com/api/v1b/dataset/add_images](https://miaohua.sensetime.com/api/v1b/dataset/add_images)


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| token | string | 是 | 无，通过get_token获得 | 通过get_token获取的token |
| images | array | 是 | 无，图片列表 | 通过upload_img和upload_imgs得到的oss 图片链接 |
| dataset | int | 是 | 无，dataset id | 通过/api/v1b/dataset_all获取 |


请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/dataset/add_images'
data = {
  "token": "xxx", # get_token获取到的token
  "dataset": 1,
  "images": [
    "https://bkmk.oss-accelerate.aliyuncs.com/xxxx.JPEG?OSSAccessKeyId=xxx&Signature=xxx"
  ]
}

response = requests.post(url, json=data)

print(response.status_code)
print(response.text)
```

返回示例

成功

```json
{
    "code": 0,
    "info": "",
    "msg": "OK"
}
```

失败

```json
{
  "code": 400,
  "info": "",
  "msg": "invalid image"
}
```

### 数据集中删除图片

请求地址

> POST       [https://miaohua.sensetime.com/api/v1b/dataset/delete_images](https://miaohua.sensetime.com/api/v1b/dataset/delete_images)


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| token | string | 是 | 无，通过get_token获得 | 通过get_token获取的token |
| images | array | 是 | 无，图片列表 | 通过upload_img和upload_imgs得到的oss 图片链接 |
| dataset | int | 是 | 无，dataset id | 通过/api/v1b/dataset_all获取 |


请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/dataset/delete_images'
data = {
  "token": "xxx", # get_token获取到的token
  "dataset": 1,
  "images": [
    "https://bkmk.oss-accelerate.aliyuncs.com/xxxx.JPEG?OSSAccessKeyId=xxx&Signature=xxx"
  ]
}

response = requests.post(url, json=data)

print(response.status_code)
print(response.text)
```

返回示例

成功

```json
{
    "code": 0,
    "info": "",
    "msg": "OK"
}
```

失败

```json
{
  "code": 400,
  "info": "",
  "msg": "invalid image"
}
```

### 提交训练任务

请求地址

> POST       [https://miaohua.sensetime.com/api/v1b/train_submit](https://miaohua.sensetime.com/api/v1b/train_submit)


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| token | string | 是 | 无，通过get_token获得 | 通过get_token获取的token |
| name | string | 是 | 无 | 训练的LoRA模型名称 |
| description | string | 否 | "" | 训练的模型描述 |
| base_model | string | 否 | 系统指定 | 训练所需要的基本模型，可选模型范围通过/api/v1b/get_train_form获取 |
| trigger_word | string | 否 | "" | 触发词 |
| main_body | string | 是 | "Style" | 主体， ["Style", "Human", "Architecture", "Plants", "Animals"] |
| dataset | list | 是 | 无 | 训练所选的数据集id的list,需要先创建数据集 |


请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/train_submit'
data = {
    "token": "", # get_token获取的token
    "name": "train model1", # 训练的模型名称
    "description": "train model1 description",  # 训练的模型描述
    "base_model": "自动选择", # 基模型
    "trigger_word": "one girl", # 触发词
    "main_body": "Style", # 主体
    "dataset": [5, 7], # 选择的数据集的id组合，list
}

response = requests.post(url, json=data)

print(response.status_code)
print(response.text)
```

返回示例

```json
{'code': 0, 'info': {'task_id': 'xxxxx'}, 'msg': ''}
```

### 获取训练进度

请求地址

> POST       [https://miaohua.sensetime.com/api/v1b/train_progress](https://miaohua.sensetime.com/api/v1b/train_progress)


请求参数

| 参数名称 | 类型 | 是否必须 | 默认值 | 含义 |
| --- | --- | --- | --- | --- |
| token | string | 是 | 无，通过get_token获得 | 通过get_token获取的token |
| task_id | string | 是 | 无 | 训练任务的task_id |


请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/train_progress'
data = {
    "token": "", # get_token获取的token
    "task_id": "xxxxxx", # 训练任务的task_id
}

response = requests.post(url, json=data)

print(response.status_code)
print(response.text)
```

返回示例

```json
{'create_time': '2023-05-08 14:22:52', 'description': 'train model1 description', 'main_body': 'Style', 'model_name': 'train_model1', 'state': 'done', 'training_progress': 1.0, 'trigger_word': 'one girl'}
```

返回训练state为状态，training_progress为进度(0-1)，状态为'done'后，可通过之前的生成接口生成图片，模型的model_name取训练时取的名字。

### 获取可供训练的基模列表

请求地址

> GET       [https://miaohua.sensetime.com/api/v1b/get_train_form](https://miaohua.sensetime.com/api/v1b/get_train_form)


请求参数

无

请求示例

**python示例**

```python
import requests

url = 'https://miaohua.sensetime.com/api/v1b/get_train_form'

response = requests.get(url)

print(response.status_code)
print(response.text)
```

返回示例

```json
{
  "code": 0,
  "info": {
    "base_model": [
      "商汤自研作画模型 Beta",
      "水彩风",
    ],
    "main_body": [
      "Style",
      "Human",
      "Architecture",
      "Plants",
      "Animals"
    ]
  },
  "msg": ""
}
```

主要是base_model和main_body这两个字段。
