# GitHub Actions CI - Spring Boot + React テンプレート

## 概要
Spring Boot（Gradle）と React（Vite + Vitest）の両方のテストを CI で自動実行する GitHub Actions ワークフロー。バックエンドとフロントエンドを並列ジョブで実行する。

## 使用場面
- Spring Boot + React のフルスタックプロジェクトに CI を導入する時
- PR のたびにバックエンド・フロントエンドの両テストを自動実行したい時
- Gradle のキャッシュで CI を高速化したい時

## コード

```yaml
name: CI

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  # バックエンド（Spring Boot + Gradle）
  backend-test:
    name: Backend Tests
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: backend  # バックエンドディレクトリをデフォルトに
    steps:
      - uses: actions/checkout@v4

      - name: Set up Java 17
        uses: actions/setup-java@v4
        with:
          distribution: "temurin"
          java-version: "17"

      - name: Cache Gradle packages
        uses: actions/cache@v4
        with:
          path: |
            ~/.gradle/caches
            ~/.gradle/wrapper
          key: ${{ runner.os }}-gradle-${{ hashFiles('backend/**/*.gradle', 'backend/**/gradle-wrapper.properties') }}
          restore-keys: |
            ${{ runner.os }}-gradle-

      - name: Grant execute permissions to gradlew
        run: chmod +x gradlew

      - name: Run tests
        run: ./gradlew test

  # フロントエンド（React + Vite + Vitest）
  frontend-test:
    name: Frontend Tests
    runs-on: ubuntu-latest
    defaults:
      run:
        working-directory: frontend  # フロントエンドディレクトリをデフォルトに
    steps:
      - uses: actions/checkout@v4

      - name: Set up Node.js 20
        uses: actions/setup-node@v4
        with:
          node-version: "20"
          cache: "npm"
          cache-dependency-path: frontend/package-lock.json

      - name: Install dependencies
        run: npm ci  # package-lock.json から正確に再現（npm install より厳密）

      - name: Run tests
        run: npx vitest run  # 一度だけ実行（ウォッチモードなし）
```

## ディレクトリ構成

```
project-root/
├── .github/
│   └── workflows/
│       └── ci.yml          ← このファイル
├── backend/                ← Spring Boot プロジェクト
│   ├── gradlew
│   ├── build.gradle
│   └── src/
└── frontend/               ← Vite + React プロジェクト
    ├── package.json
    ├── package-lock.json
    └── src/
```

## 説明

### ジョブの並列実行

`backend-test` と `frontend-test` は依存関係がないため、並列で実行される。

```
push/PR
  ├─ backend-test  ← 並列実行
  └─ frontend-test ← 並列実行
```

sequential（直列）にする場合は `needs` を使う。

```yaml
frontend-test:
  needs: backend-test  # backend-test が成功してから実行
```

### defaults.run.working-directory

各ステップの `run` コマンドのデフォルト実行ディレクトリを設定する。個々のステップで `working-directory` を指定する必要がなくなる。

```yaml
defaults:
  run:
    working-directory: backend

steps:
  - run: ./gradlew test  # backend/ で実行される
```

### Gradle キャッシュ

```yaml
- name: Cache Gradle packages
  uses: actions/cache@v4
  with:
    path: |
      ~/.gradle/caches
      ~/.gradle/wrapper
    key: ${{ runner.os }}-gradle-${{ hashFiles('backend/**/*.gradle', 'backend/**/gradle-wrapper.properties') }}
    restore-keys: |
      ${{ runner.os }}-gradle-
```

- `key`: `*.gradle` ファイルのハッシュをキーにする。ファイルが変わるとキャッシュミス → 再取得
- `restore-keys`: 完全一致しない場合の部分一致フォールバック（前回のキャッシュを再利用）

### npm ci vs npm install

```yaml
- run: npm ci   # ✅ CI 向け：package-lock.json から完全再現（速くて確実）
# - run: npm install  # ❌ package-lock.json を更新する可能性がある
```

### Vitest の CI 実行

```bash
npx vitest run    # 一度だけ実行して終了（CI 向け）
npx vitest        # ウォッチモード（開発向け、CI には使わない）
```

### トリガーの設定

```yaml
on:
  push:
    branches: [main]         # main へのプッシュ時
  pull_request:
    branches: [main]         # main への PR 作成・更新時
```

PR ベースの開発では `pull_request` トリガーが最重要。マージ前に全テストを通過させる運用にできる。

## 発展

### ビルドジョブの追加

```yaml
backend-build:
  needs: backend-test
  steps:
    - run: ./gradlew build -x test  # テストはスキップ（テスト済みのため）
```

### テスト結果のレポート表示

```yaml
- name: Publish Test Results
  uses: EnricoMi/publish-unit-test-result-action@v2
  if: always()  # テスト失敗でも実行
  with:
    files: backend/build/test-results/**/*.xml
```

## 参考
- [GitHub Actions - 公式ドキュメント](https://docs.github.com/ja/actions)
- [actions/setup-java](https://github.com/actions/setup-java)
- [actions/setup-node](https://github.com/actions/setup-node)
- [actions/cache](https://github.com/actions/cache)

## 関連スニペット
- [Vitest 設定](../../react/config/vite-test-config.md)
- [@WebMvcTest + MockMvc テスト](../../spring/test/controller-test.md)

## 作成日
2026-03-21

## タグ
#github-actions #ci #spring-boot #react #gradle #vitest #devops #automation
