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

