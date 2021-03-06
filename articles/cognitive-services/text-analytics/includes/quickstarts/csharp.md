---
author: aahill
ms.service: cognitive-services
ms.topic: include
ms.date: 10/28/2019
ms.author: aahi
ms.openlocfilehash: fd3d53dce398c445d309a19f1f58a8d298080c45
ms.sourcegitcommit: 827248fa609243839aac3ff01ff40200c8c46966
ms.translationtype: HT
ms.contentlocale: zh-CN
ms.lasthandoff: 11/07/2019
ms.locfileid: "73750162"
---
<a name="HOLTop"></a>

[参考文档](https://docs.microsoft.com/dotnet/api/overview/azure/cognitiveservices/client/textanalytics?view=azure-dotnet-preview) | [库源代码](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/cognitiveservices/Language.TextAnalytics) | [包 (NuGet)](https://www.nuget.org/packages/Microsoft.Azure.CognitiveServices.Language.TextAnalytics/) | [示例](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples)

> [!NOTE]
> 为简单起见，本文中的代码使用文本分析 .NET SDK 的同步方法。 对于生产方案，我们建议使用批处理的异步方法来提高性能和可伸缩性。 例如，调用 [SentimentBatchAsync()](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.textanalytics.textanalyticsclientextensions.sentimentbatchasync?view=azure-dotnet&viewFallbackFrom=azure-dotnet-preview) 而不是 [Sentiment()](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.textanalytics.textanalyticsclientextensions.sentiment?view=azure-dotnet)。

## <a name="prerequisites"></a>先决条件

* Azure 订阅 - [免费创建订阅](https://azure.microsoft.com/free/)
* 最新版本的 [.NET Core SDK](https://dotnet.microsoft.com/download/dotnet-core)。

## <a name="setting-up"></a>设置

### <a name="create-a-text-analytics-azure-resource"></a>创建文本分析 Azure 资源

[!INCLUDE [text-analytics-resource-creation](resource-creation.md)]

### <a name="create-a-new-net-core-application"></a>创建新的 .NET Core 应用程序

在控制台窗口（例如 cmd、PowerShell 或 Bash）中，使用 `dotnet new` 命令创建名为 `text-analytics quickstart` 的新控制台应用。 此命令将创建包含单个 C# 源文件的简单“Hello World”项目：program.cs  。 

```console
dotnet new console -n text-analytics-quickstart
```

将目录更改为新创建的应用文件夹。 可使用以下代码生成应用程序：

```console
dotnet build
```

生成输出不应包含警告或错误。 

```console
...
Build succeeded.
 0 Warning(s)
 0 Error(s)
...
```

从项目目录中，打开 Program.cs  文件，并添加以下 `using` 指令：

[!code-csharp[Import directives](~/cognitive-services-dotnet-sdk-samples/samples/TextAnalytics/synchronous/Program.cs?name=imports)]

在应用程序的 `Program` 类中，为资源的密钥以及先前创建的环境变量中的终结点创建变量。 如果在开始编辑应用程序后创建了这些环境变量，则需要关闭并重新打开用于访问这些变量的编辑器、IDE 或 shell。

[!INCLUDE [text-analytics-find-resource-information](../find-azure-resource-info.md)]

[!code-csharp[initial variables](~/cognitive-services-dotnet-sdk-samples/samples/TextAnalytics/synchronous/Program.cs?name=vars)]

替换应用程序的 `Main` 方法。 稍后将定义此处调用的方法。

[!code-csharp[main method](~/cognitive-services-dotnet-sdk-samples/samples/TextAnalytics/synchronous/Program.cs?name=main)]

### <a name="install-the-client-library"></a>安装客户端库

在应用程序目录中，使用以下命令安装适用于 .NET 的文本分析客户端库：

```console
dotnet add package Microsoft.Azure.CognitiveServices.Language.TextAnalytics --version 4.0.0
```

## <a name="object-model"></a>对象模型

文本分析客户端是一个 [TextAnalyticsClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.textanalytics.textanalyticsclient?view=azure-dotnet) 对象，该对象使用你的密钥在 Azure 中进行身份验证，并提供用于接受文本（单个字符串或批）的函数。 可以同步方式或异步方式将文本发送到 API。 响应对象包含发送的每个文档的分析信息。 

## <a name="code-examples"></a>代码示例

* [对客户端进行身份验证](#authenticate-the-client)
* [情绪分析](#sentiment-analysis)
* [语言检测](#language-detection)
* [实体识别](#entity-recognition)
* [关键短语提取](#key-phrase-extraction)

## <a name="authenticate-the-client"></a>验证客户端

创建用于存储凭据的新 `ApiKeyServiceClientCredentials` 类，并将凭据发送到客户端的请求。 在该类中创建 `ProcessHttpRequestAsync()` 的重写，用于将密钥添加到 `Ocp-Apim-Subscription-Key` 标头。

[!code-csharp[Client class](~/cognitive-services-dotnet-sdk-samples/samples/TextAnalytics/synchronous/Program.cs?name=clientClass)]

创建一个方法，以实例化包含你的终结点的 [TextAnalyticsClient](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.textanalytics.textanalyticsclient?view=azure-dotnet) 对象和包含你的密钥的 `ApiKeyServiceClientCredentials` 对象。

[!code-csharp[Client authentication](~/cognitive-services-dotnet-sdk-samples/samples/TextAnalytics/synchronous/Program.cs?name=authentication)]

在程序的 `main()` 方法中，调用身份验证方法来实例化客户端。

## <a name="sentiment-analysis"></a>情绪分析

创建名为 `SentimentAnalysisExample()` 的新函数，用于提取前面创建的客户端，并调用该客户端的 [Sentiment()](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.textanalytics.textanalyticsclientextensions.sentiment?view=azure-dotnet) 函数。 如果成功，返回的 [SentimentResult](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.textanalytics.models.sentimentresult?view=azure-dotnet) 对象将包含情绪 `Score`，否则包含 `errorMessage`。 

评分接近 0 表示消极情绪，评分接近 1 表示积极情绪。

[!code-csharp[Sentiment analysis](~/cognitive-services-dotnet-sdk-samples/samples/TextAnalytics/synchronous/Program.cs?name=sentiment)]

### <a name="output"></a>输出

```console
Sentiment Score: 0.87
```

## <a name="language-detection"></a>语言检测

创建名为 `languageDetectionExample()` 的新函数，用于提取前面创建的客户端，并调用该客户端的 [DetectLanguage()](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.textanalytics.textanalyticsclientextensions.detectlanguage?view=azure-dotnet#Microsoft_Azure_CognitiveServices_Language_TextAnalytics_TextAnalyticsClientExtensions_DetectLanguage_Microsoft_Azure_CognitiveServices_Language_TextAnalytics_ITextAnalyticsClient_System_String_System_String_System_Nullable_System_Boolean__System_Threading_CancellationToken_) 函数。 如果成功，返回的 [LanguageResult](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.textanalytics.models.languageresult?view=azure-dotnet) 对象将包含 `DetectedLanguages` 中检测到的语言列表，否则包含 `errorMessage`。  输出返回的第一种语言。

> [!Tip]
> 在某些情况下，可能很难根据输入区分语言。 可以使用 `countryHint` 参数指定 2 个字母的国家/地区代码。 默认情况下，API 使用“US”作为默认的 countryHint，要删除此行为，可以通过将此值设置为空字符串 `countryHint = ""` 来重置此参数。

[!code-csharp[Language Detection example](~/cognitive-services-dotnet-sdk-samples/samples/TextAnalytics/synchronous/Program.cs?name=languageDetection)]

### <a name="output"></a>输出

```console
Language: English
```

## <a name="entity-recognition"></a>实体识别

创建名为 `RecognizeEntitiesExample()` 的新函数，用于提取前面创建的客户端，并调用该客户端的 [Entities()](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.textanalytics.textanalyticsclientextensions.entities?view=azure-dotnet#Microsoft_Azure_CognitiveServices_Language_TextAnalytics_TextAnalyticsClientExtensions_Entities_Microsoft_Azure_CognitiveServices_Language_TextAnalytics_ITextAnalyticsClient_System_String_System_String_System_Nullable_System_Boolean__System_Threading_CancellationToken_) 函数。 循环访问这些结果。 如果成功，返回的 [EntitiesResult](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.textanalytics.models.entitiesresult?view=azure-dotnet) 对象将包含 `Entities` 中检测到的实体列表，否则包含 `errorMessage`。 对于每个检测到的实体，输出其类型、子类型和维基百科名称（如果存在），以及其在原始文本中的位置。

[!code-csharp[Entity Recognition example](~/cognitive-services-dotnet-sdk-samples/samples/TextAnalytics/synchronous/Program.cs?name=entityRecognition)]


### <a name="output"></a>输出

```console
Entities:
    Name: Microsoft,        Type: Organization,     Sub-Type: N/A
        Offset: 0,      Length: 9,      Score: 1.000
    Name: Bill Gates,       Type: Person,   Sub-Type: N/A
        Offset: 25,     Length: 10,     Score: 1.000
    Name: Paul Allen,       Type: Person,   Sub-Type: N/A
        Offset: 40,     Length: 10,     Score: 0.999
    Name: April 4,  Type: Other,    Sub-Type: N/A
        Offset: 54,     Length: 7,      Score: 0.800
    Name: April 4, 1975,    Type: DateTime, Sub-Type: Date
        Offset: 54,     Length: 13,     Score: 0.800
    Name: BASIC,    Type: Other,    Sub-Type: N/A
        Offset: 89,     Length: 5,      Score: 0.800
    Name: Altair 8800,      Type: Other,    Sub-Type: N/A
        Offset: 116,    Length: 11,     Score: 0.800
```

## <a name="key-phrase-extraction"></a>关键短语提取

创建名为 `KeyPhraseExtractionExample()` 的新函数，用于提取前面创建的客户端，并调用该客户端的 [KeyPhrases()](https://docs.microsoft.com/dotnet/api/microsoft.azure.cognitiveservices.language.textanalytics.textanalyticsclientextensions.keyphrases?view=azure-dotnet#Microsoft_Azure_CognitiveServices_Language_TextAnalytics_TextAnalyticsClientExtensions_KeyPhrases_Microsoft_Azure_CognitiveServices_Language_TextAnalytics_ITextAnalyticsClient_System_String_System_String_System_Nullable_System_Boolean__System_Threading_CancellationToken_) 函数。 如果成功，结果将包含 `KeyPhrases` 中检测到的关键短语列表，如果失败，则将包含 `errorMessage`。 输出任何检测到的关键短语。

[!code-csharp[Key phrase extraction example](~/cognitive-services-dotnet-sdk-samples/samples/TextAnalytics/synchronous/Program.cs?name=keyPhraseExtraction)]


### <a name="output"></a>输出

```console
Key phrases:
    cat
    veterinarian
```
