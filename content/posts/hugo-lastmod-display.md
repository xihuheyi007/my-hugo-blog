---
date: '2026-05-17T18:41:01+08:00'
draft: false
title: 'Hugo 博客实现文章修改时间显示'
categories:
  - 专业学习
tags:
  - Hugo
  - 技术
description: '详细介绍如何在 Hugo + PaperMod 主题博客中实现文章最后修改时间的自动显示功能。'
---

## 背景

博客文章通常会在发布后经过多次修订和更新。对于读者而言，了解一篇文章的最后修改时间有助于判断内容的时效性。本文记录了在本博客（Hugo + PaperMod 主题）中实现"最后修改时间"显示功能的完整过程。

## 问题分析

PaperMod 主题虽然在配置项中提供了 `ShowLastMod` 参数，但其默认的 `post_meta.html` 模板实际上**并未包含修改时间的渲染逻辑**。因此，仅在 `hugo.yml` 中设置 `ShowLastMod: true` 并不会生效。

要实现该功能，需要完成两件事：

1. 让 Hugo 能够获取每篇文章的最后修改时间
2. 在文章页面的元信息区域渲染该时间

## 实现步骤

### 第一步：启用 Git 信息

Hugo 内置了从 Git 提交记录中提取内容文件修改时间的能力。只需在 `hugo.yml` 中添加一行配置：

```yaml
enableGitInfo: true
```

启用后，Hugo 在构建时会自动读取每个内容文件的 Git 提交历史，将最后一次提交的时间写入 `.Lastmod` 变量。这意味着：

- 不需要在每篇文章的 front matter 中手动维护 `lastmod` 字段
- 修改时间会随着 Git 提交自动更新
- 如果某篇文章从未被修改过，`.Lastmod` 等于 `.Date`（发布日期）

### 第二步：覆盖 post_meta.html 模板

PaperMod 主题的 `post_meta.html` 位于 `themes/PaperMod/layouts/partials/post_meta.html`（或 `_partials`，取决于版本）。该模板负责渲染文章标题下方的元信息栏（发布日期、阅读时间、字数、作者等）。

在项目根目录下创建 `layouts/partials/post_meta.html`，Hugo 会优先使用此文件覆盖主题默认模板。

完整内容如下：

```html
{{- $scratch := newScratch }}

{{- if not .Date.IsZero -}}
{{- $scratch.Add "meta" (slice (printf "<span title='%s'>%s</span>" (.Date) (.Date | time.Format (default ":date_long" site.Params.DateFormat)))) }}
{{- end }}

{{- if (.Param "ShowLastMod") -}}
{{- if (and (not .Lastmod.IsZero) (ne .Lastmod .Date)) -}}
{{- $scratch.Add "meta" (slice (printf "<span>最后修改于 %s</span>" (.Lastmod | time.Format (default ":date_long" site.Params.DateFormat)))) }}
{{- end -}}
{{- end -}}

{{- if (.Param "ShowReadingTime") -}}
{{- $scratch.Add "meta" (slice (printf "<span>%s</span>" (i18n "read_time" .ReadingTime | default (printf "%d min" .ReadingTime)))) }}
{{- end }}

{{- if (.Param "ShowWordCount") -}}
{{- $scratch.Add "meta" (slice (printf "<span>%s</span>" (i18n "words" .WordCount | default (printf "%d words" .WordCount)))) }}
{{- end }}

{{- if not (.Param "hideAuthor") -}}
{{- with (partial "author.html" .) }}
{{- $scratch.Add "meta" (slice (printf "<span>%s</span>" .)) }}
{{- end }}
{{- end }}

{{- with ($scratch.Get "meta") }}
{{- delimit . "&nbsp;·&nbsp;" | safeHTML -}}
{{- end -}}
```

### 关键逻辑说明

新增的修改时间显示逻辑只有三行，但包含了两个重要的判断条件：

```html
{{- if (.Param "ShowLastMod") -}}
{{- if (and (not .Lastmod.IsZero) (ne .Lastmod .Date)) -}}
{{- $scratch.Add "meta" (slice (printf "<span>最后修改于 %s</span>" (.Lastmod | time.Format (default ":date_long" site.Params.DateFormat)))) }}
{{- end -}}
{{- end -}}
```

- **`(.Param "ShowLastMod")`**：检查配置中是否启用了修改时间显示，由 `hugo.yml` 中的 `params.ShowLastMod: true` 控制
- **`not .Lastmod.IsZero`**：确保 `.Lastmod` 变量有值（即 Git 信息可用）
- **`ne .Lastmod .Date`**：只有当修改时间与发布时间不同时才显示，避免对未修改过的文章显示冗余信息

### 第三步：确认 hugo.yml 配置

确保 `hugo.yml` 中包含以下两项配置：

```yaml
enableGitInfo: true    # 启用 Git 信息获取修改时间

params:
  ShowLastMod: true    # 在页面上显示修改时间
```

## 最终效果

完成上述修改后，文章页面的元信息栏将按如下格式显示：

```
发布日期 · 最后修改于 YYYY年M月D日 · 阅读时间 · 字数 · 作者
```

各字段之间以 `·` 分隔，仅在文章确实被修改过（Git 提交时间晚于首次发布时间）时才显示修改时间。

## 注意事项

- `enableGitInfo: true` 要求构建环境中安装了 Git，且内容目录是一个 Git 仓库
- 如果在 front matter 中手动指定了 `lastmod` 字段，Hugo 会优先使用手动值而非 Git 信息
- 自动构建部署场景（如 GitHub Actions）中需要确保 checkout 时包含完整的 Git 历史（`fetch-depth: 0`），否则 `.Lastmod` 可能无法正确获取
- 如需自定义显示文字（如改为"更新于"或英文 "Updated on"），修改 `post_meta.html` 中的对应字符串即可
