----------------------------
eformsign Webhook の使い方
----------------------------
eformsign でイベントが発生した時、そのイベント情報を顧客のシステム/サービスに通知する機能です。Webhook を設定すると、顧客の Webhook endpoint にそのイベント情報を HTTP POST 形式で通知します。

.. tip:: 

   Webhook の endpoint とは、顧客の client callback URL を意味します。Open API を持続的に呼び出して変更内容をチェックする方式（polling）に比べ、不要な呼び出しをせず eformsign 上のイベントについての情報を取得できます。


始めてみよう 
===============


.. _webhook:

Webhook キーの発行
--------------------

1. eformsign に代表管理者としてログインし、左側のメニューから **「コネクト」 > 「API/Webhook」** ページに移動します。 

.. image:: resources/apikey1.PNG
    :width: 700
    :alt: コネクト > API/Webhook メニューの位置


2. **「Webhook」** タブを選択し、**Webhookの作成** ボタンをクリックします。

.. image:: resources/webhook2.PNG
    :width: 700
    :alt: 「Webhookの作成」ボタン


3. **「Webhook の作成」** ポップアップに名前、Webhook の URL、アクティブ状態、適用対象を選択し、「登録」ボタンをクリックします。

.. image:: resources/webhook3.PNG
    :width: 700
    :alt: 「Webhookの作成」ポップアップ


4. 作成された Webhook リストから **「キーを表示」** ボタンをクリックして Webhook の公開鍵を確認します。

.. image:: resources/webhook4.PNG
    :width: 700
    :alt: 「キーを表示」ボタンの位置

.. image:: resources/webhook5.PNG
    :width: 700
    :alt: Webhookのキーを確認 



.. note:: 

    **「キーを再発行」** ボタンをクリックすると、その Webhook の公開鍵が再発行され、以前のキーは使用できなくなります。

.. note:: **Webhook 情報の修正**

    作成された Webhook リストから **修正** ボタンをクリックして Webhook 情報を変更することができます。


.. note:: **Webhook の削除**

    作成された Webhook リストから **削除** ボタンをクリックして Webhook を削除することができます。    



5. 作成された Webhook リストからテスト ボタンをクリックすると、テスト Webhook を送信して結果値を返します。

.. image:: resources/webhook6.PNG
    :width: 700
    :alt: Webhook テスト 確認 

次はテストのための json ファイルです。


.. code:: JSON

    {
    "webhook_id" : "Webhook ID",
    "webhook_name" : "Webhookの名前",
    "company_id" : "会社ID",
    "event_type" : “document”,
    "document" : {
      "id" : “test_doc_id”,
       "template_id" : “test_template_id”,
       "template_version" : “1”,
       "document_history_id" : “test_document_history_id”,
       "doc_status" : “doc_create”,
       "editor_id" : "ユーザーID",
       "updated_date" : "現在時間(UTC Long)"
    }
    }
    Test URL : WebhookのURL



署名の登録 
==============


署名の登録方法については、Java、Python、PHPの各言語に分けて説明します。

Java
-------

eformsign サーバーから送られたイベント情報を `Webhook キーの発行 <#webhook>`__\で発行した公開鍵で検証し、 eformsign で正常に呼び出したイベントであるか検証します。 

.. note:: 
  署名のアルゴリズムは SHA256withECDSA を使用します。


Python
-------

キーフォーマット処理用のライブラリーを使用する必要があります。作業を開始する前に、次のコマンドを実行してライブラリーをインストールしてください。

.. code:: python

   pip install https://github.com/warner/python-ecdsa/archive/master.zip


PHP
-------

次の例題の keycheck.inc.php、test.php ファイルを同じパスに保存してから例題を実施してください。


例題
-----------

次は各言語の例題です。

.. code-tabs::

    .. code-tab:: java
        :title: java

        import java.io.*;
        import java.math.BigInteger;
        import java.security.*;
        import java.security.spec.X509EncodedKeySpec;
         
        ....
        /**
         *  requestでheaderとbodyを読み取ります。
         *
         */
         
         
        //1. get eformsign signature
        //eformsignSignatureはrequest headerに含まれています。
        String eformsignSignature = request.getHeader("eformsign_signature");
         
         
        //2. get request body data
        // eformsign signature 検証のため body のデータを String に変換します。
        String eformsignEventBody = null;
        StringBuilder stringBuilder = new StringBuilder();
        BufferedReader bufferedReader = null;
         
        try {
            InputStream inputStream = request.getInputStream();
            if (inputStream != null) {
                bufferedReader = new BufferedReader(new InputStreamReader(inputStream));
                char[] charBuffer = new char[128];
                int bytesRead = -1;
                while ((bytesRead = bufferedReader.read(charBuffer)) > 0) {
                    stringBuilder.append(charBuffer, 0, bytesRead);
                }
            }
         } catch (IOException ex) {
            throw ex;
         } finally {
            if (bufferedReader != null) {
                try {
                    bufferedReader.close();
                } catch (IOException ex) {
                    throw ex;
                }
            }
         }
        eformsignEventBody = stringBuilder.toString();
         
         
         
         
        //3. publicKey設定
        String publicKeyHex = "発行したPublic Key(String)";
        KeyFactory publicKeyFact = KeyFactory.getInstance("EC");
        X509EncodedKeySpec x509KeySpec = new X509EncodedKeySpec(new BigInteger(publicKeyHex,16).toByteArray());
        PublicKey publicKey = publicKeyFact.generatePublic(x509KeySpec);
         
        //4. verify
        Signature signature = Signature.getInstance("SHA256withECDSA");
        signature.initVerify(publicKey);
        signature.update(eformsignEventBody.getBytes("UTF-8"));
        if(signature.verify(new BigInteger(eformsignSignature,16).toByteArray())){
            //verify success
            System.out.println("verify success");
            /*
             * ここでイベントに応じた処理を行います。
             */
        }else{
            //verify fail
            System.out.println("verify fail");
        }


    .. code-tab:: python
        :title: Python

        import hashlib
    		import binascii
    		 
    		from ecdsa import VerifyingKey, BadSignatureError
    		from ecdsa.util import sigencode_der, sigdecode_der
    		from flask import request
    		 
    		 
    		...
    		# requestでheaderとbodyを読み取ります。
    		# 1. get eformsign signature
    		# eformsignSignatureはrequest headerに含まれています。
    		eformsignSignature = request.headers['eformsign_signature']
    		 
    		 
    		# 2. get request body data
    		# eformsign signature検証のためbodyのデータをStringに変換します。
    		data = request.json
    		 
    		 
    		# 3. publicKey設定
    		publicKeyHex = "発行したpublic key"
    		publickey = VerifyingKey.from_der(binascii.unhexlify(publicKeyHex))
    		 
    		 
    		# 4. verify
    		try:
    		    if publickey.verify(eformsignSignature, data.encode('utf-8'), hashfunc=hashlib.sha256, sigdecode=sigdecode_der):
    		        print("verify success")
    		        # ここでイベントに応じた処理を行います。
    		except BadSignatureError:
    		    print("verify fail")

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
         
        function Verify($message, $signature, $publicKey)
        {
            return openssl_verify($message, $signature, $publicKey->openSslPublicKey, OPENSSL_ALGO_SHA256);
        }
        ?>

    .. code-tab:: php
        :title: PHP - test.php

        <?php
        require_once __DIR__ . '/keycheck.inc.php';
        use eformsignECDSA\PublicKey;
         
        define('PUBLIC_KEY', '発行したpublic keyを入力してください。');
        ...
        /*
         *  request で header と body を読み取ります。
         *
         */
         
         
        //1. get eformsign signature
        //eformsignSignatureはrequest headerに含まれています。
        $eformsignSignature = $_SERVER['HTTP_eformsign_signature'];
         
         
        //2. get request body data
        // eformsign signature検証のためbodyのデータを読み取ります。
        $eformsignEventBody = json_decode(file_get_contents('php://input'), true);
         
         
        //3. publicKey設定
        $publicKey = new PublicKey(PUBLIC_KEY);
         
         
        //4. verify
        $ret = - 1;
        $ret = eformsignECDSA\Verify(MESSAGE, $eformsignSignature, $publicKey);
          
        if ($ret == 1) {
            print 'verify success' . PHP_EOL;
            /*
             * ここでイベントに応じた処理を行います。
             */
        } else {
            print 'verify fail' . PHP_EOL;
        }
         ...
          
        ?>



テスト
==========================

生成した eformsign_signature をテストしてみましょう。 

次の eformsign_signature の生成および検証用サンプルは、Open API または Webhook の署名値を生成および検証するためのテストサンプルのソースコードです。

.. note::

   サンプルキーを使用しているため、実際の環境では正常動作しません。例題で生成した署名値の検証用途としてのみご利用ください。


Java
-------

次のサンプルキーで署名および検証テストを行ってください。


Python
-------

キーフォーマット処理用のライブラリーを使用する必要があります。作業を開始する前に、次のコマンドを実行してライブラリーをインストールしてください。

.. code:: python

   pip install https://github.com/warner/python-ecdsa/archive/master.zip


PHP
-------

次の例題の keycheck.inc.php、test.php ファイルを同じパスに保存してから例題を実施してください。



例題
-----------

次は各言語のテストキーと例題です。


.. code-tabs::

    .. code-tab:: java
        :title: Java

        String privateKeyHex = "3041020100301306072a8648ce3d020106082a8648ce3d0301070427302502010104207eae51d5e4272ebb3fe2701d25026a8c2850965981fb2efa68c8db48b32ede07";
        String publicKeyHex = "3059301306072a8648ce3d020106082a8648ce3d030107034200045ac8a472cee38601e99b2a2d731c958e738eee1ee6aca28f6f5637f231e9a8444f3cb80d9ce6c5bace1d0e71167673ff81743e0ea811ebd999f2f314f1d0a676";     //private key      
        KeyFactory privateKeyFact = KeyFactory.getInstance("EC");
        PKCS8EncodedKeySpec psks8KeySpec = new PKCS8EncodedKeySpec(new BigInteger(privateKeyHex,16).toByteArray());
        PrivateKey privateKey = privateKeyFact.generatePrivate(psks8KeySpec);
         
        //signature
        String testData = "{\"test\":\"signature test\"}";
        Signature ecdsa = Signature.getInstance("SHA256withECDSA");
        ecdsa.initSign(privateKey);
        ecdsa.update(testData.getBytes("UTF-8"));
        String eformsign_signature = new BigInteger(ecdsa.sign()).toString(16);
        System.out.println("data : "+testData);
        System.out.println("eformsign_signature : "+eformsign_signature);
         
        //public key
        KeyFactory publicKeyFact = KeyFactory.getInstance("EC");
        X509EncodedKeySpec x509KeySpec = new X509EncodedKeySpec(new BigInteger(publicKeyHex,16).toByteArray());
        PublicKey publicKey = publicKeyFact.generatePublic(x509KeySpec);
         
         
        //verify
        Signature signature = Signature.getInstance("SHA256withECDSA");
        signature.initVerify(publicKey);
        signature.update(testData.getBytes("UTF-8"));
        if(signature.verify(new BigInteger(eformsign_signature,16).toByteArray())){
            //verify success
            System.out.println("verify success");
        }else{
            //verify fail
            System.out.println("verify fail");
        }



    .. code-tab:: python
        :title: Python

        import hashlib
        import binascii
         
        from ecdsa import SigningKey, VerifyingKey, BadSignatureError
        from ecdsa.util import sigencode_der, sigdecode_der
         
        privateKeyHex = "3041020100301306072a8648ce3d020106082a8648ce3d0301070427302502010104207eae51d5e4272ebb3fe2701d25026a8c2850965981fb2efa68c8db48b32ede07"
        publicKeyHex = "3059301306072a8648ce3d020106082a8648ce3d030107034200045ac8a472cee38601e99b2a2d731c958e738eee1ee6aca28f6f5637f231e9a8444f3cb80d9ce6c5bace1d0e71167673ff81743e0ea811ebd999f2f314f1d0a676"
         
        data = "{\"test\":\"signature test\"}"
         
        sk = SigningKey.from_der(binascii.unhexlify(privateKeyHex))
        vk = VerifyingKey.from_der(binascii.unhexlify(publicKeyHex))
         
        signature = sk.sign(data.encode('utf-8'), hashfunc=hashlib.sha256, sigencode=sigencode_der)
         
        print("data: " + data)
        print("eformsign_signature : " + binascii.hexlify(signature).decode('utf-8'))
         
        try:
            if vk.verify(signature, data.encode('utf-8'), hashfunc=hashlib.sha256, sigdecode=sigdecode_der):
                print("verify success")
        except BadSignatureError:
            print("verify fail")


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
         
        function Sign($message, $privateKey)
        {
            openssl_sign($message, $signature, $privateKey->openSslPrivateKey, OPENSSL_ALGO_SHA256);
            return $signature;
        }
         
        function Verify($message, $signature, $publicKey)
        {
            return openssl_verify($message, $signature, $publicKey->openSslPublicKey, OPENSSL_ALGO_SHA256);
        }
        ?>


    .. code-tab:: php
        :title: PHP - test.php

        <?php
        require_once __DIR__ . '/keycheck.inc.php';
         
        define('PRIVATE_KEY', '3041020100301306072a8648ce3d020106082a8648ce3d0301070427302502010104207eae51d5e4272ebb3fe2701d25026a8c2850965981fb2efa68c8db48b32ede07');
        define('PUBLIC_KEY', '3059301306072a8648ce3d020106082a8648ce3d030107034200045ac8a472cee38601e99b2a2d731c958e738eee1ee6aca28f6f5637f231e9a8444f3cb80d9ce6c5bace1d0e71167673ff81743e0ea811ebd999f2f314f1d0a676');
        define('MESSAGE', '{"test":"signature test"}');
         
        use eformsignECDSA\PrivateKey;
        use eformsignECDSA\PublicKey;
         
        $sk = new PrivateKey(PRIVATE_KEY);
        $vk = new PublicKey(PUBLIC_KEY);
         
        $signature = eformsignECDSA\Sign(MESSAGE, $sk);
         
        print 'data: ' . MESSAGE . PHP_EOL;
        print 'eformsign_signature : ' . bin2hex($signature) . PHP_EOL;
         
        $ret = - 1;
        $ret = eformsignECDSA\Verify(MESSAGE, $signature, $vk);
         
        if ($ret == 1) {
            print 'verify success' . PHP_EOL;
        } else {
            print 'verify fail' . PHP_EOL;
        }
         
        ?>



Webhook 提供リスト
====================

次の Webhook を設定すると、そのイベントが発生するとき、設定した Webhook endpoint に変更情報を受信することができます。 

現在提供している `Webhook <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/Webhook>`_\は次のとおりです。


``POST``: `/webhook document event <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/Webhook#/default/post-webhook-document-event>`_\  文書イベント送信

``POST``: `/webhook pdf <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/Webhook#/default/post-webhook-pdf>`_\  PDF 生成イベント送信


各 eformsign Webhook についての詳しい説明は 
`次 <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/Webhook>`__\ で確認することができます。




Webhook についての情報
=========================

eformsign は Webhook イベントとして **文書** イベントと **PDF 生成** イベントを提供しています。


文書イベント
-------------

eformsign で文書の作成または状態の変更があるときに発生するイベントです。


.. table:: 

   ================ ====== ================
   Name             Type   説明
   ================ ====== ================
   id               String 文書のID
   template_id      String テンプレートのID
   template_name    String テンプレートの名前
   template_version String テンプレートのバージョン
   workflow_seq     int    ワークフローの順序
   workflow_name    String ワークフローの名前
   history_id       String 文書履歴のID
   status           String 文書の状態
   editor_id        String 作成者のID
   updated_date     long   文書の変更時間
   ================ ====== ================


イベントデータのうち、文書の状態を表す status の意味については、次をご覧ください。

.. _status: 

.. table:: 

   ========================== ===========================
   Name                       説明
   ========================== ===========================
   doc_create                 作成
   doc_tempsave               下書き保存
   doc_request_approval       決裁を依頼
   doc_accept_approval        決裁を承認
   doc_reject_approval        決裁を返戻
   doc_request_external       外部受信者に依頼
   doc_remind_external        外部受信者に再依頼
   doc_open_external          外部受信者が閲覧
   doc_accept_external        外部受信者が承認
   doc_reject_external        外部受信者が返戻
   doc_request_internal       内部受信者に依頼
   doc_accept_internal        内部受信者が承認
   doc_reject_internal        内部受信者が返戻
   doc_tempsave_internal      内部受信者が下書き保存
   doc_cancel_request         依頼を無効化
   doc_reject_request         返戻を依頼
   doc_decline_cancel_request 返戻依頼を拒否
   doc_delete_request         削除を依頼
   doc_decline_delete_request 削除依頼を拒否
   doc_deleted                削除
   doc_complete               完了
   ========================== ===========================


PDF 生成イベント
----------------

eformsign で文書の PDF ファイルを生成するときに発生するイベントです。

.. table:: 

   ===================== ====== ===================
   Name                  Type   説明
   ===================== ====== ===================
   document_id           String 文書のID
   template_id           String テンプレートのID
   template_name         String テンプレートの名前
   template_version      String テンプレートのバージョン
   workflow_seq          int    ワークフローの順序
   workflow_name         String ワークフローの名前
   document_history_id   String 文書履歴のID
   document_status       String 文書の状態
   ===================== ====== ===================


イベントデータのうち、文書の状態を表す status の意味については、`次 <#status>`__\をご覧ください。

