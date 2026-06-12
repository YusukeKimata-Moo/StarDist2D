# StarDist 2D Colab Notebook

Google Colab で 2D 蛍光 TIFF 画像を StarDist の学習済みモデル `2D_versatile_fluo` によりインスタンスセグメンテーションするための notebook です。

## 概要

このリポジトリには、Google Drive 上の 2D TIFF 画像を読み込み、StarDist 2D で物体ごとのラベル画像、全物体をまとめた二値マスク、オーバーレイ画像、簡単な測定 CSV を出力する Colab notebook が含まれています。

対象は、核などの単一チャンネル蛍光 intensity 画像です。RGB brightfield 画像や 3D stack TIFF は対象外です。

## ファイル

- `StarDist_2D_versatile_fluo_colab.ipynb`: Colab で実行するメイン notebook
- `260611_2.tif`: 動作確認用のサンプル TIFF
- `260611_2_med4.tif`, `260611_2_gau2.tif`: 前処理比較用のサンプル TIFF

## 使い方

1. このリポジトリを開き、`StarDist_2D_versatile_fluo_colab.ipynb` を Google Colab で開きます。
2. Colab の `ランタイム` > `ランタイムのタイプを変更` で、ハードウェア アクセラレータに `GPU` を選択します。
3. notebook のセルを上から順に実行します。
4. 初回のパッケージインストール後に Colab で「セッションがクラッシュしました」と表示されることがあります。依存関係を反映するための再起動なので、再接続後に次のセルへ進んでください。
5. `input_tiff_path` と `output_dir` を自分の Google Drive 上のパスに変更して実行します。

## 入力画像

- 2D TIFF 画像を想定しています。
- `uint8`, `uint16`, `float` 画像を読み込み、推論前に intensity を正規化します。
- multi-channel 画像の場合は、`use_channel_selection=True` と `channel_index` で使用するチャンネルを選択できます。
- 3D TIFF stack はこの notebook では直接扱いません。

## 出力

指定した `output_dir` に以下を保存します。

- ラベル TIFF: インスタンスごとに異なる整数ラベルを持つ画像
- 二値マスク TIFF: 全インスタンスを 1 つにまとめたマスク画像
- オーバーレイ PNG: 入力画像とラベル境界を重ねた確認用画像
- 測定 CSV: 各インスタンスの `label_id`, `area_px`, `centroid_y`, `centroid_x`

## 主なパラメータ

| パラメータ               | 意味                                                    |
| ------------------------ | ------------------------------------------------------- |
| `input_tiff_path`        | 入力する 2D TIFF 画像のパス                             |
| `output_dir`             | 結果を保存するフォルダ                                  |
| `percentile_low`         | intensity 正規化で下限として使う percentile             |
| `percentile_high`        | intensity 正規化で上限として使う percentile             |
| `use_custom_thresholds`  | `True` にすると `prob_thresh` / `nms_thresh` を手動指定 |
| `prob_thresh`            | 検出の信頼度しきい値。低いほど検出が増えやすい          |
| `nms_thresh`             | 重なった候補を統合するしきい値                          |
| `n_tiles_y`, `n_tiles_x` | 大きな画像でメモリ不足になる場合のタイル分割数          |

## 調整の目安

- 検出漏れが多い場合: `use_custom_thresholds=True` にして `prob_thresh` を下げます。
- 過検出が多い場合: `prob_thresh` を上げます。
- 近接した物体の分離が悪い場合: `nms_thresh` や正規化 percentile を調整します。
- 大きい物体と小さい物体が混在する場合: `prob_thresh` をやや低めにし、結果を目視確認します。
- 輪郭だけ明るく内部が暗い核は、`2D_versatile_fluo` が苦手な場合があります。`prob_thresh` を下げ、必要に応じて軽い前処理と比較してください。

## 注意点

StarDist の `2D_versatile_fluo` は、2D 蛍光核画像向けの学習済みモデルです。撮影対象、pixel size、染色、コントラストが学習データと大きく異なる場合は、パラメータ調整だけでは十分に改善しないことがあります。

結果は必ずオーバーレイ画像で確認してください。研究用途で定量解析に使う場合は、代表画像で目視検証し、必要に応じて手動アノテーションや別条件との比較を行ってください。

## 引用

この notebook を研究・発表・論文で使用する場合は、StarDist 2D の原著論文と公式リポジトリを引用してください。

- Schmidt U, Weigert M, Broaddus C, Myers G. Cell Detection with Star-Convex Polygons. MICCAI 2018. DOI: https://doi.org/10.1007/978-3-030-00934-2_30
- StarDist 公式リポジトリ: https://github.com/stardist/stardist
- StarDist 公式サイト: https://stardist.net/
