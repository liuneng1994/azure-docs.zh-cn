---
title: 关于语音设备 SDK-语音服务
titleSuffix: Azure Cognitive Services
description: 语音设备 SDK 入门。 语音服务适用于各种设备和音频源。 语音设备 SDK 是与特制的麦克风阵列开发工具包配对的一个预优化库。
services: cognitive-services
author: erhopf
manager: nitinme
ms.service: cognitive-services
ms.subservice: speech-service
ms.topic: conceptual
ms.date: 07/05/2019
ms.author: erhopf
ms.openlocfilehash: 7f6793cab3d2d7cc54252bae3a7393f836256bdd
ms.sourcegitcommit: 6c01e4f82e19f9e423c3aaeaf801a29a517e97a0
ms.translationtype: MT
ms.contentlocale: zh-CN
ms.lasthandoff: 12/04/2019
ms.locfileid: "74815529"
---
# <a name="about-the-speech-devices-sdk"></a>关于语音设备 SDK

[语音服务](overview.md)适用于各种设备和音频源。 现在，可以通过匹配的硬件和软件进一步利用语音应用程序。 语音设备 SDK 是与特制的麦克风阵列开发工具包配对的一个预优化库。

语音设备 SDK 有助于：

- 快速测试新的语音方案。
- 更轻松地将基于云的语音服务集成到设备中。
- 为客户创建出色的用户体验。

语音设备 SDK 使用[语音 SDK](speech-sdk.md)。 它使用语音 SDK 将我们的高级音频处理算法处理的音频从设备的麦克风阵列发送到[语音服务](overview.md)。 它使用多声道音频，通过噪声抑制、回音消除、波束赋形和残响消除来提供更准确的远场[语音识别](speech-to-text.md)。

你还可以使用语音设备 SDK 来构建具有你自己的[自定义关键字](speech-devices-sdk-create-kws.md)的环境设备，以便启动用户交互的提示对你的品牌是唯一的。

语音设备 SDK 有助于实现各种语音支持方案，如[语音助手](https://aka.ms/bots/speech/va)、驱动器到订购系统、[对话](conversation-transcription-service.md)脚本和智能扬声器。 可以使用文本回复用户、用默认或[自定义语音](how-to-customize-voice-font.md)回答用户、提供搜索结果以及[翻译](speech-translation.md)为其他语言等。 我们期待看到你的成果！

## <a name="get-the-speech-devices-sdk"></a>获取语音设备 SDK

### <a name="android"></a>Android

对于 Android 设备，请下载最新版本的[Android 语音设备 SDK](https://aka.ms/sdsdk-download-android)。

### <a name="windows"></a>Windows

对于 Windows，示例应用程序是以跨平台的 Java 应用程序的形式提供的。 下载最新版本的[JRE 语音设备 SDK](https://aka.ms/sdsdk-download-JRE)。
该应用程序是使用语音 SDK 程序包和 Eclipse Java IDE (v4) 在 64 位 Windows 上构建的。 它在 64 位 Java 8 运行时环境 (JRE) 中运行。

### <a name="linux"></a>Linux

对于 Linux，示例应用程序是以跨平台的 Java 应用程序的形式提供的。 下载最新版本的[JRE 语音设备 SDK](https://aka.ms/sdsdk-download-JRE)。
该应用程序是使用语音 SDK 程序包和 Eclipse Java IDE (v4) 在 64 位 Linux（Ubuntu 16.04、Ubuntu 18.04、Debian 9）上构建的。 它在 64 位 Java 8 运行时环境 (JRE) 中运行。

## <a name="next-steps"></a>后续步骤

> [!div class="nextstepaction"]
> [选择语音设备](get-speech-devices-sdk.md)
> [!div class="nextstepaction"]
> [免费获取语音服务订阅密钥](get-started.md)
