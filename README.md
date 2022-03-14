# BigQuery-ML

## 概要
BigQueryMLを使用して、Google Analyticsで収集されたセッションデータをもとに、未来の訪問ユーザー数を予想・可視化する。

<img width="957" alt="スクリーンショット 2022-03-14 22 55 18" src="https://user-images.githubusercontent.com/55085752/158186646-10276be8-18ad-4958-a74b-7670ab0d0c19.png">


## 手順
### データセットの作成
MLモデルを保存するデータセット「bqml_test」を「US」リージョンに作成する。
### モデルの作成
CREATE MODEL句で、モデルを作成します。
モデルには、時系列分析でよく使われるARIMAモデルを使用します。
```
CREATE OR REPLACE MODEL bqml_test.ga_arima_model
OPTIONS
  (model_type = 'ARIMA_PLUS',
   time_series_timestamp_col = 'parsed_date',
   time_series_data_col = 'total_visits',
   auto_arima = TRUE,
   data_frequency = 'AUTO_FREQUENCY',
   decompose_time_series = TRUE
  ) AS
SELECT
  PARSE_TIMESTAMP("%Y%m%d", date) AS parsed_date,
  SUM(totals.visits) AS total_visits
FROM
  `bigquery-public-data.google_analytics_sample.ga_sessions_*`
GROUP BY date
```
### モデルを評価
ML.ARIMA_EVALUATE句で、モデルの評価指標を確認できます。
```
SELECT
 *
FROM
 ML.ARIMA_EVALUATE(MODEL bqml_test.ga_arima_model)
```
### 未来の訪問ユーザー数を予想
```
SELECT
 *
FROM
 ML.FORECAST(MODEL bqml_test.ga_arima_model,
             STRUCT(30 AS horizon, 0.8 AS confidence_level))
```
### 予測を可視化
#### 次のクエリは、UNION句を使って、過去データと予測データを結合します
```
SELECT
 history_timestamp AS timestamp,
 history_value,
 NULL AS forecast_value,
 NULL AS prediction_interval_lower_bound,
 NULL AS prediction_interval_upper_bound
FROM
 (
   SELECT
     PARSE_TIMESTAMP("%Y%m%d", date) AS history_timestamp,
     SUM(totals.visits) AS history_value
   FROM
     `bigquery-public-data.google_analytics_sample.ga_sessions_*`
   GROUP BY date
   ORDER BY date ASC
 )
UNION ALL
SELECT
 forecast_timestamp AS timestamp,
 NULL AS history_value,
 forecast_value,
 prediction_interval_lower_bound,
 prediction_interval_upper_bound
FROM
 ML.FORECAST(MODEL bqml_test.ga_arima_model,
             STRUCT(30 AS horizon, 0.8 AS confidence_level))
```
#### 結合結果をデータポータルで可視化する
1. クエリが完了したら、「データを探索」ボタンをクリックし、「データポータルで調べる」をクリックする。
2. グラフの種類に、「時系列グラフ」を選択する。
3. 指標として、「history_value」、「forecast_value」、「prediction_interval_lower_bound」、「prediction_interval_upper_bound」を追加します。
4. 「スタイル」の「欠落データ」を、「線を途切らせる」を選択する。

### リソースの削除
余計な課金がされないように、データセット「bqml_test」を削除する。




