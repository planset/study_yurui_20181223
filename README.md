# ゆるい勉強会 2018/12/23 のメモとデモ

# 1. OpenCV.jsの概要

## OpenCV.jsとは？
まず、OpenCV はオープンソースの画像解析ライブラリです。
画像処理、モーション解析、パターン認識、機械学習、ビデオ入出力など
画像や映像の解析を簡単に行うためのライブラリです。

OpenCV.jsは、そのOpenCVのJavaScript版です。

ただし、JavaScriptで書かれたものではなく、
OpenCVをasm.jやWebAssemblyで動作するようにEmscriptenを使ってコンパイルしたものです。


### asm.jsとは
最近は割と早くなっている気もしますが、昔からJavaScriptの処理速度は
ネイティブアプリに比べると非常に遅いものでした。
そこで、JavaScriptの処理速度の高速化を目指して生まれたのがasm.jsです。
asm.jsはJavaScriptに制約を加えた書き方をすることで静的な型付けを行い、高速化しています。
基本的にはJavaScriptなので手で書くこともできますが、C言語やC++から生成することもできます。
このとき、C言語やC++からJavaScriptを生成するのに使われている技術がLLVM や Emscripten になります。

asm.jsは十分に高速でネイティブアプリと比べてもかなりの速度になったみたいでしょうが、
根本的にはJavaScriptであり、それ以上のことができません。
そのため、おそらく現在はssm.jsよりもWebAssemblyが主流になってきています。


### WebAssembly
WebAssemblyはブラウザで実行可能なバイナリフォーマットです。
各ブラウザで既にサポートされていますが、今回試した限り、少し古めのiOSのsafariでは動作しませんでした。

静的な型付けをすることで高速化を図っているのはasm.jsと同様です。
しかし、asm.jsがJavaScriptの枠を超えなかったがために実現できなかったことが、
WebAssemblyはJavaScriptには縛られないためできるようになっています。
例えば、６４ビットの整数を扱うことができるらしいですが、詳しくは知りません。

asm.jsと違ってWebAssemblyはjsではないので手で書くことはできません。
C言語やC++のソースをコンパイルしてバイナリファイル（.wasm）を生成して使います。
asm.jsと同じく、Emscriptenを使います。


### LLVM
「LLVM とは、コンパイル時、リンク時、実行時などあらゆる時点でプログラムを最適化するよう設計された、任意のプログラミング言語に対応可能なコンパイラ基盤である。」wikipedia https://ja.wikipedia.org/wiki/LLVM

コンパイラ基盤・・・ってなんぞや、って感じです。
大雑把な理解しかしていませんが次のような感じ。

ソースコード＝＞（ソース解析）＝＞LLVM IR＝＞（最適化）＝＞LLVM IR＝＞（変換）＝＞機械語

ソース解析までは、フロントエンド側の処理
そこからがLLVMの処理で、最適化まで行ったあと、
バイナリを生成します。

LLVMを利用したコンパイラで有名なところはiOS開発のXcodeで使われるClangです。
昔は別だったみたいですが、今はLLVMと一緒に提供されています。


### Emscripten
C言語やC++をLLVMビットコードにしてasm.jsやWebAssemblyにコンパイルするツールです。

emscriptenおドキュメントを見る限り、

Emscriptenでは、emcc(Emscripten Compiler Frontend)を使ってC/C++をjsに変換します。
emcssでは、Clangを使って、C/C++ファイルをLLVMビットコードに変換し、 Binaryenを使ってLLVMビットコードをWebAssemblyに変換します。


# 2. OpenCV.jsの作り方

## OpenCV.jsの作り方？

* え？　ダウンロードできないの？
* 少なくとも公式ページにはない。
* 作り方は書いてある。

## 

1. 必要なライブラリをインストール
2. Emscriptenのインストール
3. OpenCV.jsのビルド

https://docs.opencv.org/4.0.0/d4/da1/tutorial_js_setup.html


手元にあったUbuntu 16.04.2 LTSで。

### 1. 必要なライブラリをインストール

```
sudo apt-get install python2.7
sudo apt-get install nodejs
sudo apt-get install cmake
```

### 2. Emscriptenのインストール

```
mkdir opencv
cd opencv
git clone https://github.com/juj/emsdk.git
cd emsdk
git pull
./emsdk install latest
./emsdk activate latest
source ./emsdk_env.sh
cd ..
```


### 3. OpenCV.jsのビルド
結構時間がかかります。今回はそれぞれ10分以上。
masterブランチの最新の状態でビルドしています。

```
git clone https://github.com/opencv/opencv.git
cd opencv
```


WebAssembly版のビルド
```
python ./platforms/js/build_js.py build_wasm --build_wasm
```

build_wasm/bin にビルドされたファイルができる。
opencv.js, opencv_js.jsができる。
(以前はopencv_js.wasmというファイルができていたのだが、opencv.jsに含まれるようになっていた。)



asm.js版？のビルド
```
python ./platforms/js/build_js.py build_js --disable_wasm
```

--build_wasmをつけてもつけなくても、中身を見るとwasm_binaryがあるようにみえよくわからない。
ビルドオプションを見る限りは --build_wasmをつけたときが WebAssemblyで、つけないときが、asm.js、--disable_wasmをつけたときは



# 参考

## asm.js, WebAssembly
* 詳説WebAssembly https://www.slideshare.net/llamerada-jp/webassembly-75175349
* asm.jsの基本的な使い方・まとめ http://defghi1977.html.xdomain.jp/tech/asmjs/asmjs.htm
* なぜWebAssemblyはasm.jsより速いのか - Qiita https://qiita.com/chikoski/items/cd59df64b4d820125864

## emscripten
* About Emscripten — Emscripten 1.38.21 documentation https://kripken.github.io/emscripten-site/docs/introducing_emscripten/about_emscripten.html
* Building to WebAssembly — Emscripten 1.38.21 documentation https://kripken.github.io/emscripten-site/docs/compiling/WebAssembly.html?highlight=webassembly
* WebAssembly/binaryen: Compiler infrastructure and toolchain library for WebAssembly, in C++ https://github.com/WebAssembly/binaryen


## LLVM
* The Architecture of Open Source Applications: LLVM http://www.aosabook.org/en/llvm.html

## OpenCV, OpenCV.js
* OpenCV library https://opencv.org/
* OpenCV: OpenCV.js Tutorials https://docs.opencv.org/4.0.0/d5/d10/tutorial_js_root.html
* OpenCVをEmscriptenでwasmにビルドする！Webブラウザから呼び出す。OpenCV.js | 404 Motivation Not Found https://tech-blog.s-yoshiki.com/2018/09/544/
* OpenCV.js をちょっとだけ試してみた。 - Qiita https://qiita.com/hon_no_mushi/items/0edb47b46f827f9fca26
* Hough変換による画像からの直線や円の検出：CodeZine（コードジン） https://codezine.jp/article/detail/153


