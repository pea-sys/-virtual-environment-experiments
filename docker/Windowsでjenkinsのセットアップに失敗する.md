- docker で Jenkins コンテナ起動後に、コンテナにアクセスすると
  　次のようなプラグインのセットアップ画面が表示される  
  ![1](https://user-images.githubusercontent.com/49807271/230772375-4c749c7f-55e6-47f6-9305-5e9ec938ef5f.png)

- ここで install suggested plugins を選択すると、フォルダがないよと怒られる  
  ![2](https://user-images.githubusercontent.com/49807271/230772380-6e69677e-93db-41bc-af92-5e0cf0870441.png)

- 原因は不明だが、Windows 環境で同様の問題を抱えている方が何人か確認できた。  
  jenkins サービスを再起動することで使用できるようになるとのことで、私もその手順で復旧した。  
  http://localhost:8080/safeRestart  
  にアクセスすることで jenkins サービスが再起動できる。  
  ただ、推奨されているプラグインは入っていない状況だと思われる。
  または最初のプラグインインストール選択画面で[x]でウィンドウを閉じても同じ状況になると思われる。  
  Master-Slave 構成の場合、SlavePlugin が必要になる。  
  私の場合は厄介なことにプラグインが表示されない状況になった。
  https://issues.jenkins.io/browse/JENKINS-4833
