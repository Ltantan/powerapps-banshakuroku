# ARCHITECTURE.md — 晩酌録 / BanshakuLog

## 1. システム全体構成図

```mermaid
flowchart TD
    subgraph Solution[Solution BanshakuLog]
        DV[(Dataverse 5テーブル)]
        CA[Canvas App スマホ縦 9画面]
        CodeApp[Code App 管理ダッシュボード]
    end

    User((ユーザー))

    User -- スマホで入力 --> CA
    User -- PCで分析管理 --> CodeApp
    CA -- Patch/Filter --> DV
    CodeApp -- SDK getAll/update/delete --> DV
```

## 2. ER 図（Dataverse 5 テーブル）

```mermaid
erDiagram
    bs_drink {
        string  bs_name PK
        string  bs_dexno
        choice  bs_category
        string  bs_brand
        string  bs_subtitle
        int     bs_volumesize
        decimal bs_alcoholpercent
        string  bs_origin
        money   bs_price
        string  bs_purchasewhere
        date    bs_firstmetdate
        text    bs_description
        image   bs_image
        bool    bs_favorite
        text    bs_note
    }

    bs_snack {
        string bs_name PK
        choice bs_type
        string bs_brand
        text   bs_recipe
        image  bs_image
        text   bs_note
    }

    bs_session {
        string   bs_title PK
        datetime bs_datetime
        choice   bs_location
        string   bs_storename
        choice   bs_mood
        image    bs_image
        text     bs_note
    }

    bs_sessiondrink {
        string bs_name PK
        lookup bs_session FK
        lookup bs_drink FK
        int    bs_volumeml
        int    bs_glasses
    }

    bs_sessionsnack {
        string bs_name PK
        lookup bs_session FK
        lookup bs_snack FK
        string bs_note
    }

    bs_session ||--o{ bs_sessiondrink : has
    bs_session ||--o{ bs_sessionsnack : has
    bs_drink   ||--o{ bs_sessiondrink : drunk_in
    bs_snack   ||--o{ bs_sessionsnack : eaten_in
```

## 3. Canvas App 画面フロー（スマホ縦 9 画面）

```mermaid
flowchart TD
    Home[HomeScreen 今月サマリー]
    DList[DrinkListScreen お酒一覧]
    DDetail[DrinkDetailScreen 図鑑風詳細]
    DEdit[DrinkEditScreen 登録 編集 写真]
    SList[SnackListScreen つまみ一覧]
    SEdit[SnackEditScreen つまみ登録 写真]
    SesList[SessionListScreen 飲酒タイムライン]
    SesEdit[SessionEditScreen 記録登録 写真]
    Insight[InsightScreen 可視化]

    Home --> DList
    Home --> SesList
    Home --> Insight
    Home --> SesEdit

    DList --> DDetail
    DDetail --> DEdit
    DList --> DEdit

    SList --> SEdit

    SesList --> SesEdit
    SesEdit -.参照.-> DList
    SesEdit -.参照.-> SList
```

## 4. Code App 画面構成（PC/タブレット 4 ページ）

```mermaid
flowchart TD
    Dash[ダッシュボード]
    DrinkList[お酒図鑑 一覧テーブル]
    SnackList[おつまみ 一覧テーブル]
    SessionList[晩酌記録 タイムライン]

    Dash -- サイドバー --> DrinkList
    Dash -- サイドバー --> SnackList
    Dash -- サイドバー --> SessionList
    Dash -- カレンダー日付クリック --> SessionDetail[セッション詳細モーダル]
    Dash -- TOP5クリック --> DrinkDetail[お酒詳細モーダル]
    DrinkList -- 詳細ボタン --> DrinkDetail2[お酒詳細 編集モーダル]
    SnackList -- 詳細ボタン --> SnackDetail[おつまみ詳細 編集モーダル]
    SessionList -- 詳細ボタン --> SessionDetail2[セッション詳細 編集モーダル]
```

### ダッシュボード レイアウト

```
┌──────────────────────────────────────────────────┐
│ [ダッシュボード]            2025年11月 〜 2026年5月 │
├──────────┬──────────┬──────────┬──────────────────┤
│ 飲酒日数  │ 合計杯数  │ 月平均   │ 年間換算         │
├──────────┴──────────┴──────────┴──────────────────┤
│ ┌──────────────────┐  ┌──────────────────────────┐│
│ │ カレンダーヒートマップ │  │ 月別杯数 棒グラフ        ││
│ │ (月ナビ/日クリック)  │  │ (X軸2段 月+年)          ││
│ └──────────────────┘  └──────────────────────────┘│
│ ┌──────────────────┐  ┌──────────────────────────┐│
│ │ 酒種別割合 円グラフ  │  │ よく飲む酒 TOP5          ││
│ │ (12時から大きい順)  │  │ (クリックで詳細モーダル)    ││
│ └──────────────────┘  └──────────────────────────┘│
└──────────────────────────────────────────────────┘
```

## 5. シーケンス図（晩酌記録の登録フロー）

```mermaid
sequenceDiagram
    actor User
    participant CA as Canvas App
    participant DV as Dataverse
    participant CodeApp as Code App

    User->>CA: スマホで飲酒記録タップ
    CA->>CA: SessionEditScreen を開く
    User->>CA: 日時と場所を入力
    User->>CA: お酒 おつまみ 写真を選択
    User->>CA: 保存タップ
    CA->>DV: bs_session を Patch
    DV-->>CA: session GUID を返却
    loop 選択された酒ごと
        CA->>DV: bs_sessiondrink を Patch
    end
    loop 選択されたつまみごと
        CA->>DV: bs_sessionsnack を Patch
    end
    CA->>DV: bs_image を追加 Patch
    CA-->>User: 登録完了

    User->>CodeApp: PCでダッシュボード確認
    CodeApp->>DV: SDK getAll で全データ取得
    DV-->>CodeApp: 酒/つまみ/セッション/中間テーブル
    CodeApp->>CodeApp: KPI 棒グラフ 円グラフ TOP5 カレンダー描画
    CodeApp-->>User: 分析結果を表示
```

## 6. 配色トークン（Canvas App + Code App 共通）

| トークン | 値 | 用途 |
|---|---|---|
| appBg / surface | `#FFFFFF` | 通常画面背景 |
| appAccent / amber-accent | `#C68642` | 琥珀色アクセント |
| appAccentSoft / amber-soft | `#E8C896` | 琥珀の薄め ヘッダー帯 |
| appText / ink | `#202020` | 標準文字色 |
| muted | `#8E8E93` | 補助テキスト |
| dexFrame / amber-deep | `#8B4513` | 図鑑外枠 カートリッジ茶 |
| dexBg | `#FCE4E4` | 図鑑内側 薄ピンク |
| surface-dim | `#F8F7F4` | ページ背景（Code App） |

## 7. データソース

| 種別 | 名前 | 用途 |
|---|---|---|
| Dataverse | bs_drinks / bs_snacks / bs_sessions / bs_sessiondrinks / bs_sessionsnacks | 全データ |
| Connector | なし | Power Automate 通知は将来検討 |

## 8. 技術スタック

### Canvas App
- Power Apps Canvas App（スマホ縦）
- MCP (canvas-authoring) 経由で YAML 編集
- AddMedia + Image コントロールで写真登録
- Patch 2段階（本体 → 画像 `{Value: ImgPreview.Image}`）

### Code App
| レイヤー | 技術 |
|---|---|
| UI | React 19 + TypeScript 6 |
| スタイリング | Tailwind CSS v4 |
| チャート | Recharts 2 |
| データフェッチ | TanStack React Query 5 |
| ルーティング | React Router 7 (HashRouter) |
| アイコン | lucide-react |
| ビルド | Vite 8 |
| Dataverse 通信 | @microsoft/power-apps SDK 自動生成サービス |
| デプロイ | pac code push |
