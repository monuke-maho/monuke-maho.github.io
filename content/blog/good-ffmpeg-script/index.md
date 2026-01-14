---
date: '2026-01-14T16:20:56+09:00'
draft: false
title: 'いい感じのffmpeg変換用bat'
slug: 'good-ffmpeg-script'
categories: ["Note"]
tags: ["コマンドライン","FFmpeg"]
---

## 用途

ffmpegは非常に強力なツールです。けど毎回コマンドを打つのは面倒なので、batスクリプトを書きました。

Windows向けです。

## 内容

```batch
@echo off
setlocal

@REM Ask for input folder via PowerShell
set "psCommand=Add-Type -AssemblyName System.Windows.Forms; $f = New-Object System.Windows.Forms.FolderBrowserDialog; $f.Description = 'Select Input Folder'; $f.ShowNewFolderButton = $false; if ($f.ShowDialog() -eq 'OK') { return $f.SelectedPath }"

for /f "usebackq delims=" %%I in (`powershell -NoProfile -Command "%psCommand%"`) do set "input_folder=%%I"

if "%input_folder%"=="" (
    echo No folder selected.
    pause
    exit /b
)

echo Processing folder: "%input_folder%"
pushd "%input_folder%"

if not exist "conv" (
    mkdir "conv"
)

for %%f in (*.mkv,*.mp4) do (
    ffmpeg -i "%%f" -map 0:v -map 0:a -c:v hevc_nvenc -rc vbr -cq 26 -b:v 0 -tune hq -preset p7 -multipass fullres -pix_fmt p010le -profile:v main10 -c:a copy -f mp4 "conv\%%~nf.mp4" -y
)

popd
pause
```

## 使い方

1. 内容をコピペして適当なbatファイルを作成する(例:`conv.bat`)
2. ダブルクリックで起動
3. 変換したいフォルダを選択する
4. 変換が終わるまで待つ
5. 変換後は`conv`フォルダに変換されたファイルが保存される

って感じです。

## 終わり

ffmpegのオプションについては、アニメ用で書いている。  
そのため実写とかその他の用途で使う場合は適宜調整が必要になるかもしれません。

あと、NVIDIAのGPUじゃないとエラーが出ます。(`nvenc`を使用しているため)