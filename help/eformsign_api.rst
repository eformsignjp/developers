
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




Python
-------

キーフォーマット処理用のライブラリーを使用する必要があります。作業を開始する前に、次のコマンドを実行してライブラリーをインストールしてください。

.. code:: python

   pip install https://github.com/warner/python-ecdsa/archive/master.zip


PHP
-------

次の例題の keycheck.inc.php、test.php ファイルを同じパスに保存してから例題を実行してください。


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

eformsign API は、署名を作成するための API と文書の作成や処理のための API で構成されています。

署名を作成するための API
-------------------------

署名を作成するために、まず `Access Token API <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth>`_\ を使用してください。 

``POST``: `/api_auth/access_token <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth#/eformsign/post-api_auth-access_token>`_\  Access Token 発行


Access Token API についての詳しい説明は 
`次 <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth>`__\ で確認することができます。

.. caution:: 
   
   署名の作成には30秒の時間制限があります。30秒以内に署名を作成し、トークンを取得する必要があります。 
   また、サーバー時刻と現在時刻が一致しない場合があります。Access Token API を呼び出し、受信した応答メッセージの"execution_time"を確認してください。

   .. code:: JSON

      { "code": "4000002", "ErrorMessage": "The validation time has expired.",     "execution_time": 1611538409405 }

   `次 <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth>`__\ の例題の位置にも "execution_time" を入力してください。
   
   |image5| 

   Access Token は、メンバーの権限に応じて取得することができます。メンバーに対する Access Token を取得するためには、以下のように "execution_time" と一緒に "member_id" を入力してください。 
   
   |image6| 


   その後、発行した API を実行すると、Access Token が発行され、次のようなタイプの応答を受信することができます。

   .. code:: JSON

      { "api_key": { "name": "アプリケーション_", "alias": "テスト用", "company": { "company_id": "dec5418e58694d90a65d6c38e3d226db", "name": "サンプルデモ", "api_url": "https://kr-api.eformsign.com" } }, "oauth_token": { "expires_in": 3600, "token_type": "JWT", "refresh_token": "8fd0a3c1-44dc-4a03-96ad-01fa34cd159c", "access_token": "eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJlZm9ybXNpZ24uaWFtIiwiY29udGV4dCI6eyJjbGllbnRJZCI6IjY4MDk0ZWVhMjVhZjRhNjI5ZTI4ZGU5Y2ZlYzRlYmZjIiwiY2xpZW50S2V5IjoiZTNiM2IzZTUtMGEzMS00NTE1LWE5NzEtN2M4Y2FlNDI4NzZmIiwibWFuYWdlbWVudElkIjoiMzRhYWI4MDBjMmEwNDQwNThmZDRlZjc5OGFlY2RlY2EiLCJzY29wZXMiOiJzbWFydF9lZm9ybV9zY29wZSIsInR5cGUiOiJ1c2VyIiwidXNlck5hbWUiOiIzMmIzZDRmOC00MjdkLTRjZjQtOTZiYS1mYzAxNjIxNWRkNDciLCJ1c2VySWQiOiJhNTEyNGVkNmU2M2Y0OTMzOGJlOTA0MjVhNjFkYjlmNSIsInJlZnJlc2hUb2tlbiI6IjhmZDBhM2MxLTQ0ZGMtNGEwMy05NmFkLTAxZmEzNGNkMTU5YyJ9LCJjbGFpbSI6eyJjb21wYW55X2lkIjoiZGVjNTQxOGU1ODY5NGQ5MGE2NWQ2YzM4ZTNkMjI2ZGIiLCJhY2Nlc3Nfa2V5IjoiMzJiM2Q0ZjgtNDI3ZC00Y2Y0LTk2YmEtZmMwMTYyMTVkZDQ3In0sImV4cCI6MTYxMTU0MjIzNiwiaWF0IjoxNjExNTM4NjM2fQ.BltoXXBSabjXfpyLsZik9OZTE5XtLqe9lguMmJ_qfwZN1NyoVoxDqA5y1-_TLis7FvvNjfI1eegOroCZDZPFyXRaBxAj0CW8TijVjbhliJBuccHFyKXaJxmo_GMmTHYtxNNB1SUgLeFIrYROnpQndU8J7ZkfPDgYGwh1YSx-5s4" } }





.. caution:: 
   
   発行した API キーは、 `次 <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth>`__\ の位置にある **Authorize** ボタン（|image4|）をクリックして登録してください。ただし、API キー値には**必ず Base64** でエンコードした 文字列を入力する必要があります。https://www.base64encode.org/ に接続し、発行した API キーを入力してエンコードされたテキストに変換してから入力してください。


.. note:: 
  
   API の **Authorize** ボタンには API キー値を入力する必要があります。



文書の作成・処理のための API
--------------------------------

署名を作成すると、次の文書 API を使用して文書の新規作成や文書情報の照会ができ、完了文書のファイル（文書の PDF ファイル、監査証跡証明書）や添付ファイルをダウンロードすることができます。 


.. caution:: 

   本文書の API を使用するためにはまず Access Token の取得が必要です。`Access Token API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0_auth>`_\で取得した Access Token を`次 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0>`__\ のパスにある **Authorize** ボタン (|image4|) を押して登録してください。 


.. note:: 

   API の **Authorize** ボタンには API キー値を入力する必要があります。

現在提供している `文書 API <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_general>`_\は次のとおりです。


``POST``: `/api/documents <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-documents>`_\  文書の新規作成（内部受信者）

``POST``: `/api/documents/external <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/post-api-documents-external>`_\  文書の新規作成（外部受信者）

``GET``: `/api/documents/{document_id} <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/get-api-documents-DOCUMENT_ID>`_\  文書情報の照会

``GET``: `/api/documents/{document_id}/download_files <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/get-api-documents-DOCUMENT_ID-download_files>`_\  文書ファイルのダウンロード（PDF ファイル、監査証跡証明書）

``GET``: `/api/documents/{document_id}/download_attach_files <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/eformsign/get-api-documents-DOCUMENT_ID-download_attach_files>`_\  添付ファイルのダウンロード

``GET``: `/api/documents <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-documents>`_\  文書リストの照会

``DELETE``: `/api/documents <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/delete-api-documents>`_\  文書の削除

``POST``: `/api/documents/{document_id}/re_request_outsider <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-documents-document_id-re_request_outsider>`_\  外部受信者に対する再依頼


メンバーおよびグループ管理のための API
----------------------------------

APIを使用してメンバーおよびグループを管理することができます。メンバーおよびグループのリストを照会することができ、メンバーおよびグループを追加、修正、削除することができます。


.. caution:: 

   本文書の API を使用するためにはまず Access Token の取得が必要です。`Access Token API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0_auth>`_\で取得した Access Token を`次 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0>`__\ のパスにある **Authorize** ボタン (|image4|) を押して登録してください。 


.. note:: 
  
   APIの **Authorize** ボタンには API キー値を入力する必要があります。 


現在提供している `メンバーおよびグループ管理 API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0>`_\は次のとおりです。


``GET``: `/api/members <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-members>`_\  멤버 목록 조회

``POST``: `/api/members <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-members>`_\  멤버 추가

``PATCH``: `/api/members/{member_id} <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/patch-api-members-member_id>`_\  멤버 수정

``DELETE``: `/api/members/{member_id} <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/delete-api-members-member_id>`_\  멤버 삭제

``GET``: `/api/groups <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-groups>`_\  그룹 목록 조회

``POST``: `/api/groups <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-groups>`_\  그룹 추가

``PATCH``: `/api/groups/{group_id} <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/patch-api-groups>`_\  그룹 수정

``DELETE``: `/api/groups <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/delete-api-groups>`_\  그룹 삭제


テンプレート管理のための API
----------------------------------

APIを使用して作成可能なテンプレートリストの照会、テンプレートを使った일괄 작성ができます。


.. caution:: 

   本文書の API を使用するためにはまず Access Token の取得が必要です。`Access Token API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0_auth>`_\で取得した Access Token を`次 <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0>`__\ のパスにある **Authorize** ボタン (|image4|) を押して登録してください。 



.. note:: 
  
   APIの **Authorize** ボタンには API キー値を入力する必要があります。 


現在提供している `テンプレート API <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0>`_\は次のとおりです。


``GET``: `/api/forms <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/get-api-forms>`_\  作成可能なテンプレートリストの照会

``POST``: `/api/forms/mass_documents?template_id={{form_id}} <https://app.swaggerhub.com/apis-docs/eformsign_api/eformsign_API_2.0/2.0#/default/post-api-forms-mass_documents%3Ftemplate_id%3D-form_id>`_\  文書の일괄 작성


各 eformsign API についての詳しい説明は 
`次 <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_general>`__\ をご覧ください。



API コード
===================

Open API を使用するときは、次のコードをご参照ください。

API ステータスコード
---------------------

API ステータスコードは、次の通りです。

200
'''''''

===========  ===============  ===================================
Code         説明              備考
===========  ===============  ===================================
200          成功              成功
===========  ===============  ===================================


202
'''''''

===========  ===============  ===================================
Code         説明              備考
===========  ===============  ===================================
2020001      PDF作成中          - PDFファイルとしてダウンロードする際、非同期で作成されるため、文書保存後にPDFファイルの作成まで時間が所要 
                                - 数秒から数分後数分後に再要請するとダウンロード可能
===========  ===============  ===================================


400
'''''''

===========  =====================  ==========================================================================
Code         説明                    備考
===========  =====================  ==========================================================================
4000001      必須入力値漏れ         API の必須入力値（ヘッダー値またはパラメーター）が入力されていない場合                        
4000002      時間切れ               API 認証のリクエスト時間が時間切れとなった場合
4000003      API キーが存在しない   削除された API キーまたは入力ミスの場合
4000004      文書が存在しない       間違った文書 ID を入力した場合
4000005      会社が存在しない       会社が削除された場合
===========  =====================  ==========================================================================


403
'''''''

===========  =========================  ==========================================
Code         説明                        備考
===========  =========================  ==========================================
4030001      アクセス権限なし              APIが非活性状態の場合
4030002      Access Token 認証エラー       Access Tokenが正しくない場合
4030003      Refresh Token 認証エラー       Refresh Tokenが正しくない場合
4030004      署名値検証エラー               署名値が正しくない場合
4030005      サポートしない API              サポートしないAPIを呼び出した場合
===========  =========================  ==========================================

405
'''''''

===========  =====================  ===================================
Code         説明                    備考
===========  =====================  ===================================
4050001      サポートしないmethod        サポートしないmethodを呼び出した場合
===========  =====================  ===================================


500
'''''''

===================  ===============  ===================================
Code                 説明              備考
===================  ===============  ===================================
5000001~5000003      サーバーエラー         サーバーエラーが発生した場合
===================  ===============  ===================================




Step タイプ
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


User タイプ
--------------

================  ===============  ===================================
Type               Code             説明
================  ===============  ===================================
内部受信者          01               内部メンバーである場合
外部受信者          02               内部メンバーではなく、外部受信者である場合
================  ===============  ===================================

Status タイプ
--------------

======================  ===============  ===================================
Type                     Code             説明
======================  ===============  ===================================
doc_tempsave             001              下書き保存
doc_create               002              作成
doc_complete             003              完了
doc_update               043              修正
doc_request_delete       047              削除の依頼
doc_delete               049              削除
doc_request_revoke       040              無効化の依頼
doc_revoke               041              無効化
doc_request_reject       045              返戻の依頼
doc_request_approval     010              決裁の依頼
doc_accept_approval      012              決裁の承認
doc_reject_approval      011              決裁の返戻
doc_cancel               013              決裁のキャンセル
doc_request_reception    020              内部受信者に依頼
doc_accept_reception     022              内部受信者が承認
doc_reject_reception     021              内部受信者が返戻
doc_request_outsider     030              外部受信者に依頼
doc_accept_outsider      032              外部受信者が承認
doc_reject_outsider      031              外部受信者が返戻
======================  ===============  ===================================


Status タイプ
--------------

===========  ===============  ==================================================
Type          Code             説明
===========  ===============  ==================================================
下書き　　       00               開始ステップで下書きとして保存した状態
進行中　       01               決裁依頼、外部受信者に依頼、閲覧、内部受信者に依頼
修正中　       02               文書を修正中(メンバー、最初の作成者)
完了          03               完了した状態
返戻          04               決済の返戻、内部受信者が返戻、外部受信者が返戻
無効化         05              文書が無効化された状態
無効化依頼     06               内部受信者の段階
===========  ===============  ==================================================


Action タイプ
--------------

==========================  ===============  ===================================
Type                         Code             설명
==========================  ===============  ===================================
doc_tempsave                 001              문서 임시 저장
doc_create                   002              문서 생성
doc_complete                 003              문서 최종 완료
doc_request_approval         010              결재 요청
doc_reject_approval          011              결재 반려
doc_accept_approval          012              결재 승인
doc_cancel                   013              결재 요청 취소
doc_request_reception        020              내부 멤버 요청
doc_reject_reception         021              내부 멤버 반려
doc_accept_reception         022              내부 멤버 승인
doc_accept_tempsave          023              내부 멤버 임시 저장
doc_request_outsider         030              외부수신자 요청
doc_reject_outsider          031              외부수신자 반려
doc_accept_outsider          032              외부수신자 승인
doc_rerequest_outsider       033              외부수신자 재요청
doc_open_outsider            034              외부수신자 열람
doc_outsider_tempsave        035              외부수신자 임시 저장
doc_request_revoke           040              문서 취소 요청
doc_refuse_revoke            041              문서 취소 요청 거절
doc_revoke                   042              문서 취소
doc_update                   043              문서 수정
doc_cancel_update            044              문서 수정 취소
doc_request_reject           045              문서 반려 요청
doc_refuse_reject            046              문서 반려 요청 거절
doc_request_delete           047              문서 삭제 요청
doc_refuse_delete            048              문서 삭제 요청 거절
doc_delete                   049              문서 삭제
doc_complete_send_pdf        050              완료 문서 PDF 전송
doc_transfer                 051              문서 이관
doc_request_participant      060              参加者 요청
doc_reject_participant       061              参加者 반려
doc_accept_participant       062              参加者 승인
doc_rerequest_participant    063              参加者 재요청(외부자)
doc_open_participant         064              参加者 문서 열람(외부자)
doc_request_reviewer         070              検討者 요청
doc_reject_reviewer          071              検討者 반려
doc_request_reviewer         072              検討者 승인 
doc_rerequest_reviewer       073              検討者 재요청(외부자)
doc_open_review              074              検討者 문서 열람(외부자)
==========================  ===============  ===================================

======================      ===============  ===================================
Type                         Code             説明
======================      ===============  ===================================
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
==========================  ===============  ====================================


詳細 Status タイプ
-----------------------

==========================  ===============  ===================================
Type                         Code             説明
==========================  ===============  ===================================
doc_tempsave                 001              下書き保存
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
==========================  ===============  ===================================


.. |image1| image:: resources/company_id.png
   :width: 600px
.. |image2| image:: resources/column_icon.png
.. |image3| image:: resources/document_id.png
.. |image4| image:: resources/authorize_icon.png
.. |image5| image:: resources/execution_time.png
   :width: 450px
.. |image6| image:: resources/execution_time2.png
   :width: 450px