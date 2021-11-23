---
author: ["@ottanxyz"]
title: HugoのMarkdownで数式組版ライブラリであるKaTeXをサポートする
date: 2020-05-25T00:00:00+00:00
tags:
  - Hugo
  - KaTeX
categories:
  - Blog
katex: true
---
Hugoで、\\(KaTeX\\)というブラウザで数式を表現するためのライブラリをサポートする方法です。Hugo 0.70.0（Extended）で正常に表示されることを確認していますが、今後のバージョンアップ等により、以下の方法がサポートされなくなる可能性がありますので、ご注意ください。特に、Markdownパーサの変更が発生した場合、影響を受ける可能性があります。

{{% note %}}
Hugo 0.85.0で、Markdownパーサであるmmarkがサポートされなくなるため、KaTeXの導入方法を見直しました。
{{% /note %}}

## 表示

Markdownに記載することで、以下のように表示されます。

```tex
$$
f(x) = \int_{-\infty}^\infty\hat f(\xi)\,e^{2 \pi i \xi x}\,d\xi
$$
```

$$
f(x) = \int_{-\infty}^\infty\hat f(\xi)\,e^{2 \pi i \xi x}\,d\xi
$$

```tex
$$
\frac{1}{\Bigl(\sqrt{\phi \sqrt{5}}-\phi\Bigr) e^{\frac25 \pi}} = 1+\frac{e^{-2\pi}} {1+\frac{e^{-4\pi}} {1+\frac{e^{-6\pi}} {1+\frac{e^{-8\pi}} {1+\cdots} } } }
$$
```

$$
\frac{1}{\Bigl(\sqrt{\phi \sqrt{5}}-\phi\Bigr) e^{\frac25 \pi}} = 1+\frac{e^{-2\pi}} {1+\frac{e^{-4\pi}} {1+\frac{e^{-6\pi}} {1+\frac{e^{-8\pi}} {1+\cdots} } } }
$$

```tex
$$
 \begin{bmatrix}
  a & b \cr
  c & d
  \end{bmatrix}
$$
```

$$
 \begin{bmatrix}
  a & b \cr
  c & d
  \end{bmatrix}
$$

## 結論

1. インラインで描画する場合、`\\(`、および`\\)`で囲む
2. Hugoの設定に`markup.goldmark.renderer.unsafe = true`を追記する
3. KaTeXの[公式ドキュメント](https://katex.org/docs/browser.html)通りに、ライブラリを読み込む。ただし、全てのページでライブラリを読み込むと、ページの描画速度が遅くなるため、必要な場合にのみ読み込むよう一手間の工夫を加える

### Front Matterの修正

まず、`archetypes/default.md`のFront Matterを以下のように修正します。

```md
---
...
katex: false
---
```

`katex`は、Boolean値です。このMarkdownにKaTeXによる数式が含まれるかどうかを明示します。デフォルト値は、`false`です。この値が`true`の場合、テンプレートで、KaTeXによる描画に必要な各種スタイルシートやJavaScriptを読み込みます。

### テンプレートの修正

`<head>`タグに以下の記述を追記します。

```html
<head>
...
{{ block "katex-stylesheet" . }}{{ end }}
...
</head>
```

また、`</body>`タグの直後に、以下の記述を追記します。

```html
{{ block "katex-javascript" . }}{{ end }}
```

続いて、`layouts/_default/single.html`を修正します。

```html
{{ define "katex-stylesheet" }}
{{ if .Params.katex }}
<link
  rel="stylesheet"
  href="https://cdn.jsdelivr.net/npm/katex@0.13.13/dist/katex.min.css"
  integrity="sha384-RZU/ijkSsFbcmivfdRBQDtwuwVqK7GMOw6IMvKyeWL2K5UAlyp6WonmB8m7Jd0Hn"
  crossorigin="anonymous"
/>{{ end }}
{{ end }}

{{ define "katex-javascript" }}
{{ if .Params.katex }}
<script
  defer
  src="https://cdn.jsdelivr.net/npm/katex@0.13.13/dist/katex.min.js"
  integrity="sha384-pK1WpvzWVBQiP0/GjnvRxV4mOb0oxFuyRxJlk6vVw146n3egcN5C925NCP7a7BY8"
  crossorigin="anonymous"
></script>
<script
  defer
  src="https://cdn.jsdelivr.net/npm/katex@0.13.13/dist/contrib/auto-render.min.js"
  integrity="sha384-vZTG03m+2yp6N6BNi5iM4rW4oIwk5DfcNdFfxkk9ZWpDriOkXX8voJBFrAO7MpVl"
  crossorigin="anonymous"
  onload="renderMathInElement(document.body);"
></script>
t>
{{ end }}
{{ end }}
```

Front Matterの`katex`が`true`の場合のみ、KaTexのライブラリを読み込みます。以上で、KaTeXをサポートする準備は整いました。

### `config.yaml`の修正

Hugoの`config.yaml`に、以下を追記します。

```yaml
markup:
  goldmark:
    renderer:
      unsafe: true
```

### インラインでの数式描画

インラインで数式を描画したい場合、`$`ではなく`\\(`と`\\)`で囲むようにします。例えば、`\\(KaTeX)\\)`とMarkdownに記載すると、\\(KaTeX\\)と描画されます。
