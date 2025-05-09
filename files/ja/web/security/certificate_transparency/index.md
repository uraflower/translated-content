---
title: 証明書の透明性
slug: Web/Security/Certificate_Transparency
l10n:
  sourceCommit: 5b755904cd31e7329ee32ace99486a2fea0fe6a1
---

{{QuickLinksWithSubpages("/ja/docs/Web/Security")}}

**証明書の透明性** (Certificate Transparency) は、証明書の誤発行を防止し、監視するために設計されたオープンなフレームワークです。証明書の透明性においては、新しく発行された証明書は、公開で運営されている、多くの場合独立した CT ログに「記録」され、発行された TLS 証明書の追加のみの暗号的に保証された記録を維持します。

このようにして、認証局 (CA) は、はるかに大きな監視と監督を受けることができます。CA/B フォーラムの*ベースライン要件*に違反するような、潜在的に悪意のある証明書は、より迅速に検出され、失効される可能性があります。また、ブラウザーベンダーやルートストアのメンテナは、不信に繋がるかもしれない問題がある CA について、より多くの情報に基づいた決定を下すことができるようになります。

## 背景

CT ログは _Merkle_ ツリーのデータ構造をベースに構築されています。ノードには子ノードの*暗号化ハッシュ*がラベル付けされています。リーフノードには実際のデータのハッシュが含まれています。したがって、ルートノードのラベルは、ツリー内の他のすべてのノードに依存します。

証明書の透明性のコンテキストでは、リーフノードによってハッシュ化されたデータは、現在運営されている様々な異なる CA によって発行された証明書です。証明書の組み込みは、対数的な O(log n) 時間で効率的に生成・検証できる*監査証明*を介して検証することができます。

証明書の透明性は、認証局の危殆化 (2011 年の DigiNotar の違反)、疑わしい決定 (2012 年の Trustwave の下位ルート事件)、技術的な発行問題 (マレーシアの DigiCert Sdn Bhd による 512 ビットの脆弱な証明書の発行) を背景に、2013 年に初めて実現しました。

## 実装

証明書が CT ログに送信されると、_署名付き証明書のタイムスタンプ_ (SCT) が生成されて返されます。これは、証明書が提出され、ログに追加されることを証明する役割を果たします。

仕様では、準拠サーバーは接続時にこれらの SCT を TLS クライアントに提供*しなければならない*とされています。これは、いくつかの異なるメカニズムを介して実現することができます。

- 署名付き証明書のタイムスタンプを直接リーフ証明書に埋め込む X.509 v3 証明書拡張機能
- ハンドシェイク中に送信される `signed_certificate_timestamp` 型の TLS 拡張
- OCSP のステープリング (つまり、`status_request` TLS 拡張) と、1 つ以上の SCT を持つ `SignedCertificateTimestampList` の提供

X.509 資格情報の拡張機能により、記載する SCT は発行 CA によって決定されます。 2021 年 6 月以降、最も広く使用され、有効な一般的に信頼されている資格情報には、この拡張機能に埋め込まれた透明性データが含まれています。 このメソッドでは、ウェブサーバーの修正は必要ありません。

後者の方法では、必要なデータを送信するためにサーバーを更新する必要があります。利点は、サーバーオペレータが TLS 拡張/stapled OCSP レスポンスを介して送信される SCT を提供する CT ログソースをカスタマイズできることです。

## ブラウザーの要件

Google Chrome バージョン 107 以降では、2018 年 4 月 30 日以降の notBefore 日付を持つすべての証明書の問題に対して、CT ログのインクルードを要求しています。これにより、ユーザーは非準拠の TLS 証明書を使用したサイトを訪問できなくなります。
これまで Chrome では、_Extended Validation_ (EV) や Symantec が発行した証明書に対して CT のインクルードが義務付けられていました。

Apple は、Safari や他のサーバーがサーバー証明書を信頼するために、さまざまな数の SCT を[必要としています](https://support.apple.com/ja-jp/HT205280)。

Firefox デスクトップ版バージョン 135 以降では、Mozilla's Root CA Program に収録されているすべての認証局によって発行された証明書に対して CT ログのインクルードが要求されます。
現時点で、Android 版では CT ログのインクルードが要求されていません。

## 仕様書

2025 年 1 月時点で、ブラウザー実装は古い仕様である {{rfc("6962","Certificate Transparency")}} に基づいています。
現在の仕様は {{rfc("9162","Certificate Transparency Version 2.0")}} です。

## 関連情報

- [Apple の Certificate Transparency ログプログラム](https://support.apple.com/ja-jp/103703)
- [Chrome's Certificate Transparency Log Policy](https://googlechrome.github.io/CertificateTransparency/log_policy.html)
