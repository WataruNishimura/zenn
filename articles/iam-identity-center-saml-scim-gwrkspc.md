---
title: "AWS IAM Identity CenterのidpにGoogle Workspaceを設定してSSOのAWSマルチアカウント運用に取り組んだ話"
emoji: "🐕"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["aws"]
published: false
---

こんにちは、札幌のエンジニアn13uです
今日はAWS IAM Identity CenterとGoogle Workspace のSAML/SCIM連携を利用したマルチアカウント連携&運用の記事です。


## 概要

AWS IAM Identity Center / AWS Organizationsを利用しAWSアカウント管理するとともに、Google Workspaceを外部Idpに設定しSSO（シングルサインオン）ができるようになるとともにSCIMを利用した自動プロビジョニングまで設定します。

:::message
本記事ではAWSのマルチアカウント運用のベストプラクティスや必要性、AWS OrganizationのOU設計のベストプラクティス等については触れていません
:::


## 基礎知識

### AWSアカウントとIAMユーザー
