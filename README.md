# possys
部内NFC決済システム POS-System

部内の決済をNFCのIDでやろうというものです。   MADE BY KAPIPARA LICENSE : CC BY/NC/SA 表示-継承-非営利

## ver2.00 2018/08/04 released
CUI+リモートデータベースで実装。
ユーザー追加/カード追加処理にパスワードを用いるように修正。
### ver2.10 2018/08/06 released
残高照会，Slackログ機能等細かい機能を追加。
### ver2.11 2018/08/15 released
ユーザ登録してないのにカードが登録できるバグを修正。
### ver2.12 2018/08/15 released
出金/入金時に所定のユーザで動作しないバグを修正。
### ver2.13 2018/08/16 released
残高照会の参照値が誤っているバグを修正。

## ver1.00
CUI＋ローカルデータベースで実装。


# プログラムの中身について
## possys_main.py
    possysのメインロジック。  
    Python3で実行することで，重大なエラーが起きるまでは無限ループで動作し続けます。  
    各クラス，関数の動作は以下の通りです。  

### Databaseクラス
#### __init__(self)
    Databaseクラスのコンストラクタです。  
    configparserでconfigファイルをリードして，データベースと接続する処理を行っています。

#### checkIDm(self, userIDm)
    userIDmに引数として16字以内のNFC IDmを渡すことで，データベース側(/NFCID/IDm)と照合してくれます。  
    データが存在する場合はTrueを，存在しない場合はFalseを返します。

#### checkIDm_userNum(self,userIDm)
    userIDmに引数として16字以内のNFC IDmを渡すことで，データベース側(/NFCID/IDm)(/NFCID/MemberNum)と照合します。  
    データが存在する場合は最もDataNumが若いデータのUserNumを返します。存在しない場合はFalseを返します。  
    異なるユーザーに同一のカードが動作している場合は動作を保証できません。possysでは，そういった動作をサポートしていません。

#### checkUser(self,name)
    ユーザ名を渡すことで，任意のユーザが存在するかを判定します。
    存在するならTrue，しないならFalseを返します。

#### addUser(self,name,mail,hashcode)
    引数として，ユーザー名，ユーザーのメールアドレス，パスワードのsha256ハッシュコード(64字以内)を渡すことで，データベースに新規ユーザを登録します。  
    ユーザー番号は，最も新しいMemberNumに1を足した値になります。

#### addCard(self,userIDm,userName,hashcode)
    引数として，16字以内のNFC IDm，ユーザー名，パスワードのsha256ハッシュコード(64字以内)を渡すことで，データベースに任意のユーザーに新しい認証カードを追加します。  
    カード番号は，最も新しいDataNumに1を足した値になります。
    同一ユーザーに複数のカードを追加することは可能です。しかし，異なるユーザーへの同一カードの登録についてはサポートされません。

#### money(self,userNum,amount)
    引数として，ユーザー番号と金額の増減量を渡すことで，データベース(/MoneyLog)に取引の内容を追記し，データベース(/MemberList/Wallet)を書き換え，ユーザーの所持金の値を更新します。
    userNum取得には，checkIDm_userNum関数を用いてください。
    
### idmReadクラス
#### __init__(self)
    idmReadクラスのコンストラクタです。  
    特に行う動作はありません。

#### getMain(self)
    NFCカードのIDmを取得する関数です。  
    nfcpyライブラリがPython2でのみ動作するため，サブプロセスでidmRead.pyをPython2で実行します。値の受け渡しには標準出力を用いています。

### slackLinkクラス 
#### __init__(self)
    slackLinkクラスのコンストラクタです。
    setting.ini内に記述されたSlackとの接続リンクを参照して，Slackと接続します。

#### post(self,mode,logData)
    Slackへpostする関数です。
    もっと賢い方法があるような気もしますが，modeでlogがどこから来たかを判定します。
    1なら金銭処理のログ，2ならユーザ追加ログ，3ならカード追加ログです。
    logDataはリスト型もしくはタプル型で渡され，所定のフォーマットで送信します。

### meinMenuクラス
#### __init__(self)
    meinMenuクラスのコンストラクタです。
    databaseクラスとidmReadクラスのインスタンスを作成します。

#### mainLogic(self)
    possysのCUI本体です。
    無限ループで動作し，所定の動作をするためにユーザーからのデータ入力，Databaseクラスの所定の関数を呼び出します。
