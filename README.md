# Starrydata Dataset

[Starrydata2](https://starrydata.nims.go.jp/) の公式 all-data ダンプ（毎日 Google Drive に更新）を、
プロジェクト別の CSV.gz に自動分割して GitHub Releases から配信します。

公開ページ（GitHub Pages）にアクセスすると、各プロジェクトの行に
`[papers] [samples] [curves]` のダウンロードボタンが並びます。

## What you get

各プロジェクトについて 3 ファイル（gzip 圧縮された CSV）：

| ファイル | 内容 | 抽出ロジック |
|---|---|---|
| `<Project>_papers.csv.gz` | 論文メタデータ | `papers.csv` を SID で絞り込み（このプロジェクトに curves がある論文のみ） |
| `<Project>_samples.csv.gz` | 試料情報 | `samples.csv` を `(SID, sample_id)` の複合キーで絞り込み |
| `<Project>_curves.csv.gz` | 測定データ本体 | `curves.csv` の `project_names` 列を含むかで判定 |

カウントは [starrydata.github.io/links](https://starrydata.github.io/links/) と一致します。

## URLs

- 最新分: `https://github.com/atsumitanaka/starrydata-by-project/releases/latest/download/<Project>_<kind>.csv.gz`
- 日付指定: `https://github.com/atsumitanaka/starrydata-by-project/releases/download/data-YYYYMMDD/<Project>_<kind>.csv.gz`
- メタデータ: `https://github.com/atsumitanaka/starrydata-by-project/releases/latest/download/manifest.json`

## Usage

```python
import pandas as pd
url = "https://github.com/atsumitanaka/starrydata-by-project/releases/latest/download/ThermoelectricMaterials_curves.csv.gz"
df = pd.read_csv(url, compression="gzip")
```

または Excel で開く場合は `.csv.gz` を解凍してから CSV をダブルクリック。

## Architecture

```
Starrydata Google Drive  (毎日 02:00 JST 更新)
        │
        ▼  gdown
GitHub Actions  (03:00 JST 起動)
        │
        ▼  scripts/split.py
13 projects × 3 files = 39 CSV.gz  +  manifest.json
        │
        ▼  gh release create
GitHub Releases  (tag: data-YYYYMMDD + latest)
        │
        ▼  fetch
GitHub Pages SPA  (docs/index.html)
```

## Local development

```bash
python3 -m venv .venv
.venv/bin/pip install -r scripts/requirements.txt
.venv/bin/python scripts/split.py --zip /path/to/starrydata_dataset.zip --out ./out
.venv/bin/python -m http.server --directory docs 8000  # http://localhost:8000
```

## Deploy

1. このディレクトリを GitHub 上の新規リポジトリに push
2. `docs/index.html` の `REPO_OWNER` / `REPO_NAME` 定数を書き換え
3. Settings → Pages → Source: `Deploy from a branch`, Branch: `main` / `/docs`
4. Settings → Actions → Workflow permissions: `Read and write permissions`
5. 初回は Actions タブから `Daily split & release` を手動実行（`workflow_dispatch`）

## Data source

- Drive folder: <https://drive.google.com/drive/folders/1OVMP7j61CJFwLtJ-qZFef9ko40Othayh>
- File ID: `1py40fDLkTW2kcGx-ie7xHxG2Iqisfcuk` (`starrydata_dataset.zip`, ~56 MB)
- 公開・ライセンス条項は同梱の README.md（zip 内）に従ってください
