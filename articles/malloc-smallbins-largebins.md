---
title: "malloc.c を読む (smallbins, largebins)"
emoji: "💨"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: []
published: false
---

    if (largebins のサイズ) {
      if (largebins の末尾から取得) {
        unlink
        if (nb + MINSIZE 未満) {
          exhaust
        } else {
          split
          unsortedbin に挿入
        }
        返す
      }
    }

    idx++;

    for (;;) {
      binmap から
      if (何もついてない) {
        binmap に書き込む
      } else {
        unlink
        Exhaust
        Split
      }
    }

      smallbins, largebins に振り分け
      binmap に書き込む

    smallbins, largebins から確保
    binmap から検索、なければ clear