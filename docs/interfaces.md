# 観測系の境界の一覧

45m 鏡の観測系の、部品のあいだの境界（インターフェース）の一覧です。各境界について、何が流れるか、担う資産、仕様の所在と版、仕様にまだ移していない断片の所在、公開か非公開か、欠けていることを示します。

- **仕様の正本は [nro45-interfaces](https://github.com/nobeyama-radio-observatory/nro45-interfaces) です。** ここには仕様を書き写さず、境界の状態と所在だけを書きます。
- この一覧の置き場所と書く項目は、ハブの判断記録 [0013](https://github.com/nobeyama-radio-observatory/nro-context/blob/main/decisions/0013-観測系のインターフェース仕様をnro45-interfacesにまとめる.md) で決めました。
- 範囲は、典型的なポジションスイッチ（PSW）観測の流れです。OTF などのほかの観測のモードは、後から足します。
- 制御の手順、装置のあいだの通信の中身、接続先、認証、計算機の上のパスは書きません。

確認日：2026-11-19（台帳の id は同日に足した `sw-cosmos-lc`、`sw-cosmos-merge`、`sw-sam45compat-lc`、`db-yksp-delivery` を含む）

## 流れの全体

```
観測者
  │ 作成
  ▼
観測指示書の作成ソフト（nobs）
  │ ① 観測指示書
  ▼
COSMOS のマネージャー ◀──② 命令とエラー（非公開）──▶ 各装置の Local Controller
                                                      │ ③ 観測時系列データ
                                                      ▼
                                          quick look（観測中の表示）
                                          マージ（観測の終了時にマネージャーが起動）
                                                      │ ④ 観測生データ
                                                      ▼
                                          保管（NRODB、電波アーカイブ）──⑤ 保管と配信──▶ 観測者
                                                      │
                                                      ▼ ⑥
                                          解析ソフト（NEWSTAR、NOSTAR、CASA への書き出し）
```

出典：YATSUGATAKE-NRO45-sam45compat の [docs/cosmos/observation-flow.md](https://github.com/nobeyama-radio-observatory/YATSUGATAKE-NRO45-sam45compat/blob/main/docs/cosmos/observation-flow.md) の「全体の流れ」

## 一覧

| 境界 | 流れるもの | 書く側 → 読む側（台帳の id） | 仕様（nro45-interfaces） | 公開 | 状態 |
|---|---|---|---|---|---|
| ① 観測指示書 | 観測指示書のファイル | `tool-obs-script` → `sw-cosmos` | `observation-script/`（準備中） | 公開（仮説） | 断片あり |
| ② マネージャーと Local Controller | 命令、エラー、装置の状態 | `sw-cosmos` ↔ `sw-cosmos-lc` | 置かない | 非公開 | 所在が記録にない |
| ③ 観測時系列データ | 装置ごとの時系列のファイル（いまの呼び名は「ロギング」） | `sw-cosmos-lc`、`sw-sam45compat-lc`（YKSP）→ quick look（`sw-obs-ui`）、`sw-cosmos-merge` | `timeseries/`（準備中） | 公開できるかは確認中 | 断片あり（SAM45 だけ） |
| ④ 観測生データ | 観測ごとに1つのファイル | `sw-cosmos-merge` → `db-nrodb-nobeyama`、`db-yksp-delivery`、解析ソフト | `raw-data/`（準備中） | 公開（仮説） | 記録がない |
| ⑤ 保管と配信 | 保管の単位と、観測者が取得するときの形 | `db-nrodb-nobeyama`、`db-nrodb-mitaka`、`db-radio-archive`、`db-yksp-delivery`（暫定）→ 観測者 | `delivery/`（準備中）。規則は nro-data の [docs/data-path.md](https://github.com/nobeyama-radio-observatory/nro-data/blob/main/docs/data-path.md) | 形式と規則は公開。取得の方法は非公開 | 断片あり（暫定の YKSP だけ） |
| ⑥ 解析 | 解析ソフトが読む形式 | 観測生データ → `tool-newstar`、`tool-nostar`、`tool-cont`、`tool-casa-export` | ④ と同じ | 公開 | 記録がない |

## 境界ごとの詳細

### ① 観測指示書

- **流れるもの：** 観測指示書は、同じ名前で拡張子だけが違うファイルの組で出る。
  - `.start`：1行に1命令で、1行目から順に実行される。命令は装置ごとに書く。`%` と `#` で始まる行はコメントで、COSMOS は読まない。
  - `.dat`：分光計の前の共通 IF 系の、Array ごとの設定。`.start` から参照される。
- **出典：**
  - observation-flow.md の「1. 観測指示書」
  - tz230-firstlight の [docs/lessons.md](https://github.com/ars096/tz230-firstlight/blob/main/docs/lessons.md) の「2. nobs が出す指示書」
- **作成ソフトを替えても、形式は変えない予定である。** nobs は近いうちに大幅に改修し、別のソフトに置き換える予定とされている（observation-flow.md の1）。この境界が、作り直しのときの契約になる。
- **断片：**
  - tz230-firstlight の [docs/fields.md](https://github.com/ars096/tz230-firstlight/blob/main/docs/fields.md)：周波数まわりの欄と、欄どうしの関係。H40 のサンプル1件から読み取り、検算したもの
  - 同じリポジトリの [docs/changes.md](https://github.com/ars096/tz230-firstlight/blob/main/docs/changes.md)：H40 から TZ 230 GHz への書き換え
- **欠けていること：**
  - 命令の一覧と、各パラメータの型、単位、範囲、既定値（記録がない）
  - 周波数のほかの欄（アンテナ、スキャン、積分など）
  - H40 のほかの受信機での形。TZ 86 GHz で同じ形かも未確認（fields.md）
  - 7BEE と eQ（nobs は対応していない。7BEE は別の手順で作る。nro-user-tools の台帳の `tool-obs-script`）
  - 指示書に形式の版を示す欄があるか（記録がない）
  - サンプルと、実機で観測者が作ったものとで形が違うことがある（lessons.md）。違いの範囲は記録がない
- **公開の注意：** 指示書には観測者 ID が入る（lessons.md）。例とテストケースにするときは伏せるか、合成した例にする。

### ② マネージャーと Local Controller

- **流れるもの：** マネージャーは、指示書の装置ごとの命令を、各装置の Local Controller に送る。Local Controller は共通の規格で作られており、エラーをマネージャーに返す。マネージャーは観測中も Local Controller の状態を受け取り、装置の異常を検出する（observation-flow.md の2、3）。
- **公開：** 非公開。通信の方式と中身は書かない。
- **非公開の文書の所在：** 記録がない。ハブの[非公開の一覧](https://github.com/nobeyama-radio-observatory/nro-context/blob/main/policy/non-public.md)の「所員向けの制御の手順書と構成の資料」も（要確認）のまま。文書名、版、ページで引用できる文書があるかを、計算機グループに確かめる。
- **関係する予定：**
  - YATSUGATAKE の最終版の連携ソフトウェア（YKSP-LC）は、マネージャーにつながる予定である（[聞き取り 2026-11-18](../interviews/2026-11-18.md) 問6）。
  - アンテナ制御ユニット（ACU）のインターフェースが変われば、COSMOS の改修も要る（同 問6）。ACU との境界は nro-45m-antenna の `nro45m-acu` が持つ。
- **欠けていること：** 各装置の Local Controller は、まとめて `sw-cosmos-lc` として台帳に置いた。装置ごとに分けるかは聞き取りで決める（下の「聞き取りの候補」6）。

### ③ 観測時系列データ

- **流れるもの：** 装置ごとの時系列のデータ（例：アンテナの向き、分光計のスペクトル）。
  - ファイルは観測の開始時にでき、観測中に書き足される。
  - ほかの計算機と共有され、quick look とマージが読む。
  - 観測者は、通常は直接は触らない。
  - 出典：observation-flow.md の4。sam45compat の [docs/cosmos/sam45-logging.md](https://github.com/nobeyama-radio-observatory/YATSUGATAKE-NRO45-sam45compat/blob/main/docs/cosmos/sam45-logging.md)
- **SAM45 について分かっていること：**
  - 1観測で1ファイルで、全 Array の分が一つのファイルに入る。
  - 形式の版を示す欄がある。
  - YATSUGATAKE で SAM45 の故障した Array を肩代わりするデータ（YKSP）は、SAM45 と同じ形式で書かれる（YATSUGATAKE-NRO45-sam45compat の README）。
  - 野辺山の NRODB は、2015年度から SAM45 のロギングのデータも保管している（nro-data の台帳の `db-nrodb-nobeyama`）。観測時系列データも保管の対象になる。
- **断片：** sam45-logging.md は SAM45 の形式を要約している。COSMOS のヘッダファイルと実際のデータ1件を突き合わせて読んだもの。
- **公開：** 確認中。COSMOS のソースから読み取った形式を公開してよいかは、契約の確認が要る（0013）。確認が済むまで、nro45-interfaces には書かない。
- **欠けていること：**
  - アンテナなど、SAM45 のほかの装置の形式（記録がない）
  - YKSP が SAM45 と完全に同じ形式か、違う部分があるか
  - 公開してよいかの確認

### ④ 観測生データ

- **流れるもの：** 観測の終了時にマネージャーがマージを起動する。マージは各装置の観測時系列データを読み、観測ごとに1つの観測生データを作る。観測者は通常、この観測生データを取得して解析する。マージのスクリプトには、観測者はアクセスできない（observation-flow.md の6）。
- **YKSP：** SAM45 とは別の観測生データとして作る。観測者から見ると、観測生データが SAM45 と YKSP の二つになる（YATSUGATAKE-NRO45-sam45compat の README）。
- **欠けていること：**
  - 観測生データの形式の記述（記録がない。断片も見つかっていない）
  - 形式の版を示す欄があるか（記録がない）
  - マージの保守の担い手の数（台帳の `sw-cosmos-merge` は未記入）

### ⑤ 保管と配信

- **SAM45 の経路：**
  - 観測とマージが終わると、野辺山の NRODB に保存する。週1回の保守のときに三鷹の NRODB に送り、二か所で持つ。
  - 電波アーカイブには、NRODB から取り出してパイプラインを適用し、登録する。この作業には人手がかかっている。
  - 観測者は Web とコマンドラインで取得する。
  - 出典：nro-data の台帳の `db-nrodb-nobeyama`、`db-nrodb-mitaka`、`db-radio-archive`
- **暫定の YKSP の経路：**
  - SAM45 とは別に配信している。観測者は、自分のグループとプロジェクトの単位で取得する。取得できるのは所内 LAN からだけ。
  - 1年を過ぎたものの扱いは決まっていない。
  - 出典：YATSUGATAKE-NRO45-sam45compat の docs/plan.md の5
- **公開：** 保管の単位と規則は公開する。取得の方法（接続と認証）は非公開。正本は、計算機グループの計算機とネットワークのシステムのページ（ハブの非公開の一覧）。
- **欠けていること：**
  - YKSP を電波アーカイブに入れるか、三鷹に二重化するか
  - 保管期間を過ぎたデータの扱い
  - リモートの観測者の取得の方法
  - 1観測あたりと1年あたりのデータ量
  - NRODB を改修するか、新しいシステムを作るか（nro-data の台帳の `renewal`）
  - 規則と状態は nro-data の [docs/data-path.md](https://github.com/nobeyama-radio-observatory/nro-data/blob/main/docs/data-path.md) にまとめた。

### ⑥ 解析

- **流れるもの：** 解析ソフトは観測生データを読む。
  - PSW は NEWSTAR、OTF は NOSTAR、連続波は cont で解析する。
  - 解析の主力は今後 CASA に移る。CASA の形式への書き出しは、アーカイブと連携している。
  - 出典：nro-user-tools の台帳
- **欠けていること：**
  - 各解析ソフトが、どの版の観測生データを読めるか（記録がない）
  - YKSP の観測生データを、いまの解析ソフトがそのまま読めるか（記録がない）
  - NEWSTAR、NOSTAR、cont は担い手が 0 で、維持していない（nro-user-tools の台帳）。形式が変わったときの対応の担い手がいない。

## この一覧の範囲の外

次の境界も観測に関わります。PSW の流れを書き終えたら、足すかを決めます。

- アンテナ制御ユニット（ACU）と COSMOS（非公開。nro-45m-antenna）
- 受信機と IF 系の設定。① の `.dat` と、受信機ごとの LO の切り替え（nro-45m-receivers、nro-45m-if）
- 標準時計と分光計の時刻。PSW の観測は 1PPS の秒の境界から始まる（observation-flow.md の7。nro-time-standard）
- リモート観測と、観測の操作のユーザーインターフェース（`sw-remote-obs`、`sw-obs-ui`）

## 聞き取りの候補

計算機グループに、一度にまとめて聞きます。

1. ② のマネージャーと Local Controller の通信を、文書名、版、ページで引用できる非公開の文書があるか
2. ③ の形式（COSMOS のヘッダから読み取ったもの）を公開してよいか。メーカーとの契約に制約があるか
3. ④ の観測生データの形式の記述があるか。形式の版を示す欄があるか
4. ① の命令の一覧と、パラメータの型、単位、範囲の記述があるか。nobs のソースの所在と言語
5. ⑤ の YKSP の保管と配信の方針（アーカイブ、二重化、保管期間、リモートからの取得、データ量）
6. 各装置の Local Controller（いまは `sw-cosmos-lc` にまとめている）を、装置ごとに分けて書くか。マージと Local Controller の保守の担い手の数
