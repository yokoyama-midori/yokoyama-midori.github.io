---
title: Ankiのエスペラント/英語の共用デッキをエスペラント/日本語にカスタマイズする
date: 2025-11-09 10:00 0900
categories: [エスペラント]
tags: [Anki]
# math : true
---

## はじめに
[esperanto.cards](https://esperanto.cards/)や[ankiweb上の共用デッキ](https://ankiweb.net/shared/info/353142617)にてEsperanto cards (for English speakers)というデッキが公開されています。これはAnkiというアプリで使える英語話者用のエスペラントの単語帳なんですが、2500語収録されており（EO→ENとEN→EOで5000枚）音声や例文等も充実していて結構作り込まれているようです。このデッキを日本語話者向けにカスタマイズしたいというのがこの記事の内容です。

AnkiはWindows用のものを用い、作業はWSL2上で行います。

## データの作成
まず、共用デッキから`Esperanto_cards_for_English_speakers.apkg`をダウンロードします。そしてAnkiからファイル→インポートからファイルを選んでインポートします。次にファイル→エクスポートを選び、ファイルの形式：テキストファイル形式のノート(.txt)、追加オプションでEsperanto cards (for English speakers)を選びエクスポートします。チェックは上の2つのみ選びます。こうして得られるファイルはタブで区切られたテキストデータになっています。各行について、エスペラントのカラムを選び日本語訳を探し新しいカラムとして追加するということをします。そのためには辞書が必要ですが、実用エスペラント小辞典という辞書が配布されているので[ここ](https://www.vastalto.com/jpn/#e-Dic)からダウンロードします。解凍すると`pejvo.txt`が
```
a:{Ｏ}［文字名］アー（文字A，aの名称）
-a:{Ｂ}［形容詞語尾］
abak/o:【数】計算盤,そろばん,>>sorobano;【建】（円柱頭部の）冠板（かむりいた）
abat/ec/o:修道院長の職
```
のような感じになっています。スラッシュが邪魔なので、スラッシュを除いたエスペラント、タブ文字、元の行を並べたファイルを作ります。生成AIに次のようなスクリプトを書いてもらいました。
```sh
#!/bin/bash

# This script pre-processes the 'pejvo.txt' dictionary file
# to create a lookup table for faster and more robust searching.
# It removes all '/' characters from the Esperanto word part and
# stores it along with the original dictionary line.

DICT_FILE="pejvo.txt"
LOOKUP_OUTPUT_FILE="pejvo_lookup.tsv"

echo "Creating lookup table from ${DICT_FILE}..."

# Use awk for efficient processing of the entire file.
# -F: sets the input field separator to ':'
# key=$1 takes the part before the first colon.
# gsub("/", "", key) removes all '/' from the key.
# print key"\t"$0 outputs the modified key, a tab, and the original full line.
awk -F: '
{
    # Skip lines that do not contain a colon, as they are not valid dictionary entries.
    if (NF < 2) {
        next
    }
    key = $1
    gsub("/", "", key) # Remove all slashes from the key part
    print key"\t"$0    # Output the plain key, a tab, and the original dictionary line
}' "$DICT_FILE" > "$LOOKUP_OUTPUT_FILE"

echo "Lookup table created: ${LOOKUP_OUTPUT_FILE}"

```
{: file="create_lookup_table.sh" }

```terminal
./create_lookup_table.sh
```
を実行し、`pejvo_lookup.tsv`が作られていることを確認します。これを使ってカラムに日本語を追加していきます。次のスクリプト
```shell
#!/bin/bash

# This script reads an Esperanto-English vocabulary file, looks up the
# Japanese translation for each Esperanto word from a pre-processed dictionary lookup file,
# and outputs a new file with the Japanese translation appended as a new column.

# Input and output file paths
CARD_FILE="Esperanto cards (for English speakers).txt"
OUTPUT_FILE="cards_ja.txt"
LOOKUP_FILE="pejvo_lookup.tsv"

echo "Processing card file and appending Japanese meanings..."

# Ensure the output file is empty before starting the main processing
> "$OUTPUT_FILE"

successful_lookups=0

# Phase 2: Process the card file using the lookup table.
while IFS= read -r line || [[ -n "$line" ]]; do
    # Skip empty lines
    if [ -z "$line" ]; then
        continue
    fi

    # Extract the Esperanto word from the first column (tab-separated)
    esperanto_word=$(echo "$line" | cut -f1)

    # Search for the plain word in our lookup file.
    # Match the beginning of the line and the tab to ensure whole-word matching.
    match=$(grep "^${esperanto_word}"$'	' "$LOOKUP_FILE" | head -n 1)

    japanese_meaning=""
    if [ -n "$match" ]; then
        # If a match is found, extract the original dictionary line (the part after the first tab)
        original_dict_line=$(echo "$match" | cut -f2-)

        # From that original line, extract the Japanese part, which is after the ':' character.
        japanese_meaning=$(echo "$original_dict_line" | sed 's/^[^:]*://')
        successful_lookups=$((successful_lookups + 1))
    else
        # If no match is found, use the 6th column as a fallback.
        japanese_meaning=$(echo "$line" | cut -f3)
    fi

    # Print the original card line, a tab separator, and the found Japanese meaning.
    echo -e "${line}\t${japanese_meaning}" >> "$OUTPUT_FILE"

done < "$CARD_FILE"

echo "Processing complete. Output written to ${OUTPUT_FILE}"
echo "Found ${successful_lookups} Japanese translations."
```
{: file="add_japanese.sh" }
を実行します。日本語訳が見つからなかった場合は3番目のカラムにある英語訳を使うようにしています。2500個中2366個は日本語訳が見つかりました。`cards_ja.txt`が作られたことを確認します。

## Ankiへの追加
まずツール→ノートタイプを管理→Esperantoを選びフィールド→追加をクリックしフィールド名としてJapaneseと入力します。次にカードを選びテンプレートで\{\{English\}\}となっている部分を\{\{Japanese\}\}に書き換えます。カードタイプが2種類あるので両方行います。
次にファイル→インポートから`cards_ja.txt`を選びます。インポートのオプションとして既存のノート：更新としてインポートします。このときフィールドの割り当てでJapaneseの欄に日本語訳が来るように選びます。

## 追加の設定
右下に`rate_review`などのアイコンが出るはずですが、僕の環境ではテキストになってしまっていました。テンプレートに
```
<link href="https://fonts.googleapis.com/icon?family=Material+Icons"
      rel="stylesheet">
```
を追加することでアイコンが見えるようになりました。

エスペラントの意味を答えるカードとエスペラントを答えるカードの2種類のカードがあります。僕は前者をメインでやりたかったのでデッキを分けました。デッキを新たに作っておき、ブラウザの検索窓からcard:EOを検索しctrl+Aで全選択→デッキを変更から設定しました。こういうことをすると1つのノートから2つのカードが（別のデッキに）できていることになるんですが、ここらへんの仕様がどうなってるのかあんまり分からないです。ノート自体はデッキには属さないのかな？追加をする場合ノートとデッキを選ぶけどノートが複数のデッキを生成する場合個別にデッキは選べないのかな？