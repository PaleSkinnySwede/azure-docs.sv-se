---
title: H264, enkel bit hastighet, låg kvalitet, SD för Android | Microsoft Docs
description: Avsnittet ger en översikt över **H264, Single bitrs låg kvalitet SD för Android Task för** inställning.
author: Juliako
manager: femila
editor: ''
services: media-services
documentationcenter: ''
ms.assetid: 67d3446d-e3b8-419f-bbf8-e5149c108531
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: article
ms.date: 03/19/2019
ms.author: juliako
ms.openlocfilehash: 6844d920b8726dcfee38234d539a5314afd2e40b
ms.sourcegitcommit: 849bb1729b89d075eed579aa36395bf4d29f3bd9
ms.translationtype: MT
ms.contentlocale: sv-SE
ms.lasthandoff: 04/27/2020
ms.locfileid: "61129635"
---
# <a name="h264-single-bitrate-low-quality-sd-for-android"></a>H264, enkel bithastighet, låg kvalitet, SD för Android
`Media Encoder Standard`definierar en uppsättning kodnings för inställningar som du kan använda när du skapar kodnings jobb. Du kan antingen använda en `preset name` för att ange i vilket format du vill koda medie filen. Du kan också skapa egna JSON-eller XML-baserade för hands inställningar (med UTF-8-eller UTF-16-kodning. Sedan skickar du den anpassade för inställningen till kodaren. En lista över alla förinställda namn som stöds av den här `Media Encoder Standard` kodaren finns i [uppgifts för inställningar för Media Encoder Standard](media-services-mes-presets-overview.md).  
  
 I det här avsnittet `H264 Single Bitrate Low Quality SD for Android` visas för inställningen i XML-och JSON-format.  
  
 Den här för inställningen skapar en enda MP4-fil med en bit hastighet på 56 kbit/s och stereo AAC-ljud. Detaljerad information om profil, bit hastighet, samplings frekvens osv. för den här för inställningen finns i XML-eller JSON-definitionen nedan. Förklaringar av vad varje element i de här för inställningarna innebär och giltiga värden för varje element finns i avsnittet [Media Encoder Standard schema](media-services-mes-schema.md) .  
  
 XML  
  
```  
<?xml version="1.0" encoding="utf-16"?>  
<Preset xmlns:xsd="https://www.w3.org/2001/XMLSchema" xmlns:xsi="https://www.w3.org/2001/XMLSchema-instance" Version="1.0" xmlns="https://www.windowsazure.com/media/encoding/Preset/2014/03">  
  <Encoding>  
    <H264Video>  
      <KeyFrameInterval>00:00:05</KeyFrameInterval>  
      <SceneChangeDetection>true</SceneChangeDetection>  
      <H264Layers>  
        <H264Layer>  
          <Bitrate>56</Bitrate>  
          <Width>176</Width>  
          <Height>144</Height>  
          <FrameRate>12/1</FrameRate>  
          <Profile>Baseline</Profile>  
          <Level>2</Level>  
          <BFrames>0</BFrames>  
          <ReferenceFrames>3</ReferenceFrames>  
          <Slices>0</Slices>  
          <AdaptiveBFrame>false</AdaptiveBFrame>  
          <EntropyMode>Cavlc</EntropyMode>  
          <BufferWindow>00:00:05</BufferWindow>  
          <MaxBitrate>56</MaxBitrate>  
        </H264Layer>  
      </H264Layers>  
      <Chapters />  
    </H264Video>  
    <AACAudio>  
      <Profile>HEAACV2</Profile>  
      <Channels>2</Channels>  
      <SamplingRate>48000</SamplingRate>  
      <Bitrate>24</Bitrate>  
    </AACAudio>  
  </Encoding>  
  <Outputs>  
    <Output FileName="{Basename}_{Width}x{Height}_{VideoBitrate}.mp4">  
      <MP4Format />  
    </Output>  
  </Outputs>  
</Preset>  
```  
  
 JSON  
  
```  
{  
  "Version": 1.0,  
  "Codecs": [  
    {  
      "KeyFrameInterval": "00:00:05",  
      "SceneChangeDetection": true,  
      "H264Layers": [  
        {  
          "Profile": "Baseline",  
          "Level": "2",  
          "Bitrate": 56,  
          "MaxBitrate": 56,  
          "BufferWindow": "00:00:05",  
          "Width": 176,  
          "Height": 144,  
          "ReferenceFrames": 3,  
          "EntropyMode": "Cavlc",  
          "Type": "H264Layer",  
          "FrameRate": "12/1"  
        }  
      ],  
      "Type": "H264Video"  
    },  
    {  
      "Profile": "HEAACV2",  
      "Channels": 2,  
      "SamplingRate": 48000,  
      "Bitrate": 24,  
      "Type": "AACAudio"  
    }  
  ],  
  "Outputs": [  
    {  
      "FileName": "{Basename}_{Width}x{Height}_{VideoBitrate}.mp4",  
      "Format": {  
        "Type": "MP4Format"  
      }  
    }  
  ]  
}  
```
