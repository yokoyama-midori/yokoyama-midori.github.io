---
title: Cコンパイラを作る
date: 2025-9-11 10:00 0900
categories: [コンパイラ]
tags: [C]
math : true
---

Rui Ueyamaさんの[『低レイヤを知りたい人のためのCコンパイラ作成入門』](https://www.sigbus.info/compilerbook)と[『chibicc』のリファレンス実装](https://github.com/rui314/chibicc/tree/reference)を読みながらCコンパイラをCで作っていきます。セルフホストできるところまで進めるのが当面の目標です。日記形式で書いていきます。

### 9/2 ～ 9/10
「ステップ14: 関数の呼び出しに対応する」まで進めた。

### 9/11
[Commit aedbf56
](https://github.com/rui314/chibicc/commit/aedbf56c3af4914e3f183223ff879734683bec73#diff-629fe11334ae1d560032cdb6cc6f9a4fbb0f5b1365894b6b648d6ee4d5a654beR105-R106) では関数呼び出しの際RSPが16の倍数になるように条件分岐させている。僕の実装だと、mainのプロローグの際にRSPをローカル変数の分だけ引くことになるがそこを16の倍数になるようにしており、コード生成の際にはpushした分だけpopしてるはず。従ってこういうチェックをしなくても16の倍数に最初からなっている...はず。ここに関わらず僕の実装とchibiccの実装が異なっている箇所が色々ある。後々沼にハマることを考えるとなるべく同じロジックにしておきたい気もするがどうしようか？とりあえずこのコミットは無視して進めることにする。

引数なしの関数宣言を実装していく  
↑foo(){return 1;}bar(){return 2;}main(){return foo()+bar();}みたいなときに正しく16の倍数になっていない気がする。のでやっぱりcall前の16の倍数になるような調整をいれる。

もともとgenはreturnしたときに1つスタックにpushされているつもりだったが、それも難しそうな気がしてきたのでその前提でpopしてあるコードは消した。(そこに気づかずにwihleのデバッグが長引いた)chibiccを眺めながらなんとか完了。関数ごとにローカル変数を持っているので、別の関数が同じ変数名を持てる。ブロックのスコープには対応していない。

6個までの引数を持つ関数の宣言に対応  
入出力があれば簡単な競プロの問題が解けそうなコンパイラになってきた。ところでC言語って殆ど書いたことなかったけどC++に比べて全然機能がなくて驚かされる。

### 9/12,13
関数・仮引数・ローカル変数の宣言にintをつけるようにした  
`int`や`int*,int**`,...が扱える型を導入する。
`Type*` は各Nodeに持たせるがVarには持たせていない。  
`int main(){ int **a; *a; }`   
のような場合に`int **a`の型は`*`の個数ですぐに分かるが'*a'の型をどう決めるのかに悩む
`Var`に紐づけるべきな気がするな...

### 9/14
リファレンス実装を見ながらなんとか型の実装までやった。
`ND_EXPR_STMT`をリファレンス実装では随分前に導入していたがこのことに今日気づいた。
`ND_EXPR_STMT`のノードをコードにする際にはスタックにプッシュされた分RSPを戻す。
これにより実装がいくらか簡単になる。式の値を使わないような場合にこのノードを使う。
例えば`for(init;cond;inc)`については`cond`は結果を用いるから`ND_EXPR_STMT`は用いず、`init,inc`は結果を用いないから`ND_EXPR_STMT`を使う。`peek`関数を(リファレンス同様)実装した。これは次のトークンが与えられた文字列と一致するかどうかを返す関数だ。9cc/chibiccの特徴としてコードを最初に全部読み込んでメモリに載せること、トークナイズ・パース・コード生成を並列せずに順に行うということを著者が挙げていたと思うが、この関数が自然に実装できるのはこの特徴のおかげだと思う。

<iframe width="560" height="315" src="https://www.youtube.com/embed/MicEimqeNb4?si=1NY5dkufjdqaZYsc" title="YouTube video player" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture; web-share" referrerpolicy="strict-origin-when-cross-origin" allowfullscreen></iframe>
などの動画を見た。