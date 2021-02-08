----------------------------
eformsign Webhook 사용하기
----------------------------

eformsign でイベントが発生した時、そのイベント情報を 고객 システム/サービス로 に通知する機能です。Webhook を設定すると、고객の Webhook endpoint にそのイベント情報を HTTP POST 形式で通知します。

.. tip:: 

   Webhook の endpoint とは、고객の client callback URL を意味します。Open API を持続的に呼び出して변경사항をチェックする方法（polling）に対し、不要な呼び出しを 없이 eformsign 上のイベントについての情報を取得できます。


시작하기 
=========


.. _webhook:

Webhook 키 발급하기
--------------------

1. eformsign에 대표 관리자로 로그인 후, 메뉴 트리에서 **[커넥트] > [API / Webhook]** ページに移動します。 

.. image:: resources/apikey1.PNG
    :width: 700
    :alt: 커넥트 > API/Webhook 메뉴 위치


2. **[Webhook 관리]** 탭을 선택하고 **Webhook 추가** ボタンをクリックします。

.. image:: resources/webhook2.PNG
    :width: 700
    :alt: Webhook 추가 ボタン


3. Webhook 키 생성 팝업창에 이름, Webhook URL, 활성 상태, 적용 テンプレートを入力して「登録」ボタンをクリックします。

.. image:: resources/webhook3.PNG
    :width: 700
    :alt: Webhook 키 생성 팝업창


4. 생성된 Webhook 목록から **키보기** ボタンをクリックして Webhook 공개키を確認します。

.. image:: resources/webhook4.PNG
    :width: 700
    :alt: Webhook 키보기 ボタン 위치

.. image:: resources/webhook5.PNG
    :width: 700
    :alt: Webhook 키 確認 



.. note:: 

    **키 재발행** ボタンをクリックすると해당 Webhook の공개 키が재발행され、이전의 키는 사용할 수 없게 됩니다.

.. note:: **Webhook 情報 수정 방법**

    생성된 Webhook 목록에서 **수정** ボタンをクリックしてWebhook 情報를 변경할 수 있습니다.


.. note:: **Webhook 삭제 방법**

    생성된 Webhook 목록에서 **삭제** ボタンをクリックしてWebhook을 삭제할 수 있습니다.    



5. 생성된 Webhook リスト에서 テスト ボタンをクリックするとテスト Webhook을 전송하고 결과를 반환합니다.

.. image:: resources/webhook6.PNG
    :width: 700
    :alt: Webhook テスト 確認 

次はテストのための json ファイルです。

.. code:: json

	{
	"webhook_id" : "해당 Webhook ID",
	"webhook_name" : "해당 Webhook 이름",
	"company_id" : "회사의 ID",
	"event_type" : “document”,
	"document" : {
	  "id" : “test_doc_id”,
	   "template_id" : “test_template_id”,
	   "template_version" : “1”,
	   "document_history_id" : “test_document_history_id”,
	   "doc_status" : “doc_create”,
	   "editor_id" : "사용자 ID",
	   "updated_date" : "현재 시간(UTC Long)"
	}
	}
	Test URL : 해당 Webhook の URL




署名の登録 
==============


署名の登録方法については、Java、Python、PHPの各言語に分けて説明します。

Java
-------

eformsign サーバー로 부터 전달 받은 イベント情報を `Webhook Key 발급하기 <#webhook>`__\에서 발급받은 public key で検証し、 eformsign で正常に呼び出したイベントであるかについての検証を行います。 

.. note:: 
  署名のアルゴリズムは SHA256withECDSA を使用します。


Python
-------

キーフォーマット処理用のライブラリーを使用する必要があります。作業を開始する前に、次のコマンドを実行してライブラリーをインストールしてください。

.. code:: python

   pip install https://github.com/warner/python-ecdsa/archive/master.zip


PHP
-------

次の例題の keycheck.inc.php、test.php ファイルを同じパスに保存してから例題を実行してください。


各言語の例題
---------------------

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
         *  request에서 header와 body를 읽습니다.
         *
         */
         
         
        //1. get eformsign signature
        //eformsignSignature는 request header에 담겨 있습니다.
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
         
         
         
         
        //3. publicKey 세팅
        String publicKeyHex = "발급 받은 Public Key(String)";
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
             * 이곳에서 イベント에 맞는 처리를 진행합니다.
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
    		# request에서 header와 body를 읽습니다.
    		# 1. get eformsign signature
    		# eformsignSignature는 request header에 담겨 있습니다.
    		eformsignSignature = request.headers['eformsign_signature']
    		 
    		 
    		# 2. get request body data
    		# eformsign signature 検証のため body のデータを String に変換します。
    		data = request.json
    		 
    		 
    		# 3. publicKey 세팅
    		publicKeyHex = "발급받은 public key"
    		publickey = VerifyingKey.from_der(binascii.unhexlify(publicKeyHex))
    		 
    		 
    		# 4. verify
    		try:
    		    if publickey.verify(eformsignSignature, data.encode('utf-8'), hashfunc=hashlib.sha256, sigdecode=sigdecode_der):
    		        print("verify success")
    		        # 이곳에 イベント에 맞는 처리를 진행 합니다.
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
         
        define('PUBLIC_KEY', '발급 받은 public key を入力してください。');
        ...
        /*
         *  request で header と body を읽습니다.
         *
         */
         
         
        //1. get eformsign signature
        //eformsignSignature は request header に담겨 있습니다.
        $eformsignSignature = $_SERVER['HTTP_eformsign_signature'];
         
         
        //2. get request body data
        // eformsign signature 検証のため body のデータ를 읽습니다.
        $eformsignEventBody = json_decode(file_get_contents('php://input'), true);
         
         
        //3. publicKey 세팅
        $publicKey = new PublicKey(PUBLIC_KEY);
         
         
        //4. verify
        $ret = - 1;
        $ret = eformsignECDSA\Verify(MESSAGE, $eformsignSignature, $publicKey);
          
        if ($ret == 1) {
            print 'verify success' . PHP_EOL;
            /*
             * 이곳에서 イベント에 맞는 처리를 진행합니다.
             */
        } else {
            print 'verify fail' . PHP_EOL;
        }
         ...
          
        ?>



テスト
==========================

생성한 eformsign_signatureをテストしてみましょう。 

次の eformsign_signature 생성および検証用サンプルは、Open API または Webhook の署名値を생성 및 検証するためのテストサンプルのソースコードです。

.. note::

   サンプルキー를 사용하고 있어 실 사용시에는 정상 동작 하지 않습니다. 생성하신 署名値の検証용으로만 사용해 주세요.


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

次の例題の keycheck.inc.php、test.php ファイルを同じパスに保存してから例題を実行してください。



各言語の例題
---------------------

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

次の Webhook を設定すると、해당 イベントが発生するとき、設定した Webhook endpoint に変更情報を受信することができます。 

現在提供している `Webhook <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/Webhook>`_\は次のとおりです。


``POST``: `/webhook document event <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/Webhook#/default/post-webhook-document-event>`_\  文書イベント 전송

``POST``: `/webhook pdf <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/Webhook#/default/post-webhook-pdf>`_\  PDF 생성 イベント 전송


各 eformsign Webhook についての詳しい説明は 
`次 <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/Webhook>`__\ で確認することができます。




Webhook 관련 情報
===================

eformsign は Webhook イベントとして **文書** イベントと **PDF 생성** イベントを提供しています。


文書イベント
-------------

eformsign で文書の作成または상태 변경 시 発生するイベントです。


.. table:: 

   ================ ====== ================
   Name             Type   説明
   ================ ====== ================
   id               String 文書 ID
   template_id      String テンプレート ID
   template_name    String テンプレート 명
   template_version String テンプレート 제목
   workflow_seq     int    ワークフロー 순서
   workflow_name    String ワークフロー 명칭
   history_id       String 文書 히스토리 ID
   status           String 文書 상태
   editor_id        String 작성자 ID
   updated_date     long   文書 변경시간
   ================ ====== ================


イベントデータのうち、文書の상태を表す status の意味については、次をご覧ください。

.. _status: 

.. table:: 
文書イベント
   ========================== ==================
   Name                       説明
   ========================== ==================
   doc_create                 文書 생성시
   doc_tempsave               文書 임시 저장시
   doc_request_approval       결재 요청시
   doc_accept_approval        결재 승인시
   doc_reject_approval        결재 반려시
   doc_request_external       외부자 요청시
   doc_remind_external        외부자 재 요청시
   doc_open_external          외부자 열람시
   doc_accept_external        외부자 승인시
   doc_reject_external        외부자 반려시
   doc_request_internal       내부자 요청시
   doc_accept_internal        내부자 승인시
   doc_reject_internal        내부자 반려시
   doc_tempsave_internal      내부자 임시 저장시
   doc_cancel_request         요청 취소시
   doc_reject_request         반려 요청시
   doc_decline_cancel_request 반려 요청 거절시
   doc_delete_request         삭제 요청시
   doc_decline_delete_request 삭제 요청 거절시
   doc_deleted                文書 삭제시
   doc_complete               文書 완료시
   ========================== ==================


PDF 생성 イベント
----------------

eformsign で文書の PDF ファイルを生成するときに発生するイベントです。

.. table:: 

   ===================== ====== ================
   Name                  Type   説明
   ===================== ====== ================
   document_id           String 文書 ID
   template_id           String テンプレート ID
   template_name         String テンプレート 명
   template_version      String テンプレート 제목
   workflow_seq          int    ワークフロー 순서
   workflow_name         String ワークフロー 명칭
   document_history_id   String 文書 히스토리 ID
   document_status       String 文書 상태
   ===================== ====== ================


イベントデータのうち、文書の상태を表す status の意味については、`次 <#status>`__\をご覧ください。

