---
layout: post
title: Linear regression with Bayesian inference while imputing missing input data
date: 2024-01-10 09:56:00-0400
description: Statistical Machine Learning with Bayes Imputation in Python
tags: 
categories: sample-posts
---

予測モデルを構築する中で，データが欠損値を含むことは多々あります．素朴な対処方法としては①欠損値を含む行・列を捨てる，②平均値などの適当な値で埋める，の二つが考えられるでしょう．①・②いずれの場合も，情報を捨ててしまっていることになるため精度の低下につながります．いい方法がないか調べたところ，ベイズ的な考え方により補完が可能であると知りました．
なお，今回の内容は以下の須山さんの記事を参考にしています．須山さんは2値分類をされているので，こちらではベイズ線形回帰をやってみることにしました．

http://machine-learning.hatenablog.com/entry/2017/08/30/221801

# 欠損の分類と扱いやすさ
あまり気にしたことはありませんでしたが，欠損にもいくつかの種類があるそうです．ここでは例として，性別と体重を説明変数とし，ある病気にかかっているか否かを予測するような問題を考えます．欠損の分類については初めて学んだので，誤解などあればご指摘いただけますと幸いです．

* MCAR (Missing Completely At Random): 欠損値の発生が完全にランダムである場合です．つまり，ほかの説明変数の値によって欠損が生じやすくなったりしないというケースです．例えば，何人かに対して体重の情報を聞き取るのをうっかり忘れてしまったとします．これはまさにMCARにあたります．
* MAR (Missing At Random): 欠損値の発生がほかの説明変数に依る場合です．先ほどの例では，性別が女性の場合に体重を申告してくれない場合が多かったとすると，これはMARに該当します．
* MNAR (Missing Not At Random): 他の説明変数を固定しても，欠損値の発生が欠損値の値に依る場合です．例としては，男女別で考えたとしても，体重が重い人は申告してくれないというケースが考えられます．他の説明変数を固定するというのが重要なポイントです．

扱いやすさとしては，MCAR > MAR > MNARとなっています．MCARは以下で述べるベイズ的な考え方で対処することができます．MARに関しては，欠損の発生を表す確率変数を導入することで対処するようですが，わりと難しそうです．MNARへの対処も少し調べてみましたが，かなり厳しそうだということがわかりました．
今回の記事ではMCARの取り扱い方を説明します．

# 理論
まず問題設定を明確にし，事後分布の直接計算が困難であることを示します．また他の変数を所与としたときの事後分布を解析的に計算し，ギブスサンプリングの手順を示します．

## 問題設定
訓練データの入力全体を$X$で表し，その中で観測できたデータを$X_{\rm observed}$，欠損したデータを$X_{\rm missing}$と表すことにします．説明変数の数を$M$，データ数を$N$とすると，入力データ$X$は$N \times M$行列になっています．訓練データの出力値を${\bf y}$とします．また簡単のため，説明変数と目的変数はそれぞれ標準化(つまり平均0，分散1になるように)されているものとします．
今回はパラメータ${\bf w}$による線形回帰を扱うことにします：

$$
{\bf y} = X {\bf w} + \hat{\bf \epsilon}
$$

ただし$\hat{\bf \epsilon}$は平均$0$，分散$1/\beta$の正規分布に従うものとします．ここから，${\bf y}$の事後分布は

$$
p({\bf y} \mid X, {\bf w}) = \left( \dfrac{\beta}{2\pi} \right)^{N/2} \exp \left[ - \dfrac{\beta}{2} ({\bf y} - X{\bf w})^2 \right]
$$

となります．
さらにパラメータ${\bf w}$と入力データ$X$の事前分布を以下のように仮定します：

$$
p(\mathbf{w}) = \left( \frac{\alpha}{2\pi} \right)^{\frac{M}{2}} \exp \left[ -\frac{\alpha}{2} \mathbf{w}^T \mathbf{w} \right] \\
p(X) = \prod_{n} \left( \frac{ |\Lambda| }{2\pi} \right)^{\frac{1}{2}} \exp \left[ -\frac{1}{2} \mathbf{x}_n^T \Lambda \mathbf{x}_n \right]
$$

ただし$\alpha$と$\Lambda$はそれぞれパラメータ${\bf w}$と入力$X$の分布を決めるパラメータです．

いま考えたい問題はいくつかに分けることができるでしょう．
1. 与えられた訓練データ$X_{\rm observed}$と${\bf y}$から${\bf w}$と$X_{\rm missing}$の分布を推定する
2. ハイパーパラメータ$\alpha$, $\beta$として適切な値を選ぶ($\Lambda$は$X_{\rm observed}$からうまく選べるはずなのでここでは無視)
3. 新しい入力${\bf x}$に対する出力値$y$を予測する
以下では1から3を実現する手続きを示します．

## 1. 訓練データからパラメータの分布を推定
ベイズの定理を用いると${\bf w}$と$X_{\rm missing}$の事後分布は

$$
p(\mathbf{w}, X_{\text{missing}} \mid X_{\text{observed}}, \mathbf{y}) \propto p(\mathbf{y} \mid \mathbf{w}, X) p(\mathbf{w}) p(X_{\text{missing}} \mid X_{\text{observed}})
$$

という形に分解することができます(各変数の依存関係に注意しつつ，丁寧に変形すると導けます)．
目的は事後分布$p({\bf w}, X_{\rm missing} \mid X_{\rm observed}, {\bf y})$の計算により${\bf w}$と$X_{\rm missing}$を推定することですが，この事後分布は簡単に取り扱うことができません．いつも通り${\bf w}$について平方完成しようとすると，その「おつり」の項が$X_{\rm missing}$を含んでおり，$X_{\rm missing}$について複雑な形状となっていしまうことが問題です．
一方で，他のすべての変数を所与としたときの事後分布は(少し大変な計算により)

$$
\mathbf{w} \mid \sim \mathcal{N} \left( \left(X^\top X+ \frac{\alpha}{\beta} I_m \right)^{-1} X^\top \mathbf{y},~ \left(\beta X^\top X + \alpha I_m \right)^{-1} \right) \\
x_{n, m} \mid \sim \mathcal{N} \left( \frac{\beta y_n \mathbf{w}_m - \sum_{m' \neq m} (\beta \mathbf{w}_{m'} \mathbf{w}_m +  \Lambda_{m, m'} ) x_{n,m'} }{\beta \mathbf{w}_m^2 + \Lambda_{m,m} },~ \frac{1 }{\beta \mathbf{w}_m^2 + \Lambda_{m,m}} \right)
$$

となることが示せます．
ゆえに以下に示す手続きから，事後分布$p({\bf w}, X_{\rm missing} \mid X_{\rm observed}, {\bf y})$に従う${\bf w}$と$X_{\rm missing}$を抽出することができます：

(0) $X$を計算するために，平均値などを使い適当に欠損値を埋める．
(1) ${\bf w} \mid *$の従う分布に基づき${\bf w}$をサンプリングする．
(2) $x_{n, m} \mid *$の従う分布に基づき$x_{n, m}$をサンプリングする(ただし$(n,m)$は欠損値に対応するデータ数，説明変数のラベル)．
(3) (1)(2)を繰り返す．

これはまさにギブスサンプリングであり，サンプリングした結果の事後分布$p({\bf w}, X_{\rm missing} \mid X_{\rm observed}, {\bf y})$への収束が保証されている点が大きなメリットです．また，メトロポリス・ヘイスティング法では提案分布を決めるために新しいハイパーパラメータを導入する必要がありますが，ギブスサンプリングでは不要であるという点も便利です．
ギブスサンプリングが可能だったのは，他の変数が所与であるとしたときの分布$p({\bf w} \mid *)$，$p(x_{n, m} \mid *)$が計算可能だったことに由来します．ここには今回のモデルが線形回帰であるという点が強く効いており，一般の場合にはギブスサンプリングは可能でないかもしれません．その場合は，メトロポリス・ヘイスティング法などを利用する必要があるでしょう．

何にせよ，事後分布$p({\bf w}, X_{\rm missing} \mid X_{\rm observed}, {\bf y})$に従う${\bf w}$と$X_{\rm missing}$をサンプリングすることができるようになりました．ここから，事後分布$p({\bf w}, X_{\rm missing} \mid X_{\rm observed}, {\bf y})$を推定することができるようになります．実際には，${\bf w}, X_{\rm missing}$の平均や分散をもとめれば分布の特徴をおおよそ捉えることができるため，${\bf w}, X_{\rm missing} $のサンプル平均を計算することになります．

# 2. ハイパーパラメータの調整
今回は最尤推定によってハイパーパラメータを調整しています．最尤推定とは$p({\bf y} \mid X_{\rm observed}, \alpha, \beta)$を最大化する$\alpha$, $\beta$を計算することですが，

$$
\begin{align}
p(\mathbf{y} \mid X_{\text{observed}}, \alpha, \beta) &= \int p(\mathbf{y}, \mathbf{w}, X_{\text{missing}} \mid X_{\text{observed}}, \alpha, \beta)d\mathbf{w} dX_{\text{missing}}  \\\\
&= \int p(\mathbf{y} \mid \mathbf{w}, X, \alpha, \beta) p(\mathbf{w}, X_{\text{missing}} \mid X_{\text{observed}}, \alpha, \beta) d\mathbf{w} dX_{\text{missing}} \\\\
&= \mathbb{E}_{\mathbf{w}, X_{\text{missing}} \mid X_{\text{observed}}, \alpha, \beta} \left[ \mathcal{N}(\mathbf{y} \mid X \mathbf{w}, \beta^{-1}) \right]
\end{align}
$$

によって$p({\bf y} \mid X_{\rm observed}, \alpha, \beta)$を計算できるため，これを最大化する$(\alpha, \beta)$を求めればよいことになります．事後分布$p({\bf w}, X_{\rm missing} ~\mid X_{\rm observed}~~, \alpha, \beta)$に基づく${\bf w}, X_{\rm missing}$をサンプリングすることができるため，以下の手続きで$\alpha$, $\beta$を調整できます：

(0) $\alpha_0$, $\beta_0$を適当に初期化
(1) $L(\alpha_t, \beta_t) = \mathbb{E} _ {\bf w}, X _ {\rm missing} ~\mid X _ {\rm observed}~~, \alpha_t, \beta_t} \left[ \mathcal{N}({\bf y} \mid X {\bf w}, \beta_t) \right]$を，事後分布に基づきサンプリングされた${\bf w}, X_{\rm missing}$から計算
(2) 数値微分により$\alpha_{t+1} = \alpha_t + \eta \dfrac{\partial L}{\partial \alpha}(\alpha_t, \beta_t)$のように更新
ここで$\eta$は適当に決め打ちする必要があります．またスケールの問題から，実際には対数尤度を計算する方がよいでしょう．

# 3. 新しい入力に対する予測
新しい入力${\bf x}$に対して予測をしたいものとします．今回は${\bf x}$が欠損している可能性があるため，欠損している${\bf x} _ {\rm missing}$と${\bf x} _ {\rm observed}$に分けます．事後分布$p({\bf x} _ {\rm missing}, y \mid {\bf x} _ {\rm observed}, X_{\rm observed}, {\bf y}, \alpha, \beta)$を解析的に計算することはできないので，ギブスサンプリングによって分布を推定します．
先ほどと同様に，他の変数を条件づけたときの分布は

$$
\mathbf{w} \mid * \sim \mathcal{N} \left( \left(X^T X+ \frac{\alpha}{\beta} I_m \right)^{-1} X^T \mathbf{y},~ \left(\beta X^T X + \alpha I_m \right)^{-1} \right) \\\\
x_{m} \mid * \sim \mathcal{N} \left( \frac{\beta y w_m - \sum_{m' \neq m} (\beta w_{m'} w_m +  \Lambda_{m, m'} ) x_{m'} }{\beta w_m^2 + \Lambda_{m,m} },~ \frac{1}{\beta w_m^2 + \Lambda_{m,m}} \right) \\\\
y \mid * \sim \mathcal{N}(\mathbf{w}^T \mathbf{x}, \beta^{-1})
$$

となっているため，事後分布$p({\bf x} _ {\rm missing}, y , {\bf w} \mid {\bf x} _ {\rm observed}, X_{\rm observed}, {\bf y}, \alpha, \beta)$をサンプリングによって推定できます．ここで関係ないはずの${\bf w}$が混入していることに違和感があるかもしれませんが，${\bf w}$の事後分布がサンプルを通じてしか求められず，$y$の分布が${\bf w}$を通じて求められる以上，${\bf w}$も含めてサンプルする必要があります．
いま，${\bf w}$は訓練データ$X, {\bf y}$のみに依ることに注意します．そのため，${\bf w}$は依然サンプリングした結果を保存してそのまま使うことができます．求めたいのは${\bf x}$と$y$の分布なので，サンプリングした結果から平均や分散を計算すればよいでしょう．

# 実装
先ほどの理論に基づき，説明変数の補間が可能なベイズ線形回帰のクラス`Bayesian_LR_with_intepolation`を実装しました．このクラスの便利なところは，欠損に対する前処理をすることなくデータを入れてしまえばよいという点です．また標準化や事前分布の設定もクラス内でいい感じにやってくれます．
主なメソッドは以下のようになっています：
`fit`：訓練データを標準化し，ハイパーパラメータを調整する．その後，事後分布$p({\bf w}, X_{\rm missing} \mid *)$に従う${\bf w}$と$X_{\rm missing}$をサンプリングする．サンプルの平均と標準偏差(つまり事後分布に基づくの期待値と標準偏差)を計算できる．
`predict`：与えられたテストデータに対する予測をする．訓練データとテストデータを使いサンプリングし，出力の期待値と標準偏差を返す．テストデータに欠損がある場合は，欠損値に対する予測もする．

```python
class Bayesian_LR_with_interpolation: # 説明変数の補間が可能な，ベイズ線形回帰
    def __init__(self, alpha = 1.0, beta = 1.0):
        self.alpha = alpha # alphaはパラメータwの事前分布のパラメータで，alphaが大きいほどwの分布は狭くなる．
        self.beta = beta # betaは誤差のオーダーの逆数
    
    def fit(self, X_train_original, y_train_original, num_sample = 1000, fit_hyper_parameters = True, num_iteration = 100, eta = 0.1):
        dalpha = 0.01
        dbeta = 0.01
        self.hyper_param_record = np.zeros((num_iteration, 2)) # ハイパーパラメータの軌跡を保存
        self.normalize_train(X_train_original, y_train_original) # 標準化されたself.X_train, self.y_trainが生成される
        if len(self.nan_list) == 0:
            print("Note: There are no missing values in the explanatory variables.")
        
        w_sample_mean = np.zeros(len(self.X_train[0])) # サンプリングから計算したwの期待値を格納
        w_sample_std = np.zeros(len(self.X_train[0])) # サンプリングから計算したwの標準偏差を格納
        X_sample_mean = self.X_train.copy() # サンプリングから計算したXの期待値を格納，欠損値だけ更新
        X_sample_std = np.zeros(self.X_train.shape) # サンプリングから計算したXの標準偏差を格納，欠損値だけ更新
        X_sample = self.X_train.copy() # 現時点でのサンプルの値，欠損値だけ更新する
        self.w_samples = np.zeros((num_sample, len(self.X_train[0]))) # wに関するサンプリング結果をすべて集める，予測の際に必要
        
        # ハイパーパラメータを調整，勾配法により尤度を上げるようなハイパーパラメータを探す
        if fit_hyper_parameters: # ハイパーパラメータを調整する場合
            likelihood = self.likelihood(num_sample) # 初期化
            for idx_iteration in range(num_iteration):
                self.hyper_param_record[idx_iteration] = np.array([self.alpha, self.beta])
                self.alpha = self.alpha + dalpha
                likelihood_new = self.likelihood(num_sample)
                self.alpha = self.alpha - dalpha + eta*(likelihood_new - likelihood) / dalpha
                likelihood = likelihood_new
                self.beta = self.beta + dbeta
                likelihood_new = self.likelihood(num_sample) # 新しい尤度を計算
                self.beta = self.beta - dbeta + eta*(likelihood_new - likelihood) / dbeta
                likelihood = likelihood_new
                print("iteraton = {0:} / {1:}: alpha = {2:.6f}, beta = {3:.6f}, L = {4:.6f}".format(idx_iteration, num_iteration, self.alpha, self.beta, likelihood), end = "\r")
        
        # サンプリングによりwとXの統計量を計算
        for t in range(num_sample):
            # wのサンプリング
            w_sample = self.sample_w(X_sample)
            self.w_samples[t] = w_sample # predictionで使うため
            w_sample_mean = self.update_mean(w_sample_mean, w_sample, t)
            w_sample_std = self.update_std(w_sample_std, w_sample, t)
            
            # Xのサンプリング(欠損している箇所のみ)
            for idx, (n, m) in enumerate(self.nan_list):
                X_sample[n, m] = self.sample_x_nm(n, m, w_sample, X_sample[n], self.y_train[n])
                X_sample_mean[n, m] = self.update_mean(X_sample_mean[n, m], X_sample[n, m], t)
                X_sample_std[n, m] = self.update_std(X_sample_std[n, m], X_sample[n, m], t)
                
        # 標準化を解除
        X_sample_mean = np.tile(self.x_train_original_mean, reps = (len(X_train_original), 1)) \
                            + X_sample_mean * np.tile(self.x_train_original_std, reps = (len(X_train_original), 1))
        X_sample_std = np.tile(self.x_train_original_std, reps = (len(X_train_original), 1)) * X_sample_std
        print("Fitting has finished: ")
        print("alpha = {:.6f}".format(self.alpha))
        print("beta = {:.6f}".format(self.beta))
        return w_sample_mean, w_sample_std, X_sample_mean, X_sample_std
    
    def normalize_train(self, X_train_original, y_train_original): # 入力を標準化しつつ，標準化を解除できるように値を保存
        # Xについての処理
        self.x_train_original_mean = np.nanmean(X_train_original, axis = 0)
        self.x_train_original_std = np.nanstd(X_train_original, axis = 0)
        self.X_train = (X_train_original - np.tile(self.x_train_original_mean, reps = (len(X_train_original),1)) ) \
                            / np.tile(self.x_train_original_std, reps = (len(X_train_original), 1)) # 標準化
        self.Lambda = np.linalg.inv( np.cov(self.X_train[np.isnan(X_train_original).sum(1) == 0].T) )# 説明変数がどれも欠損していないデータ行から計算した共分散行列の逆行列
        self.nan_list = list(zip(*np.where(np.isnan(X_train_original)))) # もとの入力データでnullの場所を格納
        self.X_train[np.isnan(X_train_original)] = 0.0 # 標準化しているため0で埋めてok
        
        # yについての処理
        self.y_train_original_mean = y_train_original.mean()
        self.y_train_original_std = y_train_original.std()
        self.y_train = (y_train_original - self.y_train_original_mean) / self.y_train_original_std
    
    def likelihood(self, num_sample):
        L = 0.0
        X_sample = self.X_train.copy()
        for t in range(num_sample):
            w_sample = self.sample_w(X_sample)
            for idx, (n, m) in enumerate(self.nan_list):
                X_sample[n, m] = self.sample_x_nm(n, m, w_sample, X_sample[n], self.y_train[n])
            L += np.exp( -0.5 * self.beta * ((self.y_train - X_sample.dot(w_sample))**2.0).sum() )
        # 訓練データ数が増えるにつれてlog_likelihoodは線形に増える(はず)なので規格化．これをしないとデータ数を変えるたびにetaを変更する必要がありそう
        L = np.log(L) / len(self.y_train) + 0.5*np.log(self.beta) 
        return L
    
    def update_mean(self, x_mean, x, t):
        return (1.0 / (t+1.0)) * (t*x_mean + x)
    
    def update_std(self, x_std, x, t):
        return np.sqrt( (t/(t+1.0)) * (x_std**2.0 + (x_std-x)**2.0 / (t+1.0) ) )
    
    def sample_w(self, X): # 他を条件づけたときのwをサンプル
        y, alpha, beta = self.y_train, self.alpha, self.beta
        w_mu = np.linalg.inv(X.T.dot(X) + (alpha/beta)*np.eye(len(X[0]))).dot(X.T).dot(y) # 他の変数を条件づけたときの分布の期待値n
        w_sigma = np.linalg.inv(beta * X.T.dot(X) + alpha*np.eye(len(X[0]))) # 標準偏差
        return np.random.multivariate_normal(w_mu, w_sigma)
    
    def sample_x_nm(self, n, m, w, x_n, y_n): # 他を条件づけたときのx_nm(欠損値)をサンプル
        alpha, beta = self.alpha, self.beta
        x_nm_mu = x_n[m] + (beta*y_n*w[m] - (beta*w*w[m]+self.Lambda[m]).dot(x_n)) / (beta*w[m]*w[m] + self.Lambda[m,m])
        x_nm_sigma = 1.0 / np.sqrt(beta*w[m]*w[m] + self.Lambda[m,m])
        return np.random.normal(x_nm_mu, x_nm_sigma)
    
    def sample_y(self, X, w): # 他を条件づけたときの出力yをサンプル
        y_mu = X.dot(w)
        y_sigma = (1.0/self.beta) * np.eye(len(y_mu))
        return np.random.multivariate_normal(y_mu, y_sigma)
    
    def predict(self, X_test_original, num_sample = 1000): # 欠損がある可能性のある入力から予測
        X_test = (X_test_original - np.tile(self.x_train_original_mean, reps = (len(X_test_original),1)) ) \
                            / np.tile(self.x_train_original_std, reps = (len(X_test_original), 1)) # 標準化
        nan_list = list(zip(*np.where(np.isnan(X_test_original)))) # もとの入力データでnullの場所を格納
        X_test[np.isnan(X_test_original)] = 0.0 # 標準化しているため0で埋めてok
        
        X_test_sample_mean = X_test.copy() # サンプリングから計算したXの期待値を格納，欠損値だけ更新
        X_test_sample_std = np.zeros(X_test.shape) # サンプリングから計算したXの標準偏差を格納，欠損値だけ更新
        X_test_sample = X_test.copy() # 現時点でのサンプルの値，欠損値だけ更新する
        
        y_test_sample_mean = np.zeros(len(X_test_original))
        y_test_sample_std = np.zeros(len(X_test_original))
        
        # サンプリングによりyとXの統計量を計算
        for t in range(num_sample):
            print("prediction: t = {0:} / {1:}".format(t, num_sample), end = "\r")
            # wのサンプル(前の結果を流用)
            w_sample = self.w_samples[t]
            # yのサンプル
            y_test_sample = self.sample_y(X_test_sample, w_sample)
            y_test_sample_mean = self.update_mean(y_test_sample_mean, y_test_sample, t)
            y_test_sample_std = self.update_std(y_test_sample_std, y_test_sample, t)
            # Xのサンプル           
            for idx, (n, m) in enumerate(nan_list):
                X_test_sample[n, m] = self.sample_x_nm(n, m, w_sample, X_test_sample[n], y_test_sample[n])
                X_test_sample_mean[n, m] = self.update_mean(X_test_sample_mean[n, m], X_test_sample[n, m], t)
                X_test_sample_std[n, m] = self.update_std(X_test_sample_std[n, m], X_test_sample[n, m], t)
        # 標準化を解除
        y_test_sample_mean = self.y_train_original_mean + self.y_train_original_std * y_test_sample_mean
        y_test_sample_std = self.y_train_original_std * y_test_sample_std
        X_test_sample_mean = np.tile(self.x_train_original_mean, reps = (len(X_test), 1)) \
                            + X_test_sample_mean * np.tile(self.x_train_original_std, reps = (len(X_test), 1))
        X_test_sample_std = np.tile(self.x_train_original_std, reps = (len(X_test), 1)) * X_test_sample_std
        print("Predictions has finished.")
        return y_test_sample_mean, y_test_sample_std, X_test_sample_mean, X_test_sample_std
    
```


# 数値実験
実装がうまくいっているか確認するために，トイモデルを作成してみました．実装は以下のリンクにあります．
https://github.com/yotapoon/Bayesian_LR_with_interpolation

## 設定
説明変数は$(x_1, x_2)$とし，目的変数を$y$とします．これらは

```math
x_1 \sim \mathcal{N}(2.0, 0.4) \\
x_2 \mid x_1 \sim \mathcal{N}(-2.0 x_1 + 0.5, 0.8) \\
y \mid x_1,x_2 \sim \mathcal{N}(3.0 x_1 + 1.5x_2, 0.8)
```

のような関係にあるものとし，訓練データとして$N = 100$点を生成します．
ここではMCARの状況を考えます．つまり，欠損の発生が他の変数に依存しないような扱いやすい状況です．また訓練データの入力値$X$の各要素には確率$p = 0.3$で欠損が生じるものとします．$X$は以下のような形状です：

```
[ 1.58535427         nan]
 [ 2.10601671 -4.55318535]
 [ 1.80926526 -3.5396434 ]
...
```

上のようなデータに対して，以下の四つの方法により`fit`します：
① 欠損のないデータ(complete data)を用いて回帰．
② $x_2$をその平均値により補完．
③ $x_2$が欠損している行をそのまま削除．
④ ベイズ線形回帰により補完．

また同様のテストデータを$N' = 1000$点生成し，テスト誤差を計算することでそれぞれの性能を評価します．ただし，各手法を比較しやすくするためにテストデータに欠損はないものとします．
期待される結果は以下の通りです：
・欠損のないデータによる回帰①は最も良い性能を与えてほしい．
・平均値による補完②はあまりよくないと聞くので，それほど精度がでないはず
・欠損している行をそのまま削除する方法③は情報量が減るためそこまでいい結果にはならないはず
・ベイズ線形回帰④をすると欠損のないデータによる回帰①より精度は出ないはずだが，適当な補完②や行の削除③よりは精度は出てほしい

## 結果と考察
数値実験の結果を以下に示します．確認のため，今回実装したクラスを使わない最尤推定の結果も示しています．`y_error`は$y$に関するテスト誤差の平均値，`X_error`は$X$に関する訓練誤差の平均値です．

① 欠損のないデータ(complete data)を用いて回帰．

```
=====最尤推定=====
y_error = 0.6841894767175021

=====ベイズ線形回帰=====
Note: There are no missing values in the explanatory variables.
alpha = 1.482402
beta = 2.115292
y_error = 0.6814939439707199
```


② $x_2$をその平均値により補完．

```
=====最尤推定=====
y_error = 0.9199739418129449

=====ベイズ線形回帰=====
Note: There are no missing values in the explanatory variables.
alpha = 4.767952
beta = 1.785241
X_error = 0.03779597748274369, 0.5054058111998435
y_error = 0.9434735826603966
```


③ $x_2$が欠損している行をそのまま削除．

```
=====最尤推定=====
y_error = 0.7259293059445424

=====ベイズ線形回帰=====
Note: There are no missing values in the explanatory variables.
alpha = 1.721914
beta = 3.476663
y_error = 0.7766452769398361
```


④ ベイズ線形回帰により補完．

```
=====ベイズ線形回帰=====
alpha = 2.567708
beta = 4.346280
X_error = 0.024276115858132833, 0.2436076829842362
y_error = 0.7147843731641442
```

最尤推定とベイズ線形回帰の$y$に関する誤差が割と整合しているので，結果は正しそうです．
また`y_error`を比較すると，平均で補完 > そのまま削除 > ベイズ線形回帰 > complete dataとなっているため，期待された結果が得られています．さらに`X_error`をみると，ベイズ線形回帰は平均値による補完と比較して優れていることがわかります．
今回は欠損の生じていないテストデータに対する予測により評価しましたが，本手法は欠損が生じているデータに対してもある程度の精度で予測が可能です．これは他の手法と比べると大きなアドバンテージです．

一方で，生成したデータによってはベイズ線形回帰の優位性がそれほど明らかでなかった場合がありました．特にデータ数が多かったりノイズが小さかったりするときには，欠損している行を削除してしまってもよい結果が得られます．そのため，本手法が効果を発揮するのは「入出力がある程度の相関を持ちつつ，ノイズも大きいような場合」であるといえそうです．

# まとめ
今回は欠損のある入力をベイズ的に補完しつつ，欠損値の推定，ハイパーパラメータの調整，新しい入力に対する予測を行うクラスを実装しました．数値実験から，実装が正しそうであることとベイズ線形回帰の効果を確認しました．本手法のメリットとしては，
・情報量増加による予測精度の向上
・訓練データへの前処理の自動化
・欠損のあるデータに対する予測能力
があります．特に最後の点に関しては，その他の手法に比べ非常に大きな利点であるといえます．

一方で，デメリットとしては
・導出，実装が非常に面倒であること
・サンプリングのために計算量が増加すること
が挙げられます．前者に関しては，欠損値を埋めたいという強いモチベーションがない限り，割に合わないというのが正直な感想です(導出・実装するのに丸二日かかった)．後者に関しては，事後分布を解析的に書けないためどうしても避けられないと考えます(もしかしたらできるのかもしれませんが．．．たかが線形回帰と思っていましたが，意外と複雑になるようです)．

今回は最も扱いやすいMCARにフォーカスしましたが，MARの場合は欠損が生じる確率をモデル化することにより同様の解析が可能になります．きちんとは読んでいませんが，
https://bookdown.org/marklhc/notes_bookdown/missing-data.html
がそのような内容になっていると思います．時間があれば，MARの場合にも同様の実装をするつもりです．

また，平均値による補完と欠損値の削除がそれぞれ有利になるのはどのような状況なのかが少し気になりました．データ数やノイズの大きさ，変数間の相関に依存すると思いますが，理論的な解析もできるのかもしれません．
