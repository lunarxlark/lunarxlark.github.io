---
title: Go Code Review Comments (2021年時点)
tags: ["go"]
date: 2021-08-23
published: false
---

## 原典

[Go Code Review Comments](https://github.com/golang/go/wiki/CodeReviewComments)

## 動機

新しいチームメンバーに、Go の文化を知ってもらう一環として。
(GitHub や Twitter で指摘してもらえると直します。)

以下、訳。

## Go Code Review Comments

略語で一つの詳細な説明まで辿り着けるように、このページでは Go のコードをレビューしている間に作られた多くの意見を集めました。
このページはよくある間違えのリストであって、包括的なスタイルガイドではありません。

[Effective Go](https://golang.org/doc/effective_go)を補足として参照するとよいです。

`小さな変更であっても、このページを編集する際は[変更内容を議論してください](https://github.com/golang/go/wiki/CodeReviewComments)。`
多くの人が意見を持っていて、このページは編集戦争をする場ではありません。

- [Gofmt](#Gofmt)
- Comment Sentences
- Contexts
- Copying
- Crypto Rand
- Declaring Empty Slices
- Doc Comments
- Don't Panic
- Error Strings
- Examples
- Goroutine Lifetimes
- Handle Errors
- Imports
- Import Blank
- Import Dot
- In-Band Errors
- Indent Error Flow
- Initialisms
- Interfaces
- Line Length
- Mixed Caps
- Named Result Parameters
- Naked Returns
- Package Comments
- Package Names
- Pass Values
- Receiver Names
- Receiver Type
- Synchronous Functions
- Useful Test Failures
- Variable Names

### Gofmt
