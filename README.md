# 本地图片批量字体识别

`Batchfontsearch-localimages.py` 调用识字 API，批量识别本地图片中的文字与字体信息。脚本会先查询企业剩余调用次数，再处理指定文件夹中的图片，并将每张图片的完整接口响应或错误写入结果文件。

## 环境准备

- Python 3.8 或更高版本
- 可访问识字 API 的网络环境及有效的企业鉴权信息
- 第三方依赖：`requests`

安装依赖：

```bash
python -m pip install requests
```

## 配置与运行

打开 `Batchfontsearch-localimages.py`，按实际环境修改以下内容：

1. 在 `ZiYouConfig` 中填写 API 地址、企业代码和签名密钥（`base_url`、`company_code`、`company_slat`）。请勿将真实密钥提交到公开仓库。
2. 在脚本末尾将 `image_folder` 改为待识别图片所在文件夹。
3. 将 `output_txt` 改为期望的结果文件路径；脚本会自动创建其父目录。
4. 如需调整并发量，修改 `batch_identify(..., max_workers=3)` 中的线程数；使用 `None` 则顺序处理。

在项目目录运行：

```bash
python Batchfontsearch-localimages.py
```

脚本只扫描 `image_folder` 的当前层级，不递归扫描子文件夹。支持小写扩展名的 `.jpg`、`.jpeg`、`.png`、`.bmp` 和 `.gif` 文件。如果没有找到图片，会提示检查路径，不会生成批量结果文件。

> 脚本定义了 `ZiYouConfig.from_env()`，可读取 `ZY_BASE_URL`、`ZY_COMPANY_CODE`、`ZY_COMPANY_SLAT` 和 `ZY_TIMEOUT`。当前主程序使用的是 `ZiYouConfig()`，因此直接设置环境变量**不会**改变运行配置；如需使用环境变量，应将主程序中的初始化语句改为 `config = ZiYouConfig.from_env()`。

## 结果格式

结果文件虽以 `.txt` 命名，内容是 UTF-8 编码的 JSON 数组，顺序与输入图片一致。每条记录包含图片路径，以及完整的 API 响应或错误信息：

```json
[
  {
    "image": "path/to/image1.png",
    "response": {
      "success": true,
      "result": {
        "text_lines": []
      }
    }
  },
  {
    "image": "path/to/image2.jpg",
    "error": "错误信息"
  }
]
```

控制台还会打印每张图片的完整响应及成功、失败数量。单张图片请求失败会记录在结果中，继续处理其他图片；网络、HTTP 或接口返回的错误信息也会出现在相应记录中。结果可能包含识别出的图片文字，请按数据使用要求保存和分享。

## 在其他 Python 代码中使用

脚本提供 `ZiYouOCRClient`，可识别本地文件、图片字节或图片 URL；`batch_identify` 接收 `{"file": "..."}` 和 `{"url": "..."}` 两种输入。由于文件名含连字符，常规 `import Batchfontsearch-localimages` 语法不可用；若要作为模块导入，可先将文件重命名为符合 Python 模块命名规则的名称。
