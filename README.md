# AWS Bedrock Quick Start Guide 快速上手指南
欢迎使用AWS Bedrock快速上手指南。本指南提供了如何快速设置和开始使用AWS Bedrock的说明以及必要的代码。

如果你需要更多的功能以及了解更多的细节，请参考AWS官方文档
[AWS Bedrock Official Documentation](https://docs.aws.amazon.com/bedrock/latest/userguide/what-is-bedrock.html)
[AWS Bedrock Official API](https://docs.aws.amazon.com/zh_cn/bedrock/latest/userguide/model-parameters-anthropic-claude-messages.html)


## 前置条件
在代码的运营环境安装boto3 (注意boto3 版本需要在1.28.59以上)
``` bash
pip install boto3 >=1.28.59
```

## Claude3 代码 （文生文）
以下是一段可以快速执行的Claude3 python代码
``` python
bedrock_runtime = boto3.client(service_name='bedrock-runtime', region_name='us-east-1', aws_access_key_id='ACCESS_KEY',
    aws_secret_access_key='SECRET_KEY')
    
payload = {
    "modelId": "anthropic.claude-3-sonnet-20240229-v1:0",
    "contentType": "application/json",
    "accept": "application/json",
    "body": {
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 1000,
        "messages": [
            {
                "role": "user",
                "content": [
                    {
                        "type": "text",
                        "text": "告诉我你是谁"
                    }
                ]
            }
        ]
    }
}


body_bytes = json.dumps(payload['body']).encode('utf-8')

response = bedrock_runtime.invoke_model(
    body=body_bytes,
    contentType=payload['contentType'],
    accept=payload['accept'],
    modelId=payload['modelId']
)


response_body = response['body'].read().decode('utf-8')
print(response_body)

```
响应和返回 
``` json
{
	"id": "msg_01Lyao2g9yt7wcDv3SXgsRuA",
	"type": "message",
	"role": "assistant",
	"content": [{
		"type": "text",
		"text": "您好,我是一个基于大型语言模型训练而成的人工智能助理。我可以回答各种问题,并协助完成诸如写作、分析、编程等多项任务。我虽然是由机器学习算法创建,但会努力以理性、客观和有益的方式回应您,并尽量避免出现有偏差或不当的言行。我没有真正的身份,只是一个旨在帮助和服务人类的工具。很高兴能与您交流,希望我们的对话会让您获得一些有价值的信息或帮助。"
	}],
	"model": "claude-3-sonnet-28k-20240229",
	"stop_reason": "end_turn",
	"stop_sequence": null,
	"usage": {
		"input_tokens": 16,
		"output_tokens": 180
	}
}
```

## Claude3 代码 （多模态）

1. 应用视觉能力，需要将文件以base64的形式输入作为input给到大模型。
2. 支持的图片格式目前包括  JPEG, PNG, GIF以及 WebP。
3. token cost = (width px * height px)/750.
4. 可以在单个接口里面上传多张图片。
5. 多模态场景下，如果你用中文提问，大模型默认会使用英文回答，需要显性的在system prompt或者message里面指定回复的语言。


``` python
#多模态需要将文件以base64的形式输入给大模型
with open("aws.png", "rb") as image_file:
    encoded_string = base64.b64encode(image_file.read())
    base64_string = encoded_string.decode('utf-8')

# Create a BedrockRuntime client
bedrock_runtime = boto3.client(service_name='bedrock-runtime', region_name='us-east-1', aws_access_key_id='ACCESS_KEY',
    aws_secret_access_key='SECRET_KEY')

payload = {
    "modelId": "anthropic.claude-3-sonnet-20240229-v1:0",
    "contentType": "application/json",
    "accept": "application/json",
    "body": {
        "anthropic_version": "bedrock-2023-05-31",
        "max_tokens": 1000,
        "messages": [
            {
                "role": "user",
                "content": [
                    {
                        "type": "image",
                        "source": {
                            "type": "base64",
                            "media_type": "image/png",
                            "data": base64_string
                        }
                    },
                    {
                        "type": "text",
                        "text": "告诉我图片中有什么内容"
                    }
                ]
            }
        ]
    }
}


```

* 是使用Claude3文生文的参考代码 `/python/bedrock_claude3.py`
* 是使用Claude3图片视觉的参考代码（多模态）  `/python/bedrock_claude3_vision.py`


## Marketplace 模型使用
AWS Bedrock Marketplace 提供了多种第三方模型供选择。以下是使用 Marketplace 上 DeepSeek 模型的示例：

* DeepSeek 模型调用示例 `/python/bedrock_marketplace_deepseek.py`
  - 支持多种聊天模板格式 (LLAMA, QWEN, DEEPSEEK)
  - 提供格式化提示词功能
  - 包含参数控制（max_tokens, temperature, top_p等）

## Mistral 模型使用
AWS Bedrock 提供了 Mistral 系列模型的访问。示例代码：

* Mistral 模型调用示例 `/python/bedrock_mistral.py`

## 跨区域推理
AWS Bedrock 支持跨区域模型推理，可以使用预配置的推理配置文件来实现：

* 跨区域推理示例 `/python/bedrock_claude3_cross_region_inference.py`
  - 使用预配置的推理配置文件
  - 支持不同区域的模型 ARN
  - 通过统一的模型标识符进行调用

## Thanks
Thank you for using AWS Bedrock!
