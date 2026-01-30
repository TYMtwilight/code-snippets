# Spring Boot 組み込みサーバー（Embedded Server）

> **学習日:** 2026-01-30
> **情報源:** [Spring Boot公式](https://spring.io/projects/spring-boot)

---

## 何のための概念か？

WebアプリケーションをJARファイル1つで実行可能にするため、Tomcat/Jetty/Undertowなどのサーバーソフトウェアをアプリケーション内に組み込む仕組み。

---

## 要点

### この概念を一言で

アプリケーションサーバー（Tomcat等）を別途インストール・設定せずに、`java -jar`だけでWebアプリを起動できる仕組み。

### 重要ポイント
1. 「サーバー」には2種類ある：物理サーバー（マシン）とアプリケーションサーバー（ソフトウェア）
2. 不要になるのはアプリケーションサーバー（Tomcat等）のインストール・設定作業
3. 物理的なマシンとJava実行環境（JDK）は引き続き必要

---

## 注意点

### よくある誤解
- ❌ 「サーバーが不要になる」＝物理的なマシンも不要
  - ✅ 物理マシンは必要。不要になるのはTomcat等のソフトウェアのインストール・設定作業

### 気をつけるべきこと
- WARデプロイが必要な環境（既存のTomcatサーバーを使う必要がある場合）では従来の方法も選択可能
- 組み込みサーバーの設定は`application.properties`や`application.yml`で行う

---

## 理解のポイント

### いつ使うか？
- ✅ 新規のSpring Bootアプリケーション開発
- ✅ Dockerコンテナ化してクラウドにデプロイする場合
- ✅ マイクロサービスアーキテクチャ

### いつ使わないか？
- ❌ 既存のアプリケーションサーバー（Tomcat/WebLogic等）を使う必要がある企業環境
- ❌ 複数のWARアプリケーションを1つのサーバーで動かす必要がある場合

### 従来のWARデプロイとの違い

| 項目 | 組み込みサーバー | WARデプロイ |
|------|-----------------|-------------|
| パッケージ形式 | JAR | WAR |
| サーバー設定 | アプリ内（application.properties） | 外部（server.xml等） |
| 起動方法 | `java -jar myapp.jar` | サーバーにデプロイ後起動 |
| 環境構築 | JDKのみ | JDK + Tomcat等のインストール・設定 |

---

## 具体例

### 従来の方法
```bash
# サーバーでの作業
$ sudo apt-get install tomcat9
$ vi /etc/tomcat9/server.xml  # ポート設定など
$ cp myapp.war /var/lib/tomcat9/webapps/
$ systemctl start tomcat9
```

### Spring Bootの方法
```bash
# どのサーバーでも
$ java -jar myapp.jar
# これだけ！Tomcatの設定は全てJAR内に含まれている
```

---

## 関連

- **公式:** [Spring Boot](https://spring.io/projects/spring-boot)
- **関連概念:** Spring Boot Auto-Configuration, Executable JAR
- **選択可能なサーバー:** Tomcat（デフォルト）, Jetty, Undertow

---

**次のアクション:**
- [ ] application.propertiesでのサーバー設定方法を学ぶ
- [ ] Tomcat以外のサーバー（Jetty/Undertow）への切り替え方法を確認
