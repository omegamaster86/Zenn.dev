---
title: "AmiVoiceとCursor SDKで音声情報を取得、要約するんじゃ"
emoji: "🎤"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["AmiVoice","Cursor","Cursor SDK","AI駆動開発"]
published: false
# published_at: 2026-05-08 09:30
publication_name: "genai"
---
# 初めに
この記事はZennfes Spring 2026「音声認識AmiVoice APIと生成AIで作る音声体験」エントリ記事です。

AmiVoice API と Cursor SDK を組み合わせて「音声を取り込み → テキスト化 → 要約」する流れを作る前提で書いていきます。まずは同じ **Speech-to-Text（音声認識）** の領域で、AmiVoice と並んで検討される製品・サービスを整理しておきます。


# 他製品紹介
最後にAmiVoice と同じく **音声をテキストに変換する API / SDK / サービスを紹介させてください**、大きく次の4系統に分かれます。

| 系統 | 代表例 | AmiVoice との関係 |
| --- | --- | --- |
| 国内・日本語特化 | AmiVoice、Voioi、ReazonSpeech | 日本語ビジネス会話・専門用語向け |
| グローバルクラウド API | Google / Azure / AWS / Deepgram / AssemblyAI など | 開発者向け STT API として直接競合 |
| OSS / 自前ホスト | Whisper、ReazonSpeech、Kotoba-Whisper | API ではなく自前運用。データ持ち出し制約向き |
| SaaS（完成プロダクト） | Notta、Rimo Voice、LINE WORKS AiNote | API 統合ではなく GUI で完結 |

## 国内・日本語特化

| サービス | ストリーミング | 主な特徴 |
| --- | --- | --- |
| [AmiVoice Cloud Platform](https://www.advanced-media.co.jp/amivoice/cloud/) | ✅ | 国内シェア No.1。医療・金融など領域特化エンジン、専用環境構築、日本語サポート、**発話時間のみ課金** |
| Voioi | ✅ | 93言語対応。リアルタイム・ファイル両方。国内向け |
| [ReazonSpeech](https://github.com/reazon-research/ReazonSpeech) | △ | 日本語特化 OSS。自前ホスト or API 利用 |

AmiVoice の強みは **日本語精度**、**専門用語辞書**、**国内サポート**、**オンプレ / 専用環境** あたり。海外 API と比べて、日本語のビジネス会話や専門分野では依然有力です。

:::message
AmiVoice 提供元のアドバンスト・メディアは、[音声認識 API 主要 7 社の価格・機能比較表（2026 年版）](https://www.advanced-media.co.jp/newsrelease/11316/) を公開しています。客観的な比較資料として参考になります。
:::

## グローバルクラウド STT API

AmiVoice と同じく **REST / WebSocket API + ストリーミング** で使える開発者向けサービスです。

| サービス | ストリーミング | 日本語 | ざっくり特徴 |
| --- | --- | --- | --- |
| [Google Cloud Speech-to-Text](https://cloud.google.com/speech-to-text) | ✅ | ✅ | 125+ 言語。話者分離・カスタム語彙。設定はやや複雑 |
| [Azure Speech Services](https://azure.microsoft.com/products/ai-services/speech-to-text) | ✅ | ✅ | エンタープライズ向け。価格競争力あり |
| [AWS Transcribe](https://aws.amazon.com/transcribe/) | ✅ | ✅ | AWS 連携が前提なら自然。カスタム語彙対応 |
| [Deepgram](https://deepgram.com/) | ✅ | ✅ | **低遅延（~300ms）**・低コスト。ライブ字幕向き |
| [AssemblyAI](https://www.assemblyai.com/) | ✅ | ✅ | 開発者体験が良い。要約・感情分析など付加機能も |
| [OpenAI Whisper API](https://platform.openai.com/docs/guides/speech-to-text) | ❌ | ✅ | 精度は高いが **バッチのみ**（リアルタイム向きではない） |
| [Rev AI](https://www.rev.ai/) | ✅ | △ | 精度重視だが **高コスト** |
| [Speechmatics](https://www.speechmatics.com/) | ✅ | ✅ | 英語・多言語で精度評価が高い |
| [Soniox](https://soniox.com/) | ✅ | △ | 新興。低コスト・低遅延を謳う |

ストリーミング API の比較記事としては、[ストリーミング音声認識 API/SDK の最新比較（2025 年）](https://zenn.dev/kennejima/articles/56f60e1962291e) も参考になります。

## OSS / 自前ホスト

API ではなく、自前で STT 基盤を組む選択肢です。

| 選択肢 | 特徴 |
| --- | --- |
| [OpenAI Whisper](https://github.com/openai/whisper) | 精度は高い。GPU 推奨。**ストリーミング非対応**（バッチ向き） |
| [ReazonSpeech](https://github.com/reazon-research/ReazonSpeech) | 日本語特化 OSS |
| [Kotoba-Whisper](https://huggingface.co/kotoba-tech/kotoba-whisper-v2.0) | 日本語 Whisper 派生 |
| faster-whisper + pyannote | 話者分離を自前で組み合わせる構成 |

**データを外に出せない**（オンプレ・ローカル必須）場合は、AmiVoice の専用環境構築 or Whisper 自前運用が現実的な選択肢になります。

## SaaS（API ではないが用途は近い）

Cursor SDK で要約までやるなら、API 統合ではなく **完成プロダクト** として検討する選択肢もあります。

| サービス | 特徴 |
| --- | --- |
| [Notta](https://www.notta.ai/ja) | 58 言語。Zoom 連携。個人向け月額あり |
| [Rimo Voice](https://rimo.app/) | 日本語精度が高め。編集 UI が使いやすい |
| [LINE WORKS AiNote](https://line-works.com/ai-note/) | 無料枠あり（月 300 分）。手軽 |
| Google AI Studio (Gemini) | ファイルアップロードで文字起こし + 要約。API というより GUI |

## どれを選ぶか

用途別のざっくり目安です。

```
リアルタイム文字起こし（ストリーミング）
  ├─ 日本語・専門用語・国内サポート重視 → AmiVoice
  ├─ 低遅延・低コスト重視             → Deepgram / AssemblyAI
  └─ 既存クラウド統合                → Azure / Google / AWS

ファイル文字起こし（バッチ）
  ├─ 手軽さ重視                      → OpenAI Whisper API
  └─ 日本語精度重視                  → AmiVoice / ReazonSpeech

データを外に出せない
  └─ AmiVoice 専用環境 or Whisper 自前ホスト
```
