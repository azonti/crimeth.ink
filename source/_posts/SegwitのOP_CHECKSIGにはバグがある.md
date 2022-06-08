---
title: SegwitのOP_CHECKSIGにはバグがある
tags:
  - 指摘
date: 2022-06-09 02:42:47
---

「バグ」と言っても、勘違いによってセマンティクスを意図せず変更してしまったという話であって、脆弱性とかではないです。

## 前提

- 秘密鍵と公開鍵の組$(\mathit{sk},\mathit{pk})$があるとする。
- ある取引$\mathit{tx}$があり、その中のある入力を$\mathit{txIn}$とする。
- $\mathit{tx}$の中に$\mathit{txIn}$と同じインデックスをもつ出力は存在しないとする。
- $\mathit{txIn}$の親出力は$\mathit{pk}$でロックされているとする。

## Pre-segwitの場合

$\mathit{txIn}$の親出力をアンロックするためには、$\mathit{sk}$を用いて固定値`0x00...01`の署名$\mathit{bugSig}$を生成する必要がある[^1]。$\mathit{bugSig}$には**この入力と同じインデックスをもつ出力は存在しない**という情報しかコミットされていない。したがって$\mathit{bugSig}$を手に入れた攻撃者は、出力数より入力数が多い取引を構築することによって、$\mathit{pk}$でロックされている任意の出力（$\mathit{txIn}$の親出力以外も）を盗むことができる。

[^1]: [https://en.bitcoin.it/wiki/OP_CHECKSIG#Procedure_for_Hashtype_SIGHASH_SINGLE](https://en.bitcoin.it/wiki/OP_CHECKSIG#Procedure_for_Hashtype_SIGHASH_SINGLE)

## Segwitの場合

$\mathit{txIn}$の親出力をアンロックするためには、$\mathit{sk}$を用いて`outpoint`をコミットした署名を生成する必要がある[^2]。ゆえにこの署名は、少なくとも`outpoint`で指定された出力をアンロックするためにしか使えない。

[^2]: [https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki#Specification](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki#Specification)

## BIP-143の間違い

Segwitにおける`OP_CHECKSIG`のアルゴリズムを規定した[BIP-143](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki)には次のような記述がある[^3]。

> In the original algorithm, a `uint256` of `0x0000......0001` is committed if the input index for a `SINGLE` signature is greater than or equal to the number of outputs. In this BIP a `0x0000......0000` is committed, without changing the semantics.

著者は"without changing the semantics"と述べているが、上述したようにセマンティクスは変化してしまっている。

[^3]: [https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki#cite_note-7](https://github.com/bitcoin/bips/blob/master/bip-0143.mediawiki#cite_note-7)
