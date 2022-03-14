# BigQuery-ML

## 概要
BigQueryMLを使用して、Google Analyticsで収集されたセッションデータをもとに、未来の訪問ユーザー数を予想・可視化する。

<img width="957" alt="スクリーンショット 2022-03-14 22 55 18" src="https://user-images.githubusercontent.com/55085752/158186646-10276be8-18ad-4958-a74b-7670ab0d0c19.png">


## 手順
### データセットの作成
MLモデルを保存するデータセット「bqml_test」を「US」リージョンに作成する。
#### モデルの作成
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
#### モデルを評価する
ML.ARIMA_EVALUATE句で、モデルの評価指標を確認できます。
```
SELECT
 *
FROM
 ML.ARIMA_EVALUATE(MODEL bqml_test.ga_arima_model)
```
