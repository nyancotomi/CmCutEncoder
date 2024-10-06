# 字幕表示機能 / Web 低遅延視聴機能について

Web 視聴での字幕機能を有効にするため、`config.yml` で幾つかのパラメータを指定する必要があります

## HLS 配信時の字幕表示機能

iOS Safari を含むフルプラットフォーム対応しています。

[node-arib-subtitle-timedmetadater][] を利用するため、特に設定は不要です。

そのため、`config.yml` の `useSubtitleUnrecognizerCmd` オプションが廃止となりますので、ご注意ください。

なお、HLS 配信時の文字スーパー表示は未支援です。

[node-arib-subtitle-timedmetadater]: https://github.com/monyone/node-arib-subtitle-timedmetadater

### ※iOS 及び iPadOS(PWA モード時のみ)での制約

iOS 及び iPadOS(PWA モード時のみ) でフルスクリーン再生した場合に字幕が表示できません。(os のビデオプレーヤが使用されるた
め)

ただし WebVTT による字幕表示は可能なので、libaribb24 を有効化した ffmpeg による VTT 字幕は表示できます。 (表現力は劣りま
す)

### ffmpeg による VTT 字幕 720p のサンプル配置

```bash
'%FFMPEG% -re -dual_mono_mode main -fix_sub_duration -i pipe:0 -threads 0 -ignore_unknown -max_muxing_queue_size 1024 -f hls -hls_time 3 -hls_list_size 0 -hls_allow_cache 1 -hls_segment_filename %streamFileDir%/stream%streamNum%-%09d.ts -hls_flags delete_segments -c:a aac -ar 48000 -b:a 192k -ac 2 -c:v libx264 -vf yadif,scale=-2:720 -b:v 3000k -preset veryfast -flags +loop-global_header -c:s webvtt -master_pl_name stream%streamNum%.m3u8 %streamFileDir%/stream%streamNum%-child.m3u8'
```

## M2TS-LL 低遅延配信の設定および字幕表示の設定

`config.yml` にて stream/live/ts/m2tsll に FFmpeg パラメータを正しく入れる必要があります。

なお、mpegts.js による低遅延視聴機能は iPadOS に対応しますが、iOS は非対応です。

### 字幕/文字スーパーの抽出

字幕ストリームとデータストリームを抽出するように指定してください

```bash
-map 0 -c:s copy -c:d copy -ignore_unknown
```

### 低遅延最適化

FFmpeg 内部のパイプラインを低遅延化させるには、以下のように指定してください

```bash
-fflags nobuffer -flags low_delay -max_delay 250000 -max_interleave_delta 1
```

視聴遅延を 1 秒前後まで短縮することができます

なお、`-re` パラメータは不要になりますので、入らないように注意してください

### 起動速度の最適化

`-f mpegts -analyzeduration 500000` を `-i pipe:0` の前に入れて置くと視聴起動が早くなります

```bash
-f mpegts -analyzeduration 500000 -i pipe:0
```

配信の起動時間を約 3 秒まで短縮することができます

### OpenGOP の禁止

libx264 使用の場合、`-flags +cgop` をつけて closed-gop を明示的に指定するようにしてください

```bash
-c:v libx264 -flags +cgop
```

### M2TS-LL libx264 720p のサンプル配置

```bash
'%FFMPEG% -dual_mono_mode main -f mpegts -analyzeduration 500000 -i pipe:0 -map 0 -c:s copy -c:d copy -ignore_unknown -fflags nobuffer -flags low_delay -max_delay 250000 -max_interleave_delta 1 -threads 0 -c:a aac -ar 48000 -b:a 192k -ac 2 -c:v libx264 -flags +cgop -vf yadif,scale=-2:720 -b:v 3000k -preset veryfast -y -f mpegts pipe:1'
```
