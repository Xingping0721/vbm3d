IMPLEMENTATION OF THE VIDEO DENOISING ALGORITHM VBM3D
=====================================================

* Author    : EHRET Thibaud <ehret.thibaud@gmail.com>
* Copyright : (C) 2018 IPOL Image Processing On Line http://www.ipol.im/
* Licence   : GPL v3+, see gpl.txt




---
---
---





## VBM3D 実行ガイド
### 1. 実行場所の基本ルール
VBM3Dに付属しているシェルスクリプト（*.sh）は、「自分が今いるディレクトリ（./）」にある実行ファイルを呼び出すように書かれています。
そのため、プロジェクトのルートではなく、必ず build/bin/ フォルダの中に移動してからスクリプトを実行する必要があります。

### 2. 実行前の準備
画像シーケンスを入力として使用します。ルートディレクトリ（vbm3d/）に video フォルダを作成し、連番画像を配置してください。

テスト用ダミー画像の生成例（ルートディレクトリで実行）:
```Bash
mkdir -p video
ffmpeg -f lavfi -i testsrc=duration=1:size=640x480:rate=10 video/i%04d.png
```

### 3. 基本のノイズ除去を実行する
バイナリファイル（VBM3Ddenoising）を直接実行します。

実行コマンド:

```Bash
cd build/bin
./VBM3Ddenoising -i ../../video/i%04d.png -f 1 -l 10 -sigma 20
```

- -i : 入力画像のパス（build/bin/ から見た相対パス ../../ を指定）

- -f : 開始フレーム番号

- -l : 終了フレーム番号

- -sigma : ノイズの標準偏差
- 出力先 : build/bin/ 内に bsic_*.png と deno_*.png が生成されます。

### 4. オプティカルフロー付きノイズ除去を実行する（推奨）
動き補償を用いて精度を高めるスクリプトです。内部で複数のプログラム（tvl1flowなど）を順次呼び出します。

実行コマンド:

```Bash
cd build/bin
./VBM3Ddenoising_OF.sh ../../video/i%04d.png 20 ../../deno_OF_%03d.tif 1 10
```
- 引数の順序 : 入力ファイルパス ノイズ標準偏差 出力ファイルパス 開始フレーム 終了フレーム

- 出力先 : 上記の例では、プロジェクトのルートディレクトリ（../../）に deno_OF_001.tif 等が生成されます。

- 注意 : 途中で生成されるオプティカルフローファイル（.flo）は build/bin/ 内に一時的に出力されます。






---
---
---






OVERVIEW
--------

This source code provides an implementation of VBM3D developped in "Dabov, Kostadin,
Alessandro Foi, and Karen Egiazarian. "Video denoising by sparse 3D transform-domain
collaborative filtering." 2007 15th European Signal Processing Conference. IEEE, 2007".

This code is part of an [IPOL](http://www.ipol.im/) publication. Please cite it
if you use this code as part of your research ([link](https://www.ipol.im/pub/art/2021/340/)).

It is based on Marc Lebrun's code for the image denoising version of this algorithm (BM3D)
available on [the BM3D IPOL page](http://www.ipol.im/pub/art/2012/l-bm3d/).
It also uses the TVL1 optical flow from IPOL ([here](http://www.ipol.im/pub/art/2013/26/), available 
in the folder 'tvl1flow') and the multiscale from [here](https://github.com/npd/multiscaler)
(in the folder 'multiscale').

COMPILATION
-----------

The code is compilable on Unix/Linux and hopefully on Mac OS (not tested!). 

**Compilation:** requires the cmake and make programs.

**Dependencies:** FFTW3 and OpenMP [can be disabled]. 
For image i/o we use [Enric Meinhardt's iio](https://github.com/mnhrdt/iio),
which requires libpng, libtiff and libjpeg.
 
Configure and compile the source code using cmake and make.  It is recommended
that you create a folder for building:

UNIX/LINUX/MAC:
```
$ mkdir build; cd build
$ cmake ..
$ make
```

Binaries will be created in `build/bin folder`.

NOTE: By default, the code is compiled with OpenMP multithreaded
parallelization enabled (if your system supports it). 
The code will then use the maximum number of thread available on your machine (up to 32). 
This can be reduced by changing the maximum number of threads allowed in lines 126 and 127 
of `vbm3d.cpp`.

USAGE
-----

The following commands have to be run from the current folder:

List all available options:</br>
```
$ ./VBM3Ddenoising --help
```

While being a video denoising algorithm, the method takes as input the frames of the video 
and not an actual video. The frames can be extracted using ffmpeg on linux. For example: 
```
$ ffmpeg -i video.mp4 video/i%04d.png
```

There is only four mandatory input arguments:
* `-i` the input sequence
* `-f` the index of the first frame
* `-l` the index of the last frame
* `-sigma` the standard deviation of the noise

When providing a sequence that is already noisy the option `-add` should be set to false.

All path should be given using the C standard. For example to reference to the following video:
* video/i0000.png
* video/i0001.png
* video/i0002.png
* video/i0003.png
* video/i0004.png
* video/i0005.png
* video/i0006.png
* video/i0007.png
* video/i0008.png
* video/i0009.png

The command for denoising with a noise standard deviation of 20 should be 
```
$ ./VBM3Ddenoising -i video/i%04d.png -f 0 -l 9 -sigma 20
```

This creates a basic denoised sequence (bsic_*.png) and a final denoised sequence (deno_*.png).
It also create a text file (measure.txt) with the respective PSNRs and RMSEs (when -add is set to True).

The denoising with optical flow can be called using the `VBM3Ddenoising_OF.sh` script:
```
$ ./VBM3Ddenoising_OF.sh video/i%04d.png 20 deno_%03d.tif 1 9
```

Similarly the multiscale denoising can be called using the `VBM3Ddenoising_multiscale.sh` script. The following
command performs a denoising with two scales (DCT version) with the default VBM3D parameters: 
```
$ ./VBM3Ddenoising_multiscale.sh video/i%04d.png 20 test deno_%03d.tif 1 9 "" 2 0
```

Finally the multiscale denoising with optical flow can be called using the `VBM3Ddenoising_OF_multiscale.sh` script:
```
$ ./VBM3Ddenoising_OF_multiscale.sh video/i%04d.png 20 test deno_%03d.tif 1 9 "" 2 0
```
-----

FILES
-----

This project contains the following source files:
```
	main function:            src/main.cpp
	vbm3d implementation:     src/vbm3d.h
	                          src/vbm3d.cpp
	parameters container:     src/parameters.h
	Basis trans. operations:  src/lib_transforms.h
	                          src/lib_transforms.cpp
	command line parsing:     src/Utilities/cmd_option.h
	image i/o:                src/Utilities/iio.h
	                          src/Utilities/iio.c
	image container:          src/Utilities/LibImages.h
	                          src/Utilities/LibImages.cpp
	video container:          src/Utilities/LibVideoT.hpp
	                          src/Utilities/LibVideoT.cpp
	random number generator:  src/Utilities/mt19937ar.h
	                          src/Utilities/mt19937ar.c
```

ABOUT THIS FILE
---------------

Copyright 2018 IPOL Image Processing On Line http://www.ipol.im/

Copying and distribution of this file, with or without modification,
are permitted in any medium without royalty provided the copyright
notice and this notice are preserved.  This file is offered as-is,
without any warranty.
