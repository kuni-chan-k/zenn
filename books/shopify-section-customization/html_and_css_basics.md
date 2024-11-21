---
title: "HTMLとCSSの基礎解説"
---

# HTMLとCSSの基礎解説

## この章の目的
この章では、Shopifyテーマカスタマイズに必要なHTMLとCSSの基礎を解説します。  
**ターゲット**は、HTMLやCSSが未経験、または初心者のShopify運用担当者です。  
この章を通じて、次のスキルを習得します：
- Shopifyテーマでよく使うHTMLタグの理解と使い方
- Shopify特有の構造（`section`と`block`）の基本知識
- 基本的なCSSプロパティの使い方

---

## 1. Shopifyテーマでよく使うHTMLタグ

### 1-1. `div`タグ
Shopifyテーマのレイアウトを作成するときに最も頻繁に使うタグが`div`です。  
`div`は「区切り」を意味し、複数の要素をまとめるために使用します。

例:
```html
<div class="banner">
  <h1>ショップのタイトル</h1>
  <p>セール中！今すぐチェック！</p>
</div>
```

**ポイント:**
- `class`属性を使ってCSSでスタイルを適用します。
- 複数の要素を整理するための「箱」として利用します。

---

### 1-2. `img`タグ
画像を表示するためのタグです。Shopifyのバナーや商品画像などで頻繁に使われます。

例:
```html
<img src="example.jpg" alt="商品イメージ">
```

**ポイント:**
- `src`属性で画像のURLを指定します。
- `alt`属性には画像の説明を記載します（アクセシビリティ向上のため）。

---

## 2. Shopify特有の構造

### 2-1. `section`タグ
Shopifyではテーマを構成する主要な単位として`section`を使用します。  
1つの`section`はページ内の特定の領域を担当します。

例:
```html
<section id="custom-banner">
  <div class="banner-content">
    <h1>{{ section.settings.banner_title }}</h1>
    <p>{{ section.settings.banner_text }}</p>
  </div>
</section>
```

**ポイント:**
- `section`はShopifyのカスタマイズ可能な単位です。
- 設定ファイル（JSON）で各セクションの構成を指定します。

---

### 2-2. `block`タグ
`block`は`section`の中で繰り返し可能な要素を作るために使用します。  
Shopifyの動的なコンテンツを管理する上で重要です。

例:
```liquid
{% for block in section.blocks %}
  <div>{{ block.settings.title }}</div>
{% endfor %}
```

**ポイント:**
- 複数のアイテム（例: 商品や画像）を柔軟に表示できます。
- Liquidテンプレート内で利用します。

---

## 3. CSSプロパティの基礎

### 3-1. `color`プロパティ
テキストや背景色を変更するために使用します。

例:
```css
h1 {
  color: #333333;
}
```

**ポイント:**
- テキストの色を指定します（16進数や色名で設定可能）。
- ブランドカラーに合わせた配色を指定しましょう。

---

### 3-2. `padding`プロパティ
要素の内側に余白を追加するプロパティです。

例:
```css
.banner {
  padding: 20px;
}
```

**ポイント:**
- 数値の単位（`px`や`%`）で指定します。
- レスポンシブデザインを意識して適切に設定しましょう。

---

## 練習課題
以下の課題を試して、HTMLとCSSの基礎を実際に体験しましょう。

### 課題1: シンプルなバナーを作成する
1. 以下のHTMLコードをコピーして、Shopifyテーマの`section`として追加してください。
2. 適切なCSSを記述して、以下のような見た目を作りましょう。

**HTML**
```html
<div class="custom-banner">
  <h1>ショップへようこそ</h1>
  <p>セール開催中！</p>
</div>
```

**CSS**
```css
.custom-banner {
  background-color: #f5f5f5;
  text-align: center;
  padding: 20px;
}
```

---

## 次のステップ
この章で学んだHTMLとCSSの基礎を活用して、次章ではShopifyセクションをカスタマイズする方法を学びます。最初のステップとして、バナーセクションの作成に取り組んでみましょう！
