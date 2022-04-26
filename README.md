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

