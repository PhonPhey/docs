# API method description

Generates speech for the text submitted.

## HTTP request {#http_request}

```
POST https://tts.api.cloud.yandex.net/speech/v1/tts:synthesize
```

## Parameters in the request body {#body_params}

All parameters must be URL-encoded. The maximum size of the POST request body is 30 KB.

| Parameter | Description |
| ----- | ----- |
| `text` | Required parameter.<br/>UTF-8 encoded text to be converted into speech.<br/>For homographs, use `+` before the stressed vowel. For example, `contr+ol` or `def+ect`.<br/>Maximum string length: 5000 characters. |
| `lang` | Language.<br/>Acceptable values:<ul><li>`ru-RU` (default) — Russian.</li><li>`en-US` — English.</li><li>`tr-TR` — Turkish.</li></ul> |
| `voice` | The voice for the synthesized speech.<br/>You can choose one of the following voices:<ul><li>Female voice: `alyss`, `jane`, `oksana` and `omazh`.</li><li>Male voice: `zahar` and `ermil`.</li></ul>Default value of the parameter: `oksana`. |
| `emotion` | Emotional tone of the voice.<br/>Acceptable values:<ul><li>`good` —  Cheerful and friendly.</li><li>`evil` — Irritated.</li><li>`neutral` (default) — Without emotion.</li></ul> |
| `speed` | The rate (tempo) of the synthesized speech.<br/>The speech rate is set as a decimal number in the range from `0.1` to `3.0`. Where:<ul><li>`3.0` — Fastest rate.</li><li>`1.0` (default) — Average human speech rate.</li><li>`0.1` — Slowest speech rate.</li></ul> |
| `format` | The format of the synthesized audio.<br/>Acceptable values:<ul><li>`lpcm` — Audio file is synthesized in the [LPCM](https://en.wikipedia.org/wiki/Pulse-code_modulation) format with no WAV header. Audio characteristics:<ul><li>Sampling — 8, 16, or 48 kHz, depending on the `sampleRateHertz` parameter value.</li><li>Bit depth — 16-bit.</li><li>Byte order — Reversed (little-endian).</li><li>Audio data is stored as signed integers.</li></ul></li><li>`oggopus` (default) — Data in the audio file is encoded using the OPUS audio codec and compressed using the OGG container format ([OggOpus](https://wiki.xiph.org/OggOpus)).</li></ul> |
| `sampleRateHertz` | The sampling frequency of the synthesized audio.<br/>Used if `format` is set to `lpcm`. Acceptable values:<ul><li>`48000` (default) — Sampling rate of 48 kHz.</li><li>`16000` — Sampling rate of 16 kHz.</li><li>`8000` — Sampling rate of 8 kHz.</li></ul> |
| `folderId` | Required parameter.<br/>ID of your folder.<br/>For more information about how to find the folder ID, see the section [Authorization in the API](../concepts/auth.md). |

## Response {#response}

If speech synthesis is successful, the response contains the binary content of the audio file. The output data format depends on the value of the `format` parameter.

## Examples {#examples}

### Conversion of text to speech in Ogg format {#ogg}

In this example, the text "Hello world" is synthesized and recorded as an audio file.

By default, data in the audio file is encoded using the OPUS audio codec and compressed using the OGG container format ([OggOpus](https://wiki.xiph.org/OggOpus)).

---

**[!TAB cURL]**

```bash
export FOLDER_ID=b1gvm4b03aoopfsct641
export IAM_TOKEN=CggaATEVAgA...
curl -X POST \
     -H "Authorization: Bearer ${IAM_TOKEN}" \
     --data-urlencode "text=Hello world" \
     -d "voice=zahar&emotion=good&folderId=${FOLDER_ID}" \
     "https://tts.api.cloud.yandex.net/speech/v1/tts:synthesize" > speech.ogg
```

**[!TAB C#]**

```c#
using System;
using System.Collections.Generic;
using System.Net.Http;
using System.Threading.Tasks;
using System.IO;

namespace TTS
{
  class Program
  {
    static void Main()
    {
      Tts().GetAwaiter().GetResult();
    }

    static async Task Tts()
    {
      const string iamToken = "CggaATEVAgA..."; // Specify the IAM token.
      const string folderId = "b1gvm4b03aoopfsct641"; // Specify the folder ID.

      HttpClient client = new HttpClient();
      client.DefaultRequestHeaders.Add("Authorization", "Bearer " + iamToken);
      var values = new Dictionary<string, string>
      {
        { "voice", "zahar" },
        { "emotion", "good" },
        { "folderId", folderId },
        { "lang", "en-US" },
        { "text": "Hello world" },
        { 'format': 'lpcm' },
        { 'sampleRateHertz': 48000 }
      };
      var content = new FormUrlEncodedContent(values);
      var response = await client.PostAsync("https://tts.api.cloud.yandex.net/speech/v1/tts:synthesize", content);
      var responseBytes = await response.Content.ReadAsByteArrayAsync();
      File.WriteAllBytes("speech.pcm", responseBytes);
    }
  }
}
```

**[!TAB Python]**

```python
import argparse

import requests


def synthesize(folder_id, iam_token, text):
    url = 'https://tts.api.cloud.yandex.net/speech/v1/tts:synthesize'
    headers = {
        'Authorization': 'Bearer ' + iam_token,
    }

    data = {
        'text': text,
        'voice': 'alyss',
        'emotion': 'good',
        'folderId': folder_id,
        'lang': 'en-US',
        'format': 'lpcm',
        'sampleRateHertz': 48000,
    }

    resp = requests.post(url, headers=headers, data=data)
    if resp.status_code != 200:
        raise RuntimeError("Invalid response received: code: %d, message: %s" % (resp.status_code, resp.text))

    return resp.content


if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument("--iam_token", required=True, help="IAM token")
    parser.add_argument("--folder_id", required=True, help="Folder id")
    parser.add_argument("--text", required=True, help="Text for synthesize")
    parser.add_argument("--output", required=True, help="Output file name")
    args = parser.parse_args()

    audio_content = synthesize(args.folder_id, args.iam_token, args.text)
    with open(args.output, "wb") as f:
        f.write(audio_content)
```

---

### Conversion of text to speech in WAV format {#wav}

In this example, the submitted text is synthesized in the LPCM format with a sampling rate of 48kHz and saved to a `speech.raw` file. This file is then converted to WAV format using the [SoX](http://sox.sourceforge.net/) utility.

---

**[!TAB cURL]**

```bash
export FOLDER_ID=b1gvm4b03aoopfsct641
export IAM_TOKEN=CggaATEVAgA...
curl -X POST \
    -H "Authorization: Bearer ${IAM_TOKEN}" \
    -o speech.raw \
    --data-urlencode "text=Hello world" \
    -d "voice=zahar&emotion=good&folderId=${FOLDER_ID}&format=lpcm&sampleRateHertz=48000&lang=en-US" \
    https://tts.api.cloud.yandex.net/speech/v1/tts:synthesize

sox -r 48000 -b 16 -e signed-integer -c 1 speech.raw speech.wav
```

---

