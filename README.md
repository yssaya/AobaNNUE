# AobaNNUE

AobaNNUEは2025年11月11日時点で、無料で公開されている評価関数としては
振電3([2025年2月21日公開](https://x.com/tayayan_ts/status/1892705760175665396))を抜いて最強だと思います。
振電3より +30 ELO 程度強いです。振電3付属のWindows版だと +67 ELO強いです。

ソースは[やねうら王V9.00 GitHub版](https://github.com/yaneurao/YaneuraOu)とほぼ同一です。

# 実行ファイル
[Windows用の実行ファイル(64bit版のみです)](https://github.com/yssaya/aobannue/releases)
```
  AobaNNUE_AVX2.exe  ... 最近のPCなら動作します。まずこれをお試しください。
  AobaNNUE_ZEN2.exe  ... AMDの最近のCPUなら、こちらが少し速い、かも。
  AobaNNUE_SSE42.exe ... ほぼ全てのPCで動作します。AVX2より20%ほど遅いです。
```

# ソースの変更点
  ソースはやねうら王V9.00 GitHub版を若干変更したものです。  
    [https://github.com/yaneurao/YaneuraOu/releases/tag/V9.00](https://github.com/yaneurao/YaneuraOu/releases/tag/V9.00)
  - 標準NNUE(HalfKP_256x2_32_32)ではなく HalfKP_768x2_16_64 を採用。
  - FV_SCALE のデフォルトを16からを40に。
  - 起動オプションを有効に。
  - "setoption name ConstantThinkingTime value 1000" で1手1秒で指すオプションを追加。

# 評価関数の作成
  学習させた局面、は人間の知識が含まれていますが、そこにつけた評価値は
  人間の知識を一切使っていない、ゼロからAIが見つけた評価値が使われています。

  nnue-pytorchを使っています。  
  野田さんのshogi_hao_depth9の80億局面のデータを
  2025年選手権版のAobaZeroの0手読みの評価値で上書きしたデータで学習させています。
  選手権版はAobaZeroとAoba振り飛車の棋譜を使って学習させたものです。  
  haoのデータは静止探索で動く局面は書き換えています。  
  勝率を評価値に変換(ponanza係数)するのに dlshogi では 756 で変換していたのを 600 に。600の方が +40 ELOほど強いです。
  
  epoch-size=10000000,(minibatch=8192), を32000エポック(3200億局面)を学習、RTX 4090 1枚で15日間  
  1024_8_96, 512_8_64, 2048_32_32 を試しましたが 768_x2_16_64 が同一時間で一番いい結果でした。  
  2048_32_32 は2.6倍npsが振電3より遅かったです。  
  以下のコマンドで学習させてます。学習率は27000エポックから2000ステップごとに半減
```
  $ python3 train.py --features "HalfKP" --max_epochs 1000000 --default_root_dir logs/20250930 --lr 0.5 0.05 --num-workers 8 --lambda 1.0 0.5 --label-smoothing-eps 0.001 --accelerator gpu --devices 1 --score-scaling 511 --min-newbob-scale 1e-5 --num-epochs-to-adjust-lr 500 --momentum 0.9 --network-save-period 100 --resume-from-model "" ./shogi_hao_depth9/8_seq_q_aoba_a600.bin ./shogi_hao_depth9/5340981_126_shuffle_q_aoba.bin --enable_progress_bar False
```
  詳細はこのあたりを。

  無料では最強と思われるAobaNNUEを公開しました  
  [http://www.yss-aya.com/bbs/patio.cgi?read=210&ukey=0](http://www.yss-aya.com/bbs/patio.cgi?read=210&ukey=0)  
  NNUEの学習をAobaZeroの評価値で試しています  
  [http://www.yss-aya.com/bbs/patio.cgi?read=195&ukey=1](http://www.yss-aya.com/bbs/patio.cgi?read=195&ukey=1)  
  AobaZeroの2025年の[アピール文書](https://www.apply.computer-shogi.org/wcsc35/appeal/AobaZero/2025csa_appeal.pdf)    
  評価値書き換えに使った[スクリプト](https://github.com/yssaya/cshogi_aoba/tree/main/psv_shuffle)  
  山岡さんの元の[スクリプト](https://github.com/TadaoYamaoka/DeepLearningShogi/blob/master/dlshogi/utils/hcpe_re_eval.py)  

# Windowの実行ファイルのビルド
  Windows用の実行ファイルはMSYS2 CLANG64でPGO(プロファイル最適化)を行っています。
```
  $ make pgo
    default.profraw を default.profdata に変換
  $ llvm-profdata merge -output=default.profdata default.profraw
    再度、
  $ make profuse
```

# 棋力の検証
  **振電3に付属のWindows用バイナリでの比較**  
9.00gitの方が7.70kaiより同一時間では強いので振電3に不利な条件での比較です。  
1000局で勝率0.5955、+67 ELO でした。

     AobaNNUE/YO9.0/git の Shinden3/YO7.70kai 対する勝率とELO
      575勝 41分 384敗 1000対局   勝率 0.5955  ELO ( +67)

    Windows11 Home, Core i7-1165G7, 4スレッド
    ShogiHomeで1手2秒設定(実質1秒程度で指します)。最大513手まで。
    NPSは初期局面で
       AobaNNUE 207万/秒
       振電3    240万/秒
    互角局面集(2016年)のsfenで24手目まで指定局面。1つを先後入れ替えで500局面利用
    共にAVX2のバイナリを使用。FV_SCALEはともに40。引き分けは0.5勝扱い。

  **Linux(ubuntu 24.04)での比較**  
    同条件で、思考時間を変えると +23 から +44 ELO 強いです。  
    振電3を7.70で動かすと+68とほぼWindows版と同じ結果でした。  
    100万ノード固定では+78でした。もっと差がでるかと思いましたが。

    AobaNNUE の 振電3 対する勝率とELO
       勝 分   敗 局数(宣 千 宣)  勝率  ELO
    1248-72-1080 2400 (0-72-5) 0.535(  24) 0.1秒/手(約  75万/手) 
     449-58- 293  800 (1-58-2) 0.598(  68) 0.1秒/手, 振電3は7.70kaiのソース 
     479-20- 301  800 (0-20-2) 0.611(  78) 1手100万ノード固定
     416-23- 361  800 (0-23-2) 0.534(  23) 0.2秒/手(約 151万/手)
    1245-87-1068 2400 (2-86-4) 0.537(  25) 0.8秒/手(約 367万/手), Ryzen 7 5700X, 6 スレッド
     435-31- 334  800 (0-31-0) 0.563(  44) 2.0秒/手(約1512万/手) 

  表記がないものは以下の条件です。  
  Ryzen 9 7900 (物理12コア), 共に9.00GitHub をgccでZEN3でビルド、pgoなし。  
  FV_SCALEはAobaNNUE,振電3 ともに40、共に8スレッド  
  すべて互角局面集を使って24手目から対戦開始。先後を入れ替え。800局だと400局面を利用  
  [互角局面集(2016年)](https://yaneuraou.yaneu.com/2016/08/24/)  

