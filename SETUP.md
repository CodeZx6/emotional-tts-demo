# 评测网页部署指南

## 1. 创建 Google Sheets 后端（5分钟）

### Step 1: 创建 Google Sheet
1. 打开 https://sheets.google.com 创建新表格
2. 第一行写表头：`evaluator_id | timestamp | system | emotion | sample_id | nmos | emos | smos`

### Step 2: 创建 Apps Script
1. 在 Sheet 中点击 Extensions → Apps Script
2. 删除默认代码，粘贴以下内容：

```javascript
function doPost(e) {
  var sheet = SpreadsheetApp.getActiveSpreadsheet().getActiveSheet();
  var data = JSON.parse(e.postData.contents);

  // Append each trial as a row
  data.trials.forEach(function(trial) {
    sheet.appendRow([
      data.evaluator_id,
      data.timestamp,
      trial.system,
      trial.emotion,
      trial.sample_id,
      trial.nmos,
      trial.emos,
      trial.smos
    ]);
  });

  return ContentService
    .createTextOutput(JSON.stringify({status: "ok", count: data.trials.length}))
    .setMimeType(ContentService.MimeType.JSON);
}
```

3. 点击 Deploy → New deployment
4. Type 选 "Web app"
5. Execute as: **Me**
6. Who has access: **Anyone**
7. 点 Deploy，复制生成的 URL

### Step 3: 配置 index.html
将复制的 URL 填入 `index.html` 中的 `GOOGLE_SHEET_URL` 变量。

## 2. 准备音频文件

目录结构：
```
eval/
├── index.html
├── audio/
│   ├── gt/
│   │   ├── angry/001.wav, 002.wav, ...
│   │   ├── happy/001.wav, 002.wav, ...
│   │   └── sad/001.wav, 002.wav, ...
│   ├── ours/
│   │   ├── angry/001.wav, ...
│   │   ├── happy/001.wav, ...
│   │   └── sad/001.wav, ...
│   ├── qwen3tts/  (same structure)
│   ├── cosyvoice2/
│   ├── emoknob/
│   ├── baseline/
│   └── reference/
│       ├── angry/001.wav, ...  (neutral ref audio for sMOS)
│       ├── happy/001.wav, ...
│       └── sad/001.wav, ...
```

## 3. 部署到 GitHub Pages

```bash
cd eval
git init
git add .
git commit -m "listening test"
git remote add origin git@github.com:YOUR_USERNAME/emotion-eval.git
git push -u origin main
```

在 GitHub repo Settings → Pages → Source: main branch → Save

访问: https://YOUR_USERNAME.github.io/emotion-eval/

## 4. 音频文件大小估算

- 6 systems × 3 emotions × 10 samples = 180 files
- + 30 reference files = 210 files
- 每个 ~5s, 16kHz mono WAV ≈ 160KB
- 总计 ≈ 34MB (GitHub Pages limit: 1GB)
