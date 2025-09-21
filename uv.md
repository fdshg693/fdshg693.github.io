---
layout: default
title: ページのタイトル
---

# uv（Pythonパッケージマネージャ）チートシート

最速のPythonパッケージマネージャ「uv」（by Astral）のよく使うコマンドまとめ。

## インストール / 初期設定（Linux）

```bash
# uv のインストール
curl -LsSf https://astral.sh/uv/install.sh | sh

# PATH（インストーラ既定は ~/.local/bin 配下）
export PATH="$HOME/.local/bin:$PATH"

# 動作確認
uv --version
```

## プロジェクト作成～基本フロー

```bash
# 新規プロジェクトの初期化（pyproject.toml を作成）
uv init myapp
cd myapp

# 使用する Python を固定（例: 3.12）
uv python pin 3.12

# 仮想環境の作成（.venv）
uv venv

# 依存関係の追加（本番 / 開発）
uv add requests
uv add --dev pytest ruff black

# 依存の解決・インストール（ロックに基づく）
uv sync

# テスト実行（仮想環境を有効化せずに実行可能）
uv run -m pytest -q
```

## 依存関係の管理

```bash
# 追加 / 削除
uv add fastapi[standard]
uv add --dev mypy
uv remove mypy

# ロックの更新（特定パッケージのみ上げる例）
uv lock --upgrade fastapi

# ロックどおりにインストール（CI 用）
uv sync --frozen

# 開発依存を除いてインストール
uv sync --no-dev
```

## 実行（venv をアクティブにせずに）

```bash
uv run python -V
uv run -m http.server 8000
uv run scripts/do_something.py  # 任意のスクリプト
```

## 仮想環境 / Python 本体

```bash
# 仮想環境の作成 / 任意バージョンの Python 指定
uv venv
uv venv --python 3.11

# 手動で有効化したい場合
source .venv/bin/activate

# Python のインストール・一覧・固定
uv python install 3.12
uv python list
uv python pin 3.12
```

## pip 互換コマンド（requirements.txt 連携）

```bash
# requirements.txt をインストール
uv pip install -r requirements.txt

# 現状を書き出し
uv pip freeze > requirements.txt

# 一覧・アンインストール
uv pip list
uv pip uninstall -y package_name
```

## 依存の可視化 / エクスポート

```bash
# 依存ツリーの表示
uv tree            # 全体
uv tree fastapi    # 特定パッケージ

# ロックから requirements.txt を書き出し
uv export -o requirements.txt
```

## ビルド / 公開

```bash
# sdist / wheel のビルド（dist/ に生成）
uv build

# PyPI へ公開（事前にトークン等を設定）
uv publish --username __token__ --password <pypi-token>
```

## キャッシュ

```bash
uv cache dir   # キャッシュディレクトリの場所
uv cache clean # キャッシュ削除
```

## ツールの一時実行（uvx：pipx 相当）

```bash
# Linter / Formatter / テスト等を即実行
uvx ruff check .
uvx ruff format .
uvx black --check .
uvx pytest -q
```

## よくある組み合わせ（コピペ用）

```bash
# プロジェクト新規 + 依存追加 + テスト
uv init demo && cd demo \
	&& uv python pin 3.12 \
	&& uv venv \
	&& uv add requests pytest \
	&& uv sync \
	&& uv run -m pytest -q

# 既存プロジェクトでロック固定の再現（CI）
uv sync --frozen && uv run -m pytest -q
```

備考:
- 大半のコマンドは `.venv` を自動検出します。`uv run` を使うと venv の有効化が不要です。
- `pyproject.toml` と `uv.lock` が依存のソース・ロックになります。CI では `uv sync --frozen` を推奨。
