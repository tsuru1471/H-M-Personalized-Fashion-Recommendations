# H-M-Personalized-Fashion-Recommendations


# 4/18
参加決定。まずはmost vote discussionを読む

現在の疑問

1.複数の予測をどのようにしておこなうべきか。

順位ごとにmodelを作成 ?

→ a,完全にそのランク特化で作成
　　重複すればそれを消して予測
  　
  
  b,全ランクで共通したモデルを作成
    上のランクで採用されたものを消して予測
    
    → 帰路にて、bなら、こんなことしなくても一つのモデルの予測確立を上から順に並べ替えるだけでいいと気づく。
    
 
とりあえず、無加工 lighgtgbm modelbにてbaseライン確保を目指す。時系列でやりたかったアンサンブルバリデーションやってみる。

その後は、EDA、nnアンサンブルなどやればいいかな～?
   


# 4/21

・pytorch lightning API　お勉強

例えばloaderにデコレータをつけるか否かなど、versionの違いで戸惑う部分も多い。単純に参考文献も多くなくとてもしんどいが身に着けるためKaggleしながら毎日API見ていくしかないっすね。

・subset

pure pytorch におけるデータセット後の分割方法

Subset(Train(dataset),foldを示すindex)



# 4/25

・early stopping

trainerインスタンス作成時にcallbackのリストに入れて、fit。引数にtrainとvalloader入れる？

→modelだけでもやってくれそうだが、、、

→ 4/28 追記


・training_step & traing_epoch_step

バッチ学習の間の処理がtraining_step、1エポック終わった後の処理がtrainig_epoch_step

validationも全く同じ。To activate the validation loop while trainingってドキュメントに書いてた。




・計画

DataModuleを普通に作成。FOLd分けた後に、どうにかそのFoldを反映させられるようにfitに与えたい。

案1 setup内でkfold

DataModuleのコンタラクタにどのfoldを採用するかの引数を与える。k=1なら初めのfoldなど。それを全foldでできるようにfor ループ

https://gist.github.com/ashleve/ac511f08c0d29e74566900fd3efbb3ec


対して、kfoldループを作ってからmodel定義するならこんな感じ？

kf=,,,

for train_idx,test_idx in kf.split,,, :

    def MyDataModule(LightningDataModule):
    
        def __init__(self,,,):
        
            super().__init__()
            
            self.train=data[train_idx]
            
            self.valid=data[vaid_idx]
            
            ....
            
    def Net(LightningModule):
    
    ....
   
   
  
  自分だのコードなら別によさそうだけど、可読性やしっくり感でとりあえず、setup内foldでやる
    

# 4/26

・random_seed

①　setup時に分離するseedを固定(kf)すると、train,valの組み合わせが固定される。

②　学習時に固定すれば、学習間でデータを固定できる。

   → epoch前に固定すればエポックごとに異なるデータの組となり、ロード前つまりバッチ処理時に固定数れば全エポックでバッチの組み合わせが固定される


・torch.argmax(data,ndim=None)

ndimはaxisと同じ。最大indexを格納した一次元tensor配列を作成。defaultでは最大要素のindexをreturnするのに注意


・torch.cat([],dim=0)

入力した配列をdim方向に追加。



・torch.stack([],dim=0)

入力した配列を新たな次元を追加して格納。(2,3)も配列を三つ格納したリストを入力すると(3,2,3)の配列が返る


・tensor.view()

ほぼnumpyのreshape。.view(-1)で一次元tensorにできる。


・torch.detach()

ただbackward()機能切ってるだけ?

・torch.numpy()

numpy型

・torch.item()

単一要素のその値を取り出す。


pl

・training_step はreturn でlossが必要だが、validation_stepはそうでもないっぽい。




# 4/27

・early stopping API

_run_early_stopping_check() でealry_stopping処理が行われるっぽい。ただdefaultでvalidation_epoch_end時に実行される風に書いているのに、source見る感じその逆で困惑。

実際実行してみて、どっちで学習切れてるか要確認。

↑ 引数 : check_on_train_epoch_end がdefaultのままでよいのか否か


# 4/28

・gpu

Trainerのコントラクタの引数で指定。gpus=1 ←台数

・trainer.fit()

pl.DataModuleでデータセット作ったら、そのインスタンスを引数datamodeuleに与えるっぽい

https://github.com/PyTorchLightning/pytorch-lightning/blob/master/pytorch_lightning/trainer/trainer.py

>  datamodule: An instance of :class:`~pytorch_lightning.core.datamodule.LightningDataModule`



・training_step/validation_step の 返り値

training_stepの返り値 のlossは自動的に辞書に格納されているっぽい

validationの返り値は自分で設定しない限りそのまま


・学習順序

step時にt/v,epoch_endでt_e/v_eを出力させた

![image](https://user-images.githubusercontent.com/92427575/165779040-bc5a3015-4dad-451f-b476-912e4f4f1529.png)

train_step →　valid_step → valid_epoch_end → train_epocH_end の順だった。

またなぜか初めに validtionが行われている

![image](https://user-images.githubusercontent.com/92427575/165780032-1f4caefb-3ca4-47ad-9a83-477fc67c9fa6.png)

こういう観点からもepoch_end時に特に何か出力する必要もないのではないかなと感じた。

ただdefaultでは更新時のみ出力される点、epochが不明な点を考慮して training_epoch_endで次のepochを表示するのは良いかもなと思った。

しかしこれは最後っ屁があるためそこが不満ではある。

===============================================================================================================================================================


・early_stopping

monitor

train/valのstep でself.log()したもの("val_loss"等)を追跡するっぽい

min_delta

scoreの更新量と比較する値。defaultでは0.0


・学習モデル読み込み

https://goody-jp.com/pytorch-lightning%E3%81%AEckpt%E3%83%95%E3%82%A1%E3%82%A4%E3%83%AB%E3%82%92load%E3%81%99%E3%82%8B%E3%81%AE%E3%81%AB%E3%81%AF%E3%81%BE%E3%81%A3%E3%81%9F%E8%A9%B1%E3%81%AE%E3%81%9D%E3%81%AE%E5%BE%8C/

この通りやろう!!!

modelに別の引数(layer層数など)を設定している場合、この通りやらないと "int object ... not callable"みたいなエラー出る。理由は不明





