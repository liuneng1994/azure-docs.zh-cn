---
author: aahill
ms.service: cognitive-services
ms.topic: include
ms.date: 10/02/2019
ms.author: aahi
ms.openlocfilehash: 847b2d0489dc04b4275465dbe957b72418bbf1a4
ms.sourcegitcommit: 827248fa609243839aac3ff01ff40200c8c46966
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/07/2019
ms.locfileid: "73750170"
---
[参考文档](https://docs.microsoft.com/python/api/overview/azure/cognitiveservices/textanalytics?view=azure-python) | [库源代码](https://github.com/Azure/azure-sdk-for-ruby/tree/master/data/azure_cognitiveservices_textanalytics) | [包 (RubyGems)](https://rubygems.org/gems/azure_cognitiveservices_textanalytics) | [示例](https://github.com/Azure-Samples/cognitive-services-quickstart-code)

<a name="HOLTop"></a>

## <a name="prerequisites"></a>先决条件

* Azure 订阅 - [免费创建订阅](https://azure.microsoft.com/free/)
* 最新版本的 [Ruby](https://www.ruby-lang.org/)

## <a name="setting-up"></a>设置

### <a name="create-a-text-analytics-azure-resource"></a>创建文本分析 Azure 资源 

[!INCLUDE [text-analytics-resource-creation](resource-creation.md)]

### <a name="create-a-new-ruby-application"></a>创建新的 Ruby 应用程序

在控制台窗口（例如 cmd、PowerShell 或 Bash）中，为应用创建一个新目录并导航到该目录。 然后，创建名为 `GemFile` 的文件，并为你的代码创建一个 Ruby 文件。

```console
mkdir myapp && cd myapp
```

在 `GemFile` 中，添加以下行以将客户端库添加为依赖项。

```ruby
source 'https://rubygems.org'
gem 'azure_cognitiveservices_textanalytics', '~>0.17.3'
```

在 Ruby 文件中，导入以下包。

[!code-ruby[Import statements](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=includeStatement)]

为资源的 Azure 终结点和密钥创建分别名为 `TEXT_ANALYTICS_ENDPOINT` 和 `TEXT_ANALYTICS_SUBSCRIPTION_KEY` 的变量。 如果在启动应用程序后创建了环境变量，则需要关闭再重新打开运行该应用程序的编辑器、IDE 或 shell 才能访问该变量。 

[!INCLUDE [text-analytics-find-resource-information](../find-azure-resource-info.md)]


[!code-ruby[endpoint, key variables](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=vars)]

## <a name="object-model"></a>对象模型 

文本分析客户端使用你的密钥向 Azure 进行身份验证。 该客户端提供了几种方法来分析文本，文本可以是单个字符串，也可以是批处理。 

文本将以 `documents` 的列表的形式发送到 API，该项是包含 `id`、`text` 和 `language` 属性的组合的 `dictionary` 对象，具体取决于所用的方法。 `text` 属性存储要以源 `language` 分析的文本，而 `id` 则可以是任何值。 

响应对象是一个列表，其中包含每个文档的分析信息。 

## <a name="code-examples"></a>代码示例

这些代码片段演示如何使用适用于 Python 的文本分析客户端库执行以下操作：

* [对客户端进行身份验证](#authenticate-the-client)
* [情绪分析](#sentiment-analysis)
* [语言检测](#language-detection)
* [实体识别](#entity-recognition)
* [关键短语提取](#key-phrase-extraction)

## <a name="authenticate-the-client"></a>验证客户端

创建名为的 `TextAnalyticsClient` 的类。 

```ruby
class TextAnalyticsClient
  @textAnalyticsClient
  #...
end
```

在此类中，创建名为 `initialize` 的函数以对客户端进行身份验证。 使用你的 `TEXT_ANALYTICS_SUBSCRIPTION_KEY` 和 `TEXT_ANALYTICS_ENDPOINT` 环境变量。 

[!code-ruby[initialize function for authentication](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=initialize)]

在类的外部，使用客户端的 `new()` 函数对其进行实例化。

[!code-ruby[client creation](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=clientCreation)] 

<a name="SentimentAnalysis"></a>

## <a name="sentiment-analysis"></a>情绪分析

在客户端对象中，创建名为 `AnalyzeSentiment()` 的函数，该函数采用稍后将创建的输入文档的列表。 调用客户端的 `sentiment()` 函数并获取结果。 然后循环访问结果，输出每个文档的 ID 和情绪分数。 评分接近 0 表示消极情绪，评分接近 1 表示积极情绪。

[!code-ruby[client method for sentiment analysis](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=analyzeSentiment)] 

在客户端函数外部，创建名为 `SentimentAnalysisExample()` 的新函数，该函数使用以前创建的 `TextAnalyticsClient` 对象。 创建 `MultiLanguageInput` 对象的列表，其中包含需分析的文档。 每个对象会包含 `id`、`Language` 和 `text` 属性。 `text` 属性存储要分析的文本，`language` 是文档的语言，`id` 则可以是任何值。 然后调用客户端的 `AnalyzeSentiment()` 函数。

[!code-ruby[sentiment analysis document creation and call](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=sentimentCall)] 

调用 `SentimentAnalysisExample()` 函数。

```ruby
SentimentAnalysisExample(textAnalyticsClient)
```

### <a name="output"></a>输出

```console
===== SENTIMENT ANALYSIS =====
Document ID: 1 , Sentiment Score: 0.87
Document ID: 2 , Sentiment Score: 0.11
Document ID: 3 , Sentiment Score: 0.44
Document ID: 4 , Sentiment Score: 1.00
```

<a name="LanguageDetection"></a>

## <a name="language-detection"></a>语言检测

在客户端对象中，创建名为 `DetectLanguage()` 的函数，该函数采用稍后将创建的输入文档的列表。 调用客户端的 `detect_language()` 函数并获取结果。 然后循环访问结果，输出每个文档的 ID 和检测到的语言。

[!code-ruby[client method for language detection](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=detectLanguage)] 

在客户端函数外部，创建名为 `DetectLanguageExample()` 的新函数，该函数使用以前创建的 `TextAnalyticsClient` 对象。 创建 `LanguageInput` 对象的列表，其中包含需分析的文档。 每个对象会包含 `id` 和 `text` 属性。 `text` 属性存储要分析的文本，而 `id` 则可以是任何值。 然后调用客户端的 `DetectLanguage()` 函数。

[!code-ruby[language detection document creation and call](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=detectLanguageCall)] 

调用 `DetectLanguageExample()` 函数。

```ruby
DetectLanguageExample(textAnalyticsClient)
```

### <a name="output"></a>输出

```console
===== LANGUAGE EXTRACTION ======
Document ID: 1 , Language: English
Document ID: 2 , Language: Spanish
Document ID: 3 , Language: Chinese_Simplified
```

<a name="EntityRecognition"></a>

## <a name="entity-recognition"></a>实体识别

在客户端对象中，创建名为 `RecognizeEntities()` 的函数，该函数采用稍后将创建的输入文档的列表。 调用客户端的 `entities()` 函数并获取结果。 然后循环访问结果，输出每个文档的 ID 和识别的实体。

[!code-ruby[client method for entity recognition](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=recognizeEntities)]

在客户端函数外部，创建名为 `RecognizeEntitiesExample()` 的新函数，该函数使用以前创建的 `TextAnalyticsClient` 对象。 创建 `MultiLanguageInput` 对象的列表，其中包含需分析的文档。 每个对象会包含 `id`、`language` 和 `text` 属性。 `text` 属性存储要分析的文本，`language` 是文本的语言，`id` 则可以是任何值。 然后调用客户端的 `RecognizeEntities()` 函数。

[!code-ruby[entity recognition documents and method call](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=recognizeEntitiesCall)] 

调用 `RecognizeEntitiesExample()` 函数。

```ruby
RecognizeEntitiesExample(textAnalyticsClient)
```

### <a name="output"></a>输出

```console
===== ENTITY RECOGNITION =====
Document ID: 1
        Name: Microsoft,        Type: Organization,     Sub-Type: N/A
        Offset: 0, Length: 9,   Score: 1.0

        Name: Bill Gates,       Type: Person,   Sub-Type: N/A
        Offset: 25, Length: 10, Score: 0.999847412109375

        Name: Paul Allen,       Type: Person,   Sub-Type: N/A
        Offset: 40, Length: 10, Score: 0.9988409876823425

        Name: April 4,  Type: Other,    Sub-Type: N/A
        Offset: 54, Length: 7,  Score: 0.8

        Name: April 4, 1975,    Type: DateTime, Sub-Type: Date
        Offset: 54, Length: 13, Score: 0.8

        Name: BASIC,    Type: Other,    Sub-Type: N/A
        Offset: 89, Length: 5,  Score: 0.8

        Name: Altair 8800,      Type: Other,    Sub-Type: N/A
        Offset: 116, Length: 11,        Score: 0.8

Document ID: 2
        Name: Microsoft,        Type: Organization,     Sub-Type: N/A
        Offset: 21, Length: 9,  Score: 0.999755859375

        Name: Redmond (Washington),     Type: Location, Sub-Type: N/A
        Offset: 60, Length: 7,  Score: 0.9911284446716309

        Name: 21 kilómetros,    Type: Quantity, Sub-Type: Dimension
        Offset: 71, Length: 13, Score: 0.8

        Name: Seattle,  Type: Location, Sub-Type: N/A
        Offset: 88, Length: 7,  Score: 0.9998779296875
```

<a name="KeyPhraseExtraction"></a>

## <a name="key-phrase-extraction"></a>关键短语提取

在客户端对象中，创建名为 `ExtractKeyPhrases()` 的函数，该函数采用稍后将创建的输入文档的列表。 调用客户端的 `key_phrases()` 函数并获取结果。 然后循环访问结果，输出每个文档的 ID 以及提取的密钥短语。

[!code-ruby[key phrase extraction client method](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=extractKeyPhrases)] 

在客户端函数外部，创建名为 `KeyPhraseExtractionExample()` 的新函数，该函数使用以前创建的 `TextAnalyticsClient` 对象。 创建 `MultiLanguageInput` 对象的列表，其中包含需分析的文档。 每个对象会包含 `id`、`language` 和 `text` 属性。 `text` 属性存储要分析的文本，`language` 是文本的语言，`id` 则可以是任何值。 然后调用客户端的 `ExtractKeyPhrases()` 函数。

[!code-ruby[key phrase document creation and call](~/cognitive-services-ruby-sdk-samples/samples/text_analytics.rb?name=keyPhrasesCall)]


调用 `KeyPhraseExtractionExample()` 函数。

```ruby
KeyPhraseExtractionExample(textAnalyticsClient)
```

### <a name="output"></a>输出

```console
Document ID: 1
         Key phrases:
                幸せ
Document ID: 2
         Key phrases:
                Stuttgart
                Hotel
                Fahrt
                Fu
Document ID: 3
         Key phrases:
                cat
                rock
Document ID: 4
         Key phrases:
                fútbol
```
