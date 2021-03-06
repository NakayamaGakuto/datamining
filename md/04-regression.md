
# 回帰2

## ワイン価格の予測

|ラベル|意味|
|--|--|
|OBS|通し番号|
|VINT|生産年|
|LPRICE2|ワインの相対価格（基準は1961年）の対数|
|WRAIN|10月から3月までの降水量（ml）|
|DEGREES|4月から9月までの気温の平均（摂氏）|
|HRAIN|収穫期（8月から9月）の降水量（ml）|
|TIME_SV|1983年基準でのワインの熟成年数|


```r
library(tidyverse)
library(caret)
tmp <- read.table(file = "http://www.liquidasset.com/winedata.html", # 読み込む対象
                  header = TRUE,                                     # 1行目は変数名
                  na.string = '.',                                   # 欠損値を表す文字列
                  skip = 62,                                         # 読み飛ばす行数
                  nrows = 38)                                        # 読み込む行数
# print.data.frame(tmp) # データをすべて表示する。
head(tmp)               # データの先頭部分を表示する。
```

```r
psych::describe(tmp) # 欠損値の有無，基本統計量の確認
```

```r
my_data <- na.omit(tmp) # 欠損値のあるデータの削除
head(my_data)           # 結果の確認
```

```r
my_data %>% write_csv("wine.csv") # CSV形式で保存する。
```

## 重回帰分析

キーワード：線形重回帰分析

```r
set.seed(0)
```

```r
library(tidyverse)
library(caret)
my_data <- read_csv("wine.csv")
my_result <- train(form = LPRICE2 ~ WRAIN + DEGREES + HRAIN + TIME_SV, # モデル式
                   data = my_data,                                     # データ
                   method = "lm")                                      # 手法（線形回帰分析）
```

```r
my_result$finalModel$coefficients # 線形重回帰分析で得られた係数
```

```r
my_test <- data.frame(WRAIN = 500, DEGREES = 17, HRAIN = 120, TIME_SV = 2) # 予測対象
my_result %>% predict(my_test)                                             # 予測
```

```r
my_result$results # 検証RMSE
```

## 入力変数の数とRMSEの関係

```r
set.seed(0)
```

```r
library(tidyverse)
library(caret)
my_data <- read_csv("wine.csv")
my_result1 <- train(form = LPRICE2 ~ WRAIN + DEGREES + HRAIN + TIME_SV,
                    data = my_data,
                    method = "lm")
my_pred <- my_result1 %>% predict(my_data) # 訓練データの再現
RMSE(my_pred, my_data$LPRICE2)             # 訓練RMSE
```

```r
set.seed(0)
```

```r
my_data2 <- my_data       # データをコピーする。
n <- nrow(my_data)        # データの件数
my_data2$rand1 <- runif(n) # 変数rand1の値は乱数で決める。
head(my_data2)
```

```r
set.seed(0)
```

```r
my_result2 <- train(form = LPRICE2 ~ WRAIN + DEGREES + HRAIN + TIME_SV + rand1, # rand1を追加
                    data = my_data2,
                    method = "lm")
my_prediction2 <- my_result2 %>% predict(my_data2) # 訓練データの再現
RMSE(my_prediction2, my_data$LPRICE2)              # 訓練RMSE
```

```r
c(my_result1$result$RMSE, # 検証RMSE（rand1なしのモデル）
  my_result2$result$RMSE) # 検証RMSE（rand1ありのモデル）
```

## 入力変数が複数であることへの対応

キーワード：部分集合選択

キーワード：変数選択

キーワード：正則化

キーワード：Lasso，Ridge回帰

キーワード：Elastic Net

キーワード：次元削減

キーワード：主成分回帰，部分最小2乗法

## 部分集合選択

キーワード：ステップワイズ法

|手法|説明|
|--|--|
|**変数増加法**|入力変数0個のモデルからスタートし，予測の役に立ちそうな変数を一つずつ追加する。|
|**変数減少法**|すべての入力変数を使うモデルからスタートし，予測の役に立たなそうな変数を一つずつ除去する。|
|**変数増減法**|入力変数0個のモデルからスタートし，予測の役に立ちそうな変数の追加と，予測の役に立たなそうな変数の除去を繰り返す。|


キーワード：変数増加法，変数減少法，変数増減法

```r
set.seed(3)
```

```r
library(tidyverse)
library(caret)
my_data <- read_csv("wine.csv")
my_data2 <- my_data        # データをコピーする。
n <- nrow(my_data)         # データの件数
my_data2$rand1 <- runif(n) # 変数rand1の値は乱数で決める。
my_data2$rand2 <- runif(n) # 変数rand1の値は乱数で決める。
```

```r
my_result <- train(form = LPRICE2 ~ WRAIN + DEGREES + HRAIN + TIME_SV + rand1 + rand2,
                   data = my_data2,
                   method = "leapForward",             # 変数増加法
                   tuneGrid = data.frame(nvmax = 1:6)) # 変数の最大数として1から6を試す。
my_result$results
```

```r
summary(my_result$finalModel) # モデルの概要
```

```r
coef(my_result$finalModel, id = 4) # 4番目の結果の係数
```

## 正則化（Lasso，Ridge回帰，Elastic Net回帰）

```r
set.seed(0)
```

```r
library(tidyverse)
library(caret)
my_data <- read_csv("wine.csv")
my_result <- train(form = LPRICE2 ~ WRAIN + DEGREES + HRAIN + TIME_SV,
                   data = my_data,
                   method = "glmnet")           # Elastic Net回帰
my_result$results %>% filter(RMSE == min(RMSE)) # 検証RMSEの最小値
```

```r
set.seed(0)
```

```r
my_result <- train(form = LPRICE2 ~ WRAIN + DEGREES + HRAIN + TIME_SV,
                   data = my_data, method = "glmnet",
                   tuneGrid = expand.grid(alpha  = seq(0, 1,   0.05),   # alphaの探索範囲
                                          lambda = seq(0, 0.1, 0.005)), # lambdaの探索範囲
                   trControl = trainControl(method  = "repeatedcv",     # 反復型K分割交差検証
                                            number  = 5,                # K = 5
                                            repeats = 20))              # 反復は20回
my_result$results %>% filter(RMSE == min(RMSE)) # 検証RMSEの最小値
```

```r
coef(my_result$finalModel, my_result$bestTune$lambda) # 検証RMSEが最小になるときの係数
```

```r
ggplot(data = my_result$results,                    # 検証RMSEの図示
       mapping = aes(x = alpha,                     # x軸はalpha
                     y = lambda,                    # y軸はlambda
                     z = RMSE)) +                   # z軸（色）は検証RMSE
  geom_raster(aes(fill = RMSE)) +                   # 検証RMSEの値によって塗り分ける
  scale_fill_gradientn(colors = c("blue", "red")) + # 値が小さいと青，大きいと赤
  geom_contour(color = "white")                     # 等高線（白）
```

**練習：ワインのデータに対してこれまでに紹介した手法を適用し，検証RMSEを求めてください。K最近傍法とニューラルネットワークではデータを標準化した以外は，`train()`をそのまま使った結果の一例を示します。**

|手法|検証RMSE|
|--|--|
|線形重回帰分析（`lm`，[@]）|0.348|
|部分集合選択（`leapForward`，`leapBackward`，`leapSeq`，[@]）|0.348|
|Elastic Net（`glmnet`，[@]）|0.336|
|k最近傍法（`knn`，[@]）|0.495|
|ニューラルネットワーク（`nnet`，[@]）|1.588|
|ランダムフォレスト（`rf`，[@]）|0.484|
|ブースティング（`xgbTree`，[@]）|0.466|


## 住宅価格の予測（Kaggleに挑戦）

```r
library(tidyverse)
library(caret)
my_train <- read_csv("data/house-prices-advanced-regression-techniques/train.csv")
my_test  <- read_csv("data/house-prices-advanced-regression-techniques/test.csv")
```

```r
dim(my_train) # データの件数と変数の数
```

```r
psych::describe(my_train) %>% head(5) # 基本統計量（最初の5行）
```

```r
set.seed(0)
```

```r
my_result <- train(form = SalePrice ~ .,   # すべての入力変数を使う。
                   data = my_train,        # 欠損のあるデータ
                   method = "xgbTree",     # ブースティング
                   na.action = na.pass)    # 欠損があっても止めない。
my_result$results %>% select(RMSE) %>% min # 検証RMSEの最小値
```

```r
set.seed(0)
```

```r
my_result <- train(form = SalePrice ~ .,
                   data = my_train,
                   method = "lm",                  # 線形回帰分析
                   na.action = na.pass,            # 欠損があっても止めない。
                   preProcess = c("medianImpute")) # 欠損値を中央値で代替する。
my_result$results %>% select(RMSE) %>% min
```

```r
my_pred <- my_result %>% predict(my_test,
                                 na.action = na.pass) # 欠損があっても止めない。
```

```r
my_submission <- data.frame(Id = my_test$Id,
                            SalePrice = my_pred) # データフレームの作成
head(my_submission)
```

```r
my_submission %>% write_csv("houseprice_submission.csv") # CSV形式で保存する。
```
