---
title: 'メモリ管理'
date: 2020-10-05T18:07:06+09:00
description:
menu:
  sidebar:
    name: メモリ管理
    identifier: memory-management
    parent: computer-science
    weight: 10
tags: ['computer-science']
categories: ['computer-science']
---

## はじめに

バックエンドエンジニアの[ロードマップ][ロードマップ]に沿ってエンジニアとしての自己肯定感を養うシリーズです。

## 仮想メモリ

[プロセス管理][プロセス管理]でもあったように、メモリはアドレス空間ごとにプロセスを管理します。

アドレス空間は 4KB/8KB 単位の**ページ**に分割して管理されています。

ページはそれぞれ論理アドレス、物理アドレスを対応づける単位でもあります。

論理アドレスと物理アドレスは常に紐づけられているわけではなく、そのページが必要になった時点で割り当てることも可能です。

そのため、論理アドレスを実際の物理アドレスの容量より大きく確保することができます。

(実際に使えるメモリの量よりも大きなメモリを想定できるということです。)

仮装メモリとして使う仕組みには次の３つが挙げられます。

## ページング

仮想メモリといえばこれ、という風に教えられるものの筆頭かと思います。

ハードディスクを物理メモリの代わりに使うといったものです。

物理メモリが不足すると、OS のコアである**カーネル**は使われていないページをハードディスクに移して論理アドレスを解放します。

そしてプロセスがハードディスクに移されたページにアクセスしようとすると、カーネルがプロセスを停止し、ハードディスクのページを再度物理メモリに読み込み、論理アドレスを対応づけます。

また、プロセス全体を単位にする場合は**スワッピング**と呼ばれます。

## メモリマップトファイル

ファイルをメモリとしてアクセスすることができるものです。

アクセスがあった瞬間に、カーネルがファイルをメモリに読み込みます。プロセスがメモリを使い終わると、論理アドレスと物理アドレスを解放して、メモリの内容をファイルに保存します。

## 共有メモリ

１つの物理アドレスを、複数のプロセスの論理アドレスに対応づけるものです。
アドレス空間をまたぐと危険では？！という見方もありますが、複数プロセスで処理できるため、巨大な画像データを編集するときには都合が良いみたいです。

※Go では共有メモリを使わずに Message Passing を使っています。

## メモリ管理 API

### malloc(3)

メモリを**ヒープ領域**に割り当てます。プログラム実行時に決まるサイズのメモリはヒープ領域から確保します。

ヒープは「何かを積み重ねた山」のことで、その名の通り、プログラムを実行してから決定する量だけメモリを確保しておく領域なので納得です。

malloc で確保したメモリは[free](#free)で解放しなければいけません。

### calloc(3)

メモリをヒープ領域に割り当てます。malloc と異なる点は、割り当てたメモリをゼロクリアすることです。

こちらも malloc 同様、確保したメモリは[free](#free)で解放しなければいけません。

### realloc(3)

malloc で割り当てたメモリのサイズを拡大、縮小します。こちらも確保したメモリは[free](#free)で解放しなければいけません。

### free

割り当てたメモリを開放します。いったん開放したアドレスにはアクセスしてはいけません。

メモリの開放漏れを防ぐために、malloc で確保したメモリは常に free で開放されるべきです。

### brk(2)

malloc や realloc が割り当てるためのメモリを探してくるものです。

物理アドレスが割り当てられていないページに物理アドレスを対応づけます。

### 余談

メモリはエラーでもかなりお世話になる部分なので、次回以降、実際のエラーやプログラミング言語(Go か Java)に絡めた記事を書きたいです。

## 備考

[ふつうの Linux プログラミング 第 2 版 Linux の仕組みから学べる gcc プログラミングの王道][amazon]

表紙イラスト：[Loose Drawing](https://loosedrawing.com/)

[ロードマップ]: https://github.com/kamranahmedse/developer-roadmap#back-end-roadmap
[amazon]: https://www.amazon.co.jp/%E3%81%B5%E3%81%A4%E3%81%86%E3%81%AELinux%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0-%E7%AC%AC2%E7%89%88-Linux%E3%81%AE%E4%BB%95%E7%B5%84%E3%81%BF%E3%81%8B%E3%82%89%E5%AD%A6%E3%81%B9%E3%82%8Bgcc%E3%83%97%E3%83%AD%E3%82%B0%E3%83%A9%E3%83%9F%E3%83%B3%E3%82%B0%E3%81%AE%E7%8E%8B%E9%81%93-%E9%9D%92%E6%9C%A8-%E5%B3%B0%E9%83%8E/dp/4797386479
[プロセス管理]: https://uh-zz.github.io/blog/posts/about-process/