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


・early_stopping

monitor

train/valのstep でself.log()したもの("val_loss"等)を追跡するっぽい



・training_step/validation_step の 返り値

training_stepの返り値 のlossは自動的に辞書に格納されているっぽい

validationの返り値は自分で設定しない限りそのまま


