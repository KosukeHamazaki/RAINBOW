各ファイルの説明

1.1._Sample_codes_for_RAINBOW_GWAS_real.R : RAINBOWのサンプルコード（実データ）
geno_and_pheno_data.RData : 実データ、サンプルコードで読み込む
RAINBOW_0.1.0.tar.gz : RAINBOWパッケージのsource file


はじめに、
RAINBOW_0.1.0.tar.gz
を適当なディレクトリに保存して、Rで
install.packages("保存場所/RAINBOW_0.1.0.tar.gz", repos = NULL, type = "source")
を実行してください。
あとは、
library(RAINBOW) or require(RAINBOW)
で、RAINBOWの関数を使えるようになります。

初めて使う方は、Rで
RGWAS.menu()
を実行すればどのようなGWAS関数を使えば良いかコンソールに表示されます。

あとは、RAIBOWのサンプルコードを参考に各関数がそういう時に使えるか見てみてください。

ちなみに、パッケージ化されているので、
?関数名
でヘルプも出ます。
詳しい説明はまだ書いていないですが、引数の説明は一応書いてあります。


それから、GWAS以外の関数としてはmixed modelを解く関数をいくつか用意してあります。(rrBLUPのmixed.solveに対応)
Kernel matrixが一つの場合は
EMM.cpp()
複数カーネルの場合は
EM3.cpp()
を利用してください。

C++で実装しているので多少は早いはずです。

何かわからないことがあれば濱崎まで。