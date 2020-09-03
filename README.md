# Open Bandit Dataset & Pipelineを用いたOPEの性能評価

## Overview
2020/8/31にZOZO Tech Blogにて公開されたブログ記事[Off-Policy Evaluationの基礎とZOZOTOWN大規模公開実データおよびパッケージ紹介](https://techblog.zozo.com/entry/openbanditproject)で行った簡易実験の実装. 関連する内容の発表を2020/8/27にzoomで開催された[CFML勉強会](https://cfml.connpass.com/event/183154/)でも発表しており, その際に使用した[発表資料](https://speakerdeck.com/usaito/off-policy-evaluationfalseji-chu-toopen-bandit-dataset-and-pipelinefalseshao-jie)も参考になるはず. その他研究プロジェクトに関連する要素は以下の通り.

- 論文: https://arxiv.org/abs/2008.07146
- Open Bandit Pipeline: https://github.com/st-tech/zr-obp
- Open Bandit Dataset: https://research.zozo.com/data.html
- プレスリリース: https://corp.zozo.com/news/20200818-11223/

## Requirements
- **obp==0.2.1（要確認）**
- numpy==1.18.1
- pandas==0.25.1
- pyyaml==5.1
- scikit-learn==0.23.1


## Experimental Settings

以下の実験設定で, 標準的なOPE推定量であるDirect Method (DM)・Inverse Probability Weighting (IPW)・Doubly Robust (DR)のオフライン評価の正確さの評価を行う.
推定量の評価指標には, relative estimation errorを使用した.
より詳細な実験手順は, ブログ記事の「**Open Bandit Datasetを用いたOPEの評価方法**」という章に記載.

- Random policyをbehavior policy（旧ロジック）, BernoulliTSをcounterfactual policy（新ロジック）とみなして, BernoulliTSの性能を推定した際のオフライン評価の正確さを評価
- DMやDRに必要な機械学習モデル$\hat{\mu}$には, ロジスティック回帰を使用
- $K=10$回の異なるブートストラップサンプルによる結果を用いて, relative estimation errorの信頼区間を推定

## Usage

1. Open Bandit Datasetを`./open_bandit_dataset`におく.
2. 以下のコマンドを実行 (https://github.com/st-tech/zr-obp/tree/master/examples のREADMEを切り取ってそのままもってきたのでここだけ英語).

We evaluate the estimation performances of off-policy estimators, including Direct Method (DM), Inverse Probability Weighting (IPW), and Doubly Robust (DR).
[`./src/evaluate_off_policy_estimators.py`](./src/evaluate_off_policy_estimators.py) implements the evaluation of OPE estimators.

```bash
# run evaluation of OPE estimators.
python evaluate_off_policy_estimators.py\
    --n_boot_samples $n_boot_samples\
    --counterfactual_policy $counterfactual_policy\
    --behavior_policy $behavior_policy\
    --campaign $campaign\
    --random_state $random_state
```
where `$n_boot_samples` specifies the number of bootstrap samples to estimate confidence intervals of the performance of OPE estimators (*relative estimation error*).
`$counterfactual_policy` and `$behavior_policy` specify the counterfactual and behavior policies, respectively.
They should be either 'bts' or 'random'.
`$campaign` specifies the campaign and should be one of 'all', 'men', or 'women'.


上述の実験設定に基づき, 以下のコマンドによりOPEの評価を行った (`./src/`にて実行).
```bash
for campaign in all men women
do
    python evaluate_off_policy_estimators.py\
        --n_boot_samples 10\
        --counterfactual_policy bts\
        --behavior_policy random\
        --campaign $campaign\
        --random_state 12345
done
```

これにより, `./logs/{campaign}/`に結果のcsvファイルが生成される.
なお, `./src/conf/`には, ZOZOTOWNでデータ収集期間に用いられたBernoulli Thompson Sampling policy (ここでの実験におけるcounterfactual policy)を再現するための設定を置いてある. 詳しくは, https://github.com/st-tech/zr-obp/tree/master/examples の説明を参照.


## Results

全アイテム(All)・男性用アイテム(Men's)・女性用アイテム(Women's)に対応する3つのキャンペーンに対応するそれぞれのデータで行った結果を掲載する.
relative estimation errorが小さい推定量ほど, behavior policy（旧ロジック）が蓄積したデータを用いて, counterfactual policy（新ロジック）の正確なオフライン評価ができている.


- 全アイテム向けキャンペーンのデータにおけるOPE推定量の推定精度 (relative estimation error)

| **OPE estimators** | mean | 95.0% CI (lower) | 95.0% CI (upper) |
| :--- | :--- | :---: | :---: |
**DM** | 0.23193 | 0.23017 | 0.23359 |
**IPW** | 0.11473 | 0.06933 | 0.16282 |
**DR** | 0.11813 | 0.07153 | 0.16774 |

<br>

- 男性アイテム向けキャンペーンのデータにおけるOPE推定量の推定精度 (relative estimation error)

| **OPE estimators** | mean | 95.0% CI (lower) | 95.0% CI (upper) |
| :--- | :--- | :---: | :---: |
**DM** | 0.21503 | 0.21406 | 0.21599
**IPW** | 0.13472 | 0.09492 | 0.17173
**DR** | 0.12001 | 0.07989 | 0.15751

<br>

- 女性アイテム向けキャンペーンのデータにおけるOPE推定量の推定精度 (relative estimation error)

| **OPE estimators** | mean | 95.0% CI (lower) | 95.0% CI (upper) |
| :--- | :--- | :---: | :---: |
| **DM** | 0.22614 | 0.22519 | 0.22731 |
| **IPW** | 0.07881 | 0.04603 | 0.11242 |
| **DR** | 0.07861 | 0.04533 | 0.11327 |

