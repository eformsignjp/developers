
--------------------------
eformsign API の使い方
--------------------------

eformsign が提供する API を利用し、eformsign の機能を顧客のシステム/サービスで呼び出して使用できる機能です。


始める前に 
================

eformsign API を利用するためには、次のような事前作業が必要です。

- 会社 ID と 文書 ID の確認
- API キーの作成および秘密鍵の確認
- 署名の作成

.. caution:: 
   
   署名の作成には30秒の時間制限があります。30秒以内に署名を作成し、トークンを取得してください。 



会社 ID と 文書 ID の確認
---------------------------

eformsign API を使用するためには、会社 ID と照会対象の文書 ID が必要です。 

eformsign サービスにログインし、会社 ID と文書 ID を確認してください。

.. note:: 

   会社 ID は、左側のメニューの **会社管理 > 会社情報** メニューの **基本情報** タブで確認することができます。

   |image1|

   文書 ID は、各文書トレイの右上にある文書アイコン(|image2|)をクリックし、文書 ID コラムを追加すると、照会したい文書の ID を確認することができます。 

   |image3|



.. _apikey:

API キーの作成および秘密鍵の確認
----------------------------------------

1. eformsign に代表管理者としてログインし、メニューツリーで **コネクト > API/Webhook** に移動します。 

.. image:: resources/apikey1.PNG
    :width: 700
    :alt: コネクト > API/Webhook メニューの位置


2. **API キー** タブを選択し、**API キーの作成** ボタンをクリックします。 

.. image:: resources/apikey2.PNG
    :width: 700
    :alt: 「API キー登録」ボタン


3. **API キーの作成** ポップアップ画面にエイリアスとアプリケーション名を入力し、**保存** ボタンをクリックします。

.. image:: resources/apikey3.PNG
    :width: 700
    :alt: 「API キーの作成」ポップアップ画面


4. 作成されたキーリストから **キーを表示** ボタンをクリックすると、API キーと秘密鍵を確認できます。

.. image:: resources/apikey4.PNG
    :width: 700
    :alt: 「API キーを表示」ボタンの位置

.. image:: resources/apikey5.PNG
    :width: 700
    :alt: API キーと秘密鍵の確認 



.. note:: **API キーの修正**

    作成されたキーリストから **修正** ボタンをクリックし、エイリアスとアプリケーション名を修正することができます。
  	また、ステータス領域をクリックし、活性/非活性状態に変更することもできます。

.. note:: **API キーの削除**

    作成されたキーリストから **削除** ボタンをクリックして、API キーを削除することができます。


署名の作成 
==============

eformsign_signature は、非対称キー方式と楕円曲線暗号(Elliptic curve cryptography)を使用しています。

.. tip:: 
   
   楕円曲線暗号は、公開キー暗号化方式の一つで、データ暗号化デジタル認証など現在もっとも広く使われている暗号化方式です。 


署名の作成方法については、Java、Python、PHPの各言語に分けて説明します。

Java
-------

サーバー時刻を String(UTF-8) に変換し、`API キーの作成 <#apikey>`__\で作成された private key で署名した後、署名したデータを hex string に変換します。

.. note:: 

   署名アルゴリズムは SHA256withECDSA を使用します。


Javascript(Node.JS)
------------------------

Jsrsasign(https://kjur.github.io/jsrsasign/) npm 종속성이 설치되어 있어야 합니다.

.. code:: Javascript

   npm install jsrsasign




Python
-------

キーフォーマット処理用のライブラリーを使用する必要があります。作業を開始する前に、次のコマンドを実行してライブラリーをインストールしてください。

.. code:: python

   pip install https://github.com/warner/python-ecdsa/archive/master.zip


PHP
-------

PHP 예제를 사용하려면 PHP OpenSSL 라이브러리가 설치되어 있어야 하며, 次の例題の keycheck.inc.php、test.php ファイルを同じパスに保存してから例題を実行してください。


例題
---------

次は各言語の例題です。


.. note:: 

   execution_time は long タイプを使用しています。そのため、execution_time を入力する際は Access Token 発行時に確認した時間の次に L を追加してください。  



.. code-tabs::

    .. code-tab:: java
        :title: Java

        import java.security.KeyFactory;
        import java.security.spec.PKCS8EncodedKeySpec;
        import java.security.PrivateKey;
        import java.security.Signature;
         
        //private key
        String privateKeyHexStr = "発行したprivate key(String)";    //会社アカウントのコネクト>API KeyのPrivate key値を入力
        KeyFactory keyFact = KeyFactory.getInstance("EC");
        PKCS8EncodedKeySpec psks8KeySpec = new PKCS8EncodedKeySpec(new BigInteger(privateKeyHexStr,16).toByteArray());
        PrivateKey privateKey = keyFact.generatePrivate(psks8KeySpec);
         
        //execution_time サーバー時刻
        //long execution_time = new Date().getTime();
        long execution_time = 1611537340731L;     //Access_token作成のときに発行したexecute_timeをこちらに入力。longタイプのため、作成された時間の次にLを追加     
        String execution_time_str = String.valueOf(execution_time);
         
        //eformsign_signature作成
        Signature ecdsa = Signature.getInstance("SHA256withECDSA");
        ecdsa.initSign(privateKey);
        ecdsa.update(execution_time_str.getBytes("UTF-8"));
        String eformsign_signature = new BigInteger(ecdsa.sign()).toString(16);
         
         
        //現在時刻および現在時刻の署名値
        System.out.print("execution_time : "+execution_time);
        System.out.print("eformsign_signature : "+eformsign_signature);


    .. code-tab:: javascript
        :title: Javascript(Node.JS)

        const rs = require('jsrsasign');


        // User-Data-Here
        const execution_time  = Date.now()+"";
        const privateKeyHex = "発行したprivate key(String)";

        // User-Data-Here
        var privateKey = rs.KEYUTIL.getKeyFromPlainPrivatePKCS8Hex(privateKeyHex);

        // Sign
        var s_sig = new rs.Signature({alg: 'SHA256withECDSA'});
        s_sig.init(privateKey);
        s_sig.updateString(execution_time);
        var signature = s_sig.sign();
        console.log('data:', execution_time);
        console.log('eformsign_signature:', signature);



    .. code-tab:: python
        :title: Python

        import hashlib
        import binascii
         
        from time import time
        from ecdsa import SigningKey, VerifyingKey, BadSignatureError
        from ecdsa.util import sigencode_der, sigdecode_der
         
        # private key
        privateKeyHex = "発行したprivate key(String)"
        privateKey = SigningKey.from_der(binascii.unhexlify(privateKeyHex))
         
        # execution_time - サーバー時刻
        execution_time = int(time() * 1000)
          
        # eformsign_signature作成
        eformsign_signature = privateKey.sign(execution_time.encode('utf-8'), hashfunc=hashlib.sha256, sigencode=sigencode_der)
          
        # 現在時刻および現在時刻の署名値
        print("execution_time : " + execution_time)
        print("eformsign_signature : " + binascii.hexlify(signature).decode('utf-8'))

    .. code-tab:: php
        :title: PHP - keycheck.inc.php

        <?php
        namespace eformsignECDSA;
  
        class PublicKey
        {
          
            function __construct($str)
            {
                $pem_data = base64_encode(hex2bin($str));
                $offset = 0;
                $pem = "-----BEGIN PUBLIC KEY-----\n";
                while ($offset < strlen($pem_data)) {
                    $pem = $pem . substr($pem_data, $offset, 64) . "\n";
                    $offset = $offset + 64;
                }
                $pem = $pem . "-----END PUBLIC KEY-----\n";
                $this->openSslPublicKey = openssl_get_publickey($pem);
            }
        }

        class PrivateKey
        {         

            function __construct($str)
            {
                $pem_data = base64_encode(hex2bin($str));
                $offset = 0;
                $pem = "-----BEGIN EC PRIVATE KEY-----\n";
                while ($offset < strlen($pem_data)) {
                    $pem = $pem . substr($pem_data, $offset, 64) . "\n";
                    $offset = $offset + 64;
                }
                $pem = $pem . "-----END EC PRIVATE KEY-----\n";
                $this->openSslPrivateKey = openssl_get_privatekey($pem);
            }
        }


        function getNowMillisecond()
        {
          list($microtime,$timestamp) = explode(' ',microtime());
          $time = $timestamp.substr($microtime, 2, 3);
          
          return $time;
        }
         
         
        function Sign($message, $privateKey)
        {
            openssl_sign($message, $signature, $privateKey->openSslPrivateKey, OPENSSL_ALGO_SHA256);
            return $signature;
        }
        ?>

    .. code-tab:: php
        :title: PHP - test.php

        <?php
        require_once __DIR__ . '/keycheck.inc.php';
 
        use eformsignECDSA\PrivateKey;
         
         
        define('PRIVATE_KEY', '発行したprivate key(String)');
         
         
        //private key設定
        $privateKey = new PrivateKey(PRIVATE_KEY);
         
         
        //execution_time - サーバー時刻
        $execution_time = eformsignECDSA\getNowMillisecond();
         
         
        //eformsign_signature作成
        $signature = eformsignECDSA\Sign(execution_time, $privateKey);
         
         
        //現在時刻および現在時刻の署名値
        print 'execution_time : ' . execution_time . PHP_EOL;
        print 'eformsign_signature : ' . bin2hex($signature) . PHP_EOL;
        ?>







API 提供リスト
======================

eformsign API は、署名を作成するための API と文書の作成および処理のための API、メンバーおよびグループ管理のための API で構成されています。



署名を作成するための API
-------------------------

署名を作成するために、まず `Access Token API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_\ を使用してください。 

``POST``: `Access Token 発行 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_\


Access Token API についての詳しい説明は 
`次 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`__\ で確認することができます。

.. caution:: 
   
   署名の作成には30秒の時間制限があります。30秒以内に署名を作成し、トークンを取得する必要があります。 
   また、サーバー時刻と現在時刻が一致しない場合があります。Access Token API を呼び出し、受信した応答メッセージの "execution_time" を確認してください。

   .. code:: JSON

      { "code": "4000002", "ErrorMessage": "The validation time has expired.",     "execution_time": 1611538409405 }

   `次 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`__\ の例題の位置にも "execution_time" を入力してください。
   
   |image5| 

   Access Token は、メンバーの権限に応じて取得することができます。メンバーに対する Access Token を取得するためには、以下のように "execution_time" と一緒に "member_id" を入力してください。 
   
   |image6| 


   その後、発行した API を実行すると、Access Token が発行され、次のようなタイプの応答を受信することができます。

   .. code:: JSON

      { "api_key": { "name": "アプリケーション_", "alias": "テスト用", "company": { "company_id": "dec5418e58694d90a65d6c38e3d226db", "name": "サンプルデモ", "api_url": "https://kr-api.eformsign.com" } }, "oauth_token": { "expires_in": 3600, "token_type": "JWT", "refresh_token": "8fd0a3c1-44dc-4a03-96ad-01fa34cd159c", "access_token": "eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJlZm9ybXNpZ24uaWFtIiwiY29udGV4dCI6eyJjbGllbnRJZCI6IjY4MDk0ZWVhMjVhZjRhNjI5ZTI4ZGU5Y2ZlYzRlYmZjIiwiY2xpZW50S2V5IjoiZTNiM2IzZTUtMGEzMS00NTE1LWE5NzEtN2M4Y2FlNDI4NzZmIiwibWFuYWdlbWVudElkIjoiMzRhYWI4MDBjMmEwNDQwNThmZDRlZjc5OGFlY2RlY2EiLCJzY29wZXMiOiJzbWFydF9lZm9ybV9zY29wZSIsInR5cGUiOiJ1c2VyIiwidXNlck5hbWUiOiIzMmIzZDRmOC00MjdkLTRjZjQtOTZiYS1mYzAxNjIxNWRkNDciLCJ1c2VySWQiOiJhNTEyNGVkNmU2M2Y0OTMzOGJlOTA0MjVhNjFkYjlmNSIsInJlZnJlc2hUb2tlbiI6IjhmZDBhM2MxLTQ0ZGMtNGEwMy05NmFkLTAxZmEzNGNkMTU5YyJ9LCJjbGFpbSI6eyJjb21wYW55X2lkIjoiZGVjNTQxOGU1ODY5NGQ5MGE2NWQ2YzM4ZTNkMjI2ZGIiLCJhY2Nlc3Nfa2V5IjoiMzJiM2Q0ZjgtNDI3ZC00Y2Y0LTk2YmEtZmMwMTYyMTVkZDQ3In0sImV4cCI6MTYxMTU0MjIzNiwiaWF0IjoxNjExNTM4NjM2fQ.BltoXXBSabjXfpyLsZik9OZTE5XtLqe9lguMmJ_qfwZN1NyoVoxDqA5y1-_TLis7FvvNjfI1eegOroCZDZPFyXRaBxAj0CW8TijVjbhliJBuccHFyKXaJxmo_GMmTHYtxNNB1SUgLeFIrYROnpQndU8J7ZkfPDgYGwh1YSx-5s4" } }





.. caution:: 
   
   発行した API キーは、 `次 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`__\ の位置にある **Authorize** ボタン（|image4|）をクリックして登録してください。ただし、API キー値には**必ず Base64** でエンコードした 文字列を入力する必要があります。https://www.base64encode.org/ に接続し、発行した API キーを入力してエンコードされたテキストに変換してから入力してください。


.. note:: 
  
   Access Token API の **Authorize** ボタンには API キー値を入力する必要があります。


-------------------


文書の作成および処理のための API
--------------------------------

署名を作成すると、次の文書 API を使用して文書の新規作成や文書情報の照会ができ、完了文書のファイル（文書の PDF ファイル、監査証跡証明書）や添付ファイルをダウンロードすることができます。 


.. caution:: 

   本文書の API を使用するためにはまず Access Token の取得が必要です。`Access Token API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_\で取得した Access Token を`次 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/>`__\ のパスにある **Authorize** ボタン (|image4|) を押して登録してください。 


.. note:: 

   文書 API の **Authorize** ボタンには API キー値を入力する必要があります。

現在提供している `文書 API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/document>`_\は次のとおりです。



``POST``: `文書の新規作成_内部受信者 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-documents>`_\ 

``POST``: `文書の新規作成_外部受信者 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/post-api-documents-external>`_\ 

``GET``: `文書情報の照会 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/get-api-documents-DOCUMENT_ID>`_\

``GET``: `文書ファイルのダウンロード_文書 PDFおよび監査証跡証明書 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/get-api-documents-DOCUMENT_ID-download_files>`_\

``GET``: `添付ファイルのダウンロード <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/get-api-documents-DOCUMENT_ID-download_attach_files>`_\ 

``GET``: `文書リストの照会 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-documents>`_\ 

``DELETE``: `文書の削除 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/delete-api-documents>`_\ 

``POST``: `外部受信者に対する再依頼 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-documents-document_id-re_request_outsider>`_\ 

``GET``: `作成可能なテンプレートリストの照会 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-forms>`_\  

``POST``: `文書の一括生成 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-forms-mass_documents%3Ftemplate_id%3D-form_id>`_\


------------------------


メンバーおよびグループ管理のための API
---------------------------------------------

APIを使用してメンバーおよびグループを管理することができます。メンバーおよびグループのリストを照会することができ、メンバーおよびグループを修正、削除することができます。


.. caution:: 

   本文書の API を使用するためにはまず Access Token の取得が必要です。`Access Token API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0_auth>`_\で取得した Access Token を`次 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/>`__\ のパスにある **Authorize** ボタン (|image4|) を押して登録してください。 


.. note:: 
  
   APIの **Authorize** ボタンには API キー値を入力する必要があります。 


現在提供している `メンバーおよびグループ管理 API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/members>`_\は次のとおりです。


メンバー管理 API
^^^^^^^^^^^^^^^^^


``GET``: `メンバーリストの照会 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-members>`_\   

``PATCH``: `メンバー修正 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/patch-api-members-member_id>`_\  

``DELETE``: `メンバー削除 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/delete-api-members-member_id>`_\  


グループ管理 API
^^^^^^^^^^^^^^^^^


``GET``: `グループリストの照会 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-groups>`_\  

``POST``: `グループ追加 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-groups>`_\  

``PATCH``: `グループ修正 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/patch-api-groups>`_\  

``DELETE``: `グループ削除 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/delete-api-groups>`_\  



.. note:: 

    各 eformsign API についての詳しい説明は `次 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/>`__\ をご覧ください。






API コード
===================

Open API を使用するときは、次のコードをご参照ください。

API ステータスコード
---------------------

API ステータスコードは、次の通りです。

200
^^^^^^^^

===========  ===============  ===================================
Code         説明              備考
===========  ===============  ===================================
200          成功              成功
===========  ===============  ===================================


202
^^^^^^^^

===========  ===============  ===================================
Code         説明              備考
===========  ===============  ===================================
2020001      PDF作成中          - PDFファイルとしてダウンロードする際、非同期で作成されるため、文書保存後にPDFファイルの作成まで時間が所要 
                                - 数秒から数分後数分後に再要請するとダウンロード可能
===========  ===============  ===================================


400
^^^^^^^^

===========  =====================  ======================================================================
Code         説明                    備考
===========  =====================  ======================================================================
4000001      必須入力値漏れ            API の必須入力値（ヘッダー値またはパラメーター）が入力されていない場合                        
4000002      時間切れ                 API 認証のリクエスト時間が時間切れとなった場合
4000003      API キーが存在しない        削除された API キーまたは入力ミスの場合
4000004      文書が存在しない           間違った文書 ID を入力した場合
4000005      会社が存在しない           会社が削除された場合
===========  =====================  ======================================================================


403
^^^^^^^^

===========  =========================  ==========================================
Code         説明                        備考
===========  =========================  ==========================================
4030001      アクセス権限なし              APIが非活性状態の場合
4030002      Access Token 認証エラー       Access Tokenが正しくない場合
4030003      Refresh Token 認証エラー       Refresh Tokenが正しくない場合
4030004      署名値検証エラー               署名値が正しくない場合
4030005      サポートしないAPI               サポートしないAPIを呼び出した場合
===========  =========================  ==========================================

405
^^^^^^^^

===========  =====================  ===================================
Code         説明                    備考
===========  =====================  ===================================
4050001      サポートしないmethod        サポートしないmethodを呼び出した場合
===========  =====================  ===================================


500
^^^^^^^^

===================  ===============  ===================================
Code                 説明              備考
===================  ===============  ===================================
5000001~5000003      サーバーエラー         サーバーエラーが発生した場合
===================  ===============  ===================================


---------


ステップタイプ
--------------

===========  ===============  ===================================
Type         Code             説明
===========  ===============  ===================================
Start         00               開始
Complete      01               完了
Approval      02               決裁
External      03               外部受信者
Accept        04               内部受信者
Participant   05               参加者
Reviewer      06               検討者
===========  ===============  ===================================


ユーザータイプ
--------------

================  ===============  ===================================
Type               Code             説明
================  ===============  ===================================
メンバー　　          01               メンバーである場合
外部ユーザー          02               メンバーではなく、外部ユーザーである場合
================  ===============  ===================================


ステータスタイプ
--------------------

===========  ===============  ==========================================================
Type          Code             説明
===========  ===============  ==========================================================
下書き保存     00               開始ステップで下書きとして保存した状態
進行中　       01               決裁依頼、外部受信者に依頼、閲覧、内部受信者に依頼
修正中　       02               文書を修正中 (内部受信者、作成者)
完了          03               完了した状態
返戻          04               決済の返戻、内部受信者が返戻、外部受信者が返戻
無効化         05              文書が無効化された状態
無効化依頼     06               内部受信者のステップ
===========  ===============  ==========================================================


アクションタイプ
--------------------

==========================  ===============  ===================================
Type                         Code             説明
==========================  ===============  ===================================
doc_tempsave                 001              下書き保存
doc_create                   002              作成
doc_complete                 003              完了
doc_request_approval         010              決裁の依頼
doc_reject_approval          011              決裁の返戻
doc_accept_approval          012              決裁の承認
doc_cancel                   013              決裁のキャンセル
doc_request_reception        020              内部受信者に依頼
doc_reject_reception         021              内部受信者が返戻
doc_accept_reception         022              内部受信者が承認
doc_accept_tempsave          023              内部受信者が下書き保存
doc_request_outsider         030              外部受信者に依頼
doc_reject_outsider          031              外部受信者が返戻
doc_accept_outsider          032              外部受信者が承認
doc_rerequest_outsider       033              外部受信者に再依頼
doc_open_outsider            034              外部受信者が閲覧
doc_outsider_tempsave        035              外部受信者が下書き保存
doc_request_revoke           040              無効化の依頼
doc_refuse_revoke            041              無効化依頼の拒否
doc_revoke                   042              無効化
doc_update                   043              修正
doc_cancel_update            044              修正のキャンセル
doc_request_reject           045              返戻の依頼
doc_refuse_reject            046              返戻依頼の拒否
doc_request_delete           047              削除の依頼
doc_refuse_delete            048              削除依頼の拒否
doc_delete                   049              削除
doc_complete_send_pdf        050              完了文書をPDFファイルで送信
doc_transfer                 051              移管
oc_request_participant       060              参加者に依頼
doc_reject_participant       061              参加者が返戻
doc_accept_participant       062              参加者が承認
doc_rerequest_participant    063              参加者に再依頼(外部受信者)
doc_open_participant         064              参加者が閲覧(外部受信者)
doc_request_reviewer         070              検討者に依頼
doc_reject_reviewer          071              検討者が返戻
doc_request_reviewer         072              検討者が承認 
doc_rerequest_reviewer       073              検討者に再依頼(外部受信者)
doc_open_review              074              検討者が閲覧(外部受信者)
==========================  ===============  ===================================


詳細ステータスタイプ
-----------------------

==========================  ===============  ===========================================
Type                         Code             説明
==========================  ===============  ===========================================
doc_tempsave                 001              下書き保存 (作成者が下書きとして保存した状態)
doc_create                   002              作成
doc_complete                 003              完了
doc_update                   043              修正
doc_request_delete           047              削除の依頼
doc_delete                   049              削除
doc_request_revoke           040              無効化の依頼
doc_revoke                   042              無効化
doc_request_reject           045              返戻の依頼
doc_request_approval         010              決裁の依頼
doc_accept_approval          012              決裁の承認
doc_reject_approval          011              決裁の返戻
doc_cancel                   013              決裁のキャンセル
doc_request_reception        020              内部受信者に依頼
doc_accept_reception         022              内部受信者が承認
doc_reject_reception         021              内部受信者が返戻
doc_request_outsider         030              外部受信者に依頼
doc_accept_outsider          032              外部受信者が承認
doc_reject_outsider          031              外部受信者が返戻
doc_request_participant      060              参加者に依頼
doc_accept_participant       062              参加者が承認
doc_reject_participant       061              参加者が返戻
doc_request_reviewer         070              検討者に依頼
doc_accept_reviewer          072              検討者が承認 
doc_reject_reviewer          071              検討者が返戻
==========================  ===============  ===========================================


.. |image1| image:: resources/company_id.png
   :width: 600px
.. |image2| image:: resources/column_icon.png
.. |image3| image:: resources/document_id.png
.. |image4| image:: resources/authorize_icon.png
.. |image5| image:: resources/execution_time.png
   :width: 450px
.. |image6| image:: resources/execution_time2.png
   :width: 450px