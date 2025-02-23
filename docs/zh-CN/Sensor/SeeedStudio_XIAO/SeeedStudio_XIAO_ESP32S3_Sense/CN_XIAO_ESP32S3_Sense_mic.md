---
description: This article describes how to use the microphone on the XIAO ESP32S3 Sense expansion board.
title: 麦克风使用
keywords:
- Template
image: https://files.seeedstudio.com/wiki/seeed_logo/logo_2023.png
slug: /cn/xiao_esp32s3_sense_mic
last_update:
  date: 11/09/2023
  author: Chen Lei
---

# 使用XIAO ESP32S3 上的数字麦克风

 在本教程中，我们将为您介绍如何使用XIAO ESP32S3 Sense扩展板的麦克风。首先是对I2S引脚的非常基础的使用，我们将通过I2S和麦克风得到当前环境的响度，并将其显示在串行波形图中。然后我们将介绍如何录制声音并将录制的声音保存到SD卡。

<div class="table-center">
  <table align="center">
    <tr>
        <th>Seeed Studio XIAO ESP32S3 Sense</th>
    </tr>
    <tr>
        <td><div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/SeeedStudio-XIAO-ESP32S3/img/xiaoesp32s3sense.jpg" style={{width:250, height:'auto'}}/></div></td>
    </tr>
      <tr>
        <td><div class="get_one_now_container" style={{textAlign: 'center'}}>
          <a class="get_one_now_item" href="https://www.seeedstudio.com/XIAO-ESP32S3-Sense-p-5639.html">
              <strong><span><font color={'FFFFFF'} size={"4"}> 点击购买 🖱️</font></span></strong>
          </a>
      </div></td>
    </tr>
  </table>
</div>

:::caution警告
本教程的所有内容仅适用于XIAO ESP32S3 Sense。
:::

## 开始

在开始学习本教程之前，您可能需要提前准备好以下硬件和软件。

### 安装扩展板(for Sense)

安装扩展板非常简单，只需要将扩展板上的连接器对准 XIAO ESP32S3 上的 B2B 连接器，然后用力按下，听到“咔”的一声，安装完成。

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/SeeedStudio-XIAO-ESP32S3/img/61.gif" style={{width:500, height:'auto'}}/></div>

### 准备microSD卡

在进行保存录音的项目时，可能需要MicroSD卡。

XIAO ESP32S3 Sense支持高达**32GB**的microSD卡，所以如果您准备为XIAO购买microSD卡，请参阅此规格。并在使用microSD卡之前将microSD卡格式化为**FAT32**格式。

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/SeeedStudio-XIAO-ESP32S3/img/67.png" style={{width:250, height:'auto'}}/></div>

格式化完成后，可以将microSD卡插入microSD卡插槽中。请注意插入的方向，有金手指的那一侧要朝内。

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/SeeedStudio-XIAO-ESP32S3/img/66.jpg" style={{width:500, height:'auto'}}/></div>

## 声音响度检测

在第一个项目案例中，我们使用Arduino IDE的串口波形图来检测环境中的噪声，并展示麦克风检测到的环境响度。

下面是完整的示例程序。

:::tip提示
检查并确保您使用的是哪个版本的 `esp32`，以下示例适用于 2.0.x 版本，下面的示例适用于 3.0.x 及更高版本
:::

```cpp
#include <I2S.h>

void setup() {
  // Open serial communications and wait for port to open:
  // A baud rate of 115200 is used instead of 9600 for a faster data rate
  // on non-native USB ports
  Serial.begin(115200);
  while (!Serial) {
    ; // wait for serial port to connect. Needed for native USB port only
  }

  // start I2S at 16 kHz with 16-bits per sample
  I2S.setAllPins(-1, 42, 41, -1, -1);
  if (!I2S.begin(PDM_MONO_MODE, 16000, 16)) {
    Serial.println("Failed to initialize I2S!");
    while (1); // do nothing
  }
}

void loop() {
  // read a sample
  int sample = I2S.read();

  if (sample && sample != -1 && sample != 1) {
    Serial.println(sample);
  }
}
```

:::tip提示
上面的示例仅与版本为 2.0.x 的 `esp32` 兼容，如果您使用的是最新版本（例如 3.0.x），请使用下面的示例
:::

```cpp
#include <ESP_I2S.h>
I2SClass I2S;

void setup() {
  // Open serial communications and wait for port to open:
  // A baud rate of 115200 is used instead of 9600 for a faster data rate
  // on non-native USB ports
  Serial.begin(115200);
  while (!Serial) {
    ; // wait for serial port to connect. Needed for native USB port only
  }

  // setup 42 PDM clock and 41 PDM data pins
  I2S.setPinsPdmRx(42, 41);

  // start I2S at 16 kHz with 16-bits per sample
  if (!I2S.begin(I2S_MODE_PDM_RX, 16000, I2S_DATA_BIT_WIDTH_16BIT, I2S_SLOT_MODE_MONO)) {
    Serial.println("Failed to initialize I2S!");
    while (1); // do nothing
  }
}

void loop() {
  // read a sample
  int sample = I2S.read();

  if (sample && sample != -1 && sample != 1) {
    Serial.println(sample);
  }
}
```

上传此程序为XIAO ESP32S3 Sense并打开**Serial Plotter**，您将看到声音的响度变化曲线。

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/SeeedStudio-XIAO-ESP32S3/img/83.png" style={{width:600, height:'auto'}}/></div>

### 程序注释

在程序开始时，为了使用麦克风引脚，我们首先需要导入I2S库。

```c
#include <I2S.h>
```

在I2S对象上调用`setAllPins()`函数来配置I2S接口使用的引脚。该函数接受5个整数参数，分别表示连接到I2S接口的位时钟、单词选择、数据输入、数据输出和通道选择线的GPIO引脚。

```c
I2S.setAllPins(-1, 42, 41, -1, -1);
```

在这个特定的代码中，`-1`值表示没有使用相应的引脚，而`42`和`41`值分别表示连接到单词选择和数据输入行的GPIO引脚。数据输出和通道选择行在此配置中不使用，而是设置为`-1`。

```c
if (!I2S.begin(PDM_MONO_MODE, 16000, 16)) {
    Serial.println("Failed to initialize I2S!");
    while (1); // do nothing
}
```

在I2S对象上调用`begin()`函数，用指定的参数初始化I2S接口:`PDM_MONO_MODE`、`16000` Hz采样率和`16-bit`分辨率。

:::tip提示
值得注意的是，对于目前的ESP32-S3芯片，我们只能使用`PDM_MONO_MODE`，采样位宽只能为`16bit`。只有采样率可以修改，但经过测试，16kHz的采样率比较稳定。
:::

```c
int sample = I2S.read();

if (sample && sample != -1 && sample != 1) {
    Serial.println(sample);
}
```

在I2S对象上调用`read()`函数，从I2S接口读取单个音频样本。if语句检查sample变量的值。如果采样值不是0、-1或1，它就被认为是有效的音频采样，并且If块中的代码会被执行。在这种情况下，使用`serial .println()`函数将样本值打印到串行控制台。

## 保存录制的声音到microSD卡

在下一个项目中，我们将指导您如何结合microSD卡的功能，并将录制的声音保存到microSD卡中。那么对于这个项目，请提前准备好microSD卡并格式化为**FAT32**格式。

如果这是您第一次在XIAO ESP32S3上使用microSD卡，您可以阅读[文件系统Wiki](https://wiki.seeedstudio.com/xiao_esp32s3_sense_filesystem/#prepare-the-microsd-card)内容来了解microSD卡的安装和准备。

下面是这个项目的Arduino程序。

:::tip提示

请检查并确认您使用的 ESP32 版本。以下示例适用于 2.0.x 版本，而下方的另一个示例适用于 3.0.x 及更高版本。

:::

```cpp
/* 
 * 适用于 Seeed XIAO ESP32S3 Sense 的 WAV 录音程序
*/

#include <I2S.h>
#include "FS.h"
#include "SD.h"
#include "SPI.h"

// 根据需要进行修改
#define RECORD_TIME   20  // 秒，最大值为240
#define WAV_FILE_NAME "arduino_rec"

// 不建议更改，以获得最佳效果
#define SAMPLE_RATE 16000U
#define SAMPLE_BITS 16
#define WAV_HEADER_SIZE 44
#define VOLUME_GAIN 2

void setup() {
  Serial.begin(115200);
  while (!Serial) ;
  I2S.setAllPins(-1, 42, 41, -1, -1);
  if (!I2S.begin(PDM_MONO_MODE, SAMPLE_RATE, SAMPLE_BITS)) {
    Serial.println("初始化 I2S 失败！");
    while (1) ;
  }
  if(!SD.begin(21)){
    Serial.println("挂载 SD 卡失败！");
    while (1) ;
  }
  record_wav();
}

void loop() {
  delay(1000);
  Serial.printf(".");
}

void record_wav()
{
  uint32_t sample_size = 0;
  uint32_t record_size = (SAMPLE_RATE * SAMPLE_BITS / 8) * RECORD_TIME;
  uint8_t *rec_buffer = NULL;
  Serial.printf("准备开始录音...\n");

  File file = SD.open("/"WAV_FILE_NAME".wav", FILE_WRITE);
  // 写入 WAV 文件头
  uint8_t wav_header[WAV_HEADER_SIZE];
  generate_wav_header(wav_header, record_size, SAMPLE_RATE);
  file.write(wav_header, WAV_HEADER_SIZE);

  // 使用 PSRAM 进行内存分配，用于录音缓存
  rec_buffer = (uint8_t *)ps_malloc(record_size);
  if (rec_buffer == NULL) {
    Serial.printf("内存分配失败！\n");
    while(1) ;
  }
  Serial.printf("缓冲区使用：%d 字节\n", ESP.getPsramSize() - ESP.getFreePsram());

  // 开始录音
  esp_i2s::i2s_read(esp_i2s::I2S_NUM_0, rec_buffer, record_size, &sample_size, portMAX_DELAY);
  if (sample_size == 0) {
    Serial.printf("录音失败！\n");
  } else {
    Serial.printf("录制了 %d 字节\n", sample_size);
  }

  // 增加音量
  for (uint32_t i = 0; i < sample_size; i += SAMPLE_BITS/8) {
    (*(uint16_t *)(rec_buffer+i)) <<= VOLUME_GAIN;
  }

  // 将数据写入 WAV 文件
  Serial.printf("正在写入文件...\n");
  if (file.write(rec_buffer, record_size) != record_size)
    Serial.printf("写入文件失败！\n");

  free(rec_buffer);
  file.close();
  Serial.printf("录音结束。\n");
}

void generate_wav_header(uint8_t *wav_header, uint32_t wav_size, uint32_t sample_rate)
{
  // 参考此格式说明: http://soundfile.sapp.org/doc/WaveFormat/
  uint32_t file_size = wav_size + WAV_HEADER_SIZE - 8;
  uint32_t byte_rate = SAMPLE_RATE * SAMPLE_BITS / 8;
  const uint8_t set_wav_header[] = {
    'R', 'I', 'F', 'F', // ChunkID
    file_size, file_size >> 8, file_size >> 16, file_size >> 24, // ChunkSize
    'W', 'A', 'V', 'E', // Format
    'f', 'm', 't', ' ', // Subchunk1ID
    0x10, 0x00, 0x00, 0x00, // Subchunk1Size (PCM 为 16)
    0x01, 0x00, // AudioFormat (PCM 为1)
    0x01, 0x00, // NumChannels (单声道)
    sample_rate, sample_rate >> 8, sample_rate >> 16, sample_rate >> 24, // SampleRate
    byte_rate, byte_rate >> 8, byte_rate >> 16, byte_rate >> 24, // ByteRate
    0x02, 0x00, // BlockAlign
    0x10, 0x00, // BitsPerSample (16位)
    'd', 'a', 't', 'a', // Subchunk2ID
    wav_size, wav_size >> 8, wav_size >> 16, wav_size >> 24, // Subchunk2Size
  };
  memcpy(wav_header, set_wav_header, sizeof(set_wav_header));
}
```

:::tip
上面的示例仅兼容 `esp32` 的 2.0.x 版本，如果您使用的是最新版本（例如 3.0.x），请使用下面的示例。
:::

```cpp
#include "ESP_I2S.h"
#include "FS.h"
#include "SD.h"

void setup() {
  // 创建一个 I2SClass 实例
  I2SClass i2s;

  // 创建用于存储音频数据的变量
  uint8_t *wav_buffer;
  size_t wav_size;

  // 初始化串口
  Serial.begin(115200);
  while (!Serial) {
    delay(10);
  }

  Serial.println("正在初始化 I2S 总线...");

  // 设置用于音频输入的引脚
  i2s.setPinsPdmRx(42, 41);

  // 以 16 kHz 频率及 16 位深度单声道启动 I2S
  if (!i2s.begin(I2S_MODE_PDM_RX, 16000, I2S_DATA_BIT_WIDTH_16BIT, I2S_SLOT_MODE_MONO)) {
    Serial.println("初始化 I2S 失败！");
    while (1); // 不执行任何操作
  }

  Serial.println("I2S 总线已初始化。");
  Serial.println("正在初始化 SD 卡...");

  // 设置用于访问 SD 卡的引脚
  if(!SD.begin(21)){
    Serial.println("挂载 SD 卡失败！");
    while (1) ;
  }
  Serial.println("SD 卡已初始化。");
  Serial.println("正在录制 20 秒的音频数据...");

  // 录制 20 秒的音频数据
  wav_buffer = i2s.recordWAV(20, &wav_size);

  // 在 SD 卡上创建一个文件
  File file = SD.open("/arduinor_rec.wav", FILE_WRITE);
  if (!file) {
    Serial.println("打开文件以写入失败！");
    return;
  }

  Serial.println("正在将音频数据写入文件...");

  // 将音频数据写入文件
  if (file.write(wav_buffer, wav_size) != wav_size) {
    Serial.println("将音频数据写入文件失败！");
    return;
  }

  // 关闭文件
  file.close();

  Serial.println("程序执行完成。");
}

void loop() {
  delay(1000);
  Serial.printf(".");
}
```

要执行此示例，我们需要使用 ESP-32 芯片的 PSRAM 功能，请在上传前开启。

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/SeeedStudio-XIAO-ESP32S3/img/94.png" style={{width:700, height:'auto'}}/></div>

本程序在用户 **打开串口监视器后** 仅执行一次，录制 20 秒的音频，并将录制文件以 "arduino_rec.wav" 保存到 microSD 卡中。

当在串口监视器中每隔 1 秒输出 "." 时，表示程序执行已完成，此时您可以使用读卡器读取该录制文件并进行播放。

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/SeeedStudio-XIAO-ESP32S3/img/102.png" style={{width:700, height:'auto'}}/></div>

:::tip提示
若要在 XIAO ESP32S3 上播放录制的音频，您可能需要使用支持 WAV 格式的音频播放器。
:::

### 程序注释

在这个程序中，我们为记录函数写了两个函数，一个是`record_wav()`，另一个是`generate_wav_header()`。

- `record_wav()`:记录函数。在这个函数中，程序通过I2S接口从麦克风读取音频数据，并以WAV音频文件的形式存储到SD卡中。

  a.初始化变量。程序定义了一个缓冲区`rec_buffer`来存储记录的数据，并设置了记录时间`RECORD_TIME`。

  b.打开WAV文件。该程序使用`SD.open()`函数打开WAV音频文件，并将其文件名定义为`WAV_FILE_NAME`。

  c.写入WAV文件头。该程序使用`generate_wav_header()`函数生成WAV音频文件的头信息，并将其写入打开的WAV文件中。

  d.分配内存并开始记录。该程序使用`ps_malloc()`函数在ESP32S3的PSRAM中分配一块内存用于存储记录的数据，并使用`esp_i2s::i2s_read()`函数从麦克风读取音频数据。读取的数据保存在rec_buffer缓冲区中。

  e.增加音量。该程序使用由`VOLUME_GAIN`常量定义的增益值来增加记录数据的体积。

  f.将记录的数据写入WAV文件。程序使用`file.write()`函数将记录的数据写入打开的WAV文件。

  g.释放缓冲内存并关闭WAV文件。这个程序使用`free()`函数来释放`rec_buffer`缓冲区的内存，并使用`file.close()`函数来关闭打开的WAV文件。

- `generate_wav_header(uint8_t *wav_header, uint32_t wav_size, uint32_t sample_rate)`:用于生成WAV文件的头信息的函数。在这个函数中，程序根据WAV文件的规范生成一个包含WAV文件头信息的字节数组。

  a.定义WAV文件头信息的常量。程序定义了一个包含WAV文件头信息的字节数组`set_wav_header`，并定义了WAV文件规范的常量，包括`NUM_CHANNELS`、`BITS_PER_SAMPLE`、`WAV_HEADER_SIZE`和`SUB_CHUNK_SIZE`。

  b.配置WAV文件头信息。该程序使用步骤a中定义的常量设置WAV文件头信息，并根据WAV文件的规范计算一些字段的值，包括' AUDIO_FORMAT '、' BYTE_RATE '、' BLOCK_ALIGN '、' SAMPLES_PER_CHANNEL '和' CHUNK_SIZE '。计算结果存储在`set_wav_header`字节数组中。

  c.复制WAV文件头信息。程序将存储在`set_wav_header`中的首部信息复制到字节数组`wav_header`中。
  
## 发现并修理故障

### 为什么我不能播放录制的音频文件?

如果您遇到无法玩的情况，请检查串口监视器打印的调试信息，查看有关读写卡的错误信息。如果有，请更换microSD卡或检查microSD卡与扩展板连接是否松动或不稳定。如果卡没有问题，那么音频文件应该大小，如果录音有问题，它可能显示为录制的音频文件大小只有0KB。

例如，在下面的图中，卡片读写存在问题。

<div style={{textAlign:'center'}}><img src="https://files.seeedstudio.com/wiki/SeeedStudio-XIAO-ESP32S3/img/103.png" style={{width:600, height:'auto'}}/></div>

如果卡没有问题并且录音很成功的话。然后需要检查软件是否支持WAV格式的音频播放。建议使用专用音乐播放软件播放音频，尽量不要使用视频播放器播放。经过实际测试，有很多视频播放器(虽然支持WAV格式)都不能播放。

## 技术支持和产品讨论

感谢您选择我们的产品!我们在这里为您提供不同的支持，以确保您使用我们的产品的体验尽可能顺利。我们提供多种沟通渠道，以满足不同的喜好和需求。

<div class="button_tech_support_container">
<a href="https://forum.seeedstudio.com/" class="button_forum"></a>
<a href="https://www.seeedstudio.com/contacts" class="button_email"></a>
</div>

<div class="button_tech_support_container">
<a href="https://discord.gg/eWkprNDMU7" class="button_discord"></a>
<a href="https://github.com/Seeed-Studio/wiki-documents/discussions/69" class="button_discussion"></a>
</div>
