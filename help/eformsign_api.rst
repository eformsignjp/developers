
--------------------------
eformsign APIの使い方
--------------------------

eformsignが提供するAPIを利用し、eformsignの機能をお客様のシステム/サービス内で呼び出し、ご利用いただけます。


始める前に  
================

eformsign APIを利用するためには、以下のような事前準備が必要です。

- 会社IDと文書IDの確認
- APIキーの作成および秘密鍵の確認
- 署名の作成

.. caution:: 
   
   署名の作成には30秒の時間制限があります。30秒以内に署名を作成し、トークンを取得してください。 



会社IDと文書IDの確認
---------------------------

eformsign APIを使用するためには、会社IDと照会対象の文書IDが必要です。 

eformsignサービスにログインし、会社IDと文書IDを確認してください。

.. note:: 

   会社IDは、左側のメニューの **会社管理 > 会社情報** メニューの **基本情報** タブから確認することができます。

   |image1|

   文書IDは、各文書トレイの右上にある文書アイコン(|image2|)をクリックし、文書IDコラムを追加すると、照会したい文書のID を確認することができます。

   |image3|



.. _apikey:

APIキーの作成および秘密鍵の確認
----------------------------------------

.. note:: 

    APIキーは代表管理者のみ発行することができます。代表管理者でない場合は、代表管理者にご確認ください。


1. eformsignに代表管理者としてログインし、メニューツリーから **[コネクト] >[ API/Webhook]** に移動します。 

.. image:: resources/apikey1.png
    :width: 700
    :alt: コネクト > API/Webhook メニューの位置


2. **[APIキー]** タブを選択し、**APIキーの作成** ボタンをクリックします。 

.. image:: resources/apikey2.png
    :width: 700
    :alt: 「APIキー登録」ボタン


3. **APIキーの作成** ポップアップ画面にエイリアスとアプリケーション名を入力し、**保存** ボタンをクリックします。

.. image:: resources/apikey3.PNG
    :width: 700
    :alt: 「APIキーの作成」ポップアップ画面

4. **検証タイプ**\ を選択し、保存します。

.. note:: 
    
   検証タイプは **Bearer token, Basic authentication, eformsign signature** の中から選択することができます。

   - **Bearer Token**\ : 事前に設定した値をもとに検証を行う方式です。

    .. image:: resources/apikeyauth1.PNG
        :width: 300
        :alt: 「APIキーの作成」ポップアップ1


    検証タイプを **Bearer token**\ に設定し、トークンの値を **値**\ 欄に入力し、 **保存**\ します。 `Access Tokenの発行 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_\ の際に要請ヘッダeformsign_signatureにトークンの値を **Bearer トークンの値** の形式で入力します。以下の例をご参照ください。

    .. code:: Javascript

        curl --location --request POST 'https://service.eformsign.com/v2.0/api_auth/access_token' \
        --header 'eformsign_signature: Bearer {トークンの値}' \
        --header 'Content-Type: application/json' \
        --header 'Authorization: Bearer {base64 encoded api key }' \
        --data-raw '{
         "execution_time":{現在時刻ms},
         "member_id": {eformsignアカウント}
        }'


   - **Basic Authentication**\ : IDとパスワードをもとに検証を行う方式です。

    .. image:: resources/apikeyauth2.PNG
        :width: 300
        :alt: 「APIキーの作成」ポップアップ2



    検証タイプを **Basic authentication**\ に設定し、使用するIDとパスワードを入力後 **保存**\ します。 `Access Tokenの発行 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`__\ の際は要請ヘッダeformsign_signatureに **ID:パスワード** の形式でBase64インコードした値を入力します。以下の例をご参照ください。

    .. code:: Javascript

        curl --location --request POST 'https://service.eformsign.com/v2.0/api_auth/access_token' \
        --header 'eformsign_signature: Basic {base64 encoded "id:password"}' \
        --header 'Content-Type: application/json' \
        --header 'Authorization: Bearer {base64 encoded api key }' \
        --data-raw '{
         "execution_time":{現在時刻ms},
         "member_id": {eformsignアカウント}
        }'


   - **eformsign Signature**\ : eformsign Signatureの署名の値を利用して検証を行う方式です。検証タイプを **eformsign signature**\ に設定し、保存します。eformsign Signatureで署名を生成する方法は `署名の生成方法 <#eformsign-signature>`__\をご参照ください。




4. 作成されたキーリストから **キーを表示** ボタンをクリックすると、APIキーと秘密鍵を確認することができます。

.. image:: resources/apikey4.png 
    :width: 700 
    :alt: 「APIキーを表示」ボタンの位置 

.. image:: resources/apikey5.png 
    :width: 700 
    :alt: APIキーと秘密鍵の確認 



.. note:: **APIキーの修正方法**

    作成されたキーリストから **修正** ボタンをクリックし、エイリアスとアプリケーション名を修正することができます。
  	また、ステータス領域をクリックし、活性/非活性状態に変更することもできます。

.. note:: **APIキーの削除**

    作成されたキーリストから **削除** ボタンをクリックして、APIキーを削除することができます。


.. _eformsign_signature:

署名の作成
==============

APIを使用するためのトークンを発行するためには、まずeformsign_signatureという署名を生成する必要があります。
eformsign_signatureは非対称キー方式と楕円曲線暗号化(Elliptic curve cryptography)を使用しています。
サーバーの現在時刻をString(UTF-8)に変換し、`API Keyの発行 <#apikey>`__\ で発行された秘密鍵(Private key)で署名後、署名したデータをhex stringに変換することで署名を生成することができます。

.. tip:: 
   
   楕円曲線暗号は、公開キー暗号化方式の一つで、データ暗号化デジタル認証など現在もっとも広く使われている暗号化方式です。 

.. note:: 

   署名アルゴリズムには、SHA256withECDSAを使用しています。


署名の生成方法については、Java、JavaScript (Node.js)、Python、PHPと言語ごとにご説明します。

署名の作成方法については、Java、Python、PHPの各言語に分けて説明します。

Java
-------

Javaの例では、署名生成のためにjava.securityパッケージを使用します。



Javascript(Node.JS)
------------------------

Jsrsasign(https://kjur.github.io/jsrsasign/) npmをインストールしてください。

.. code:: Javascript

   npm install jsrsasign



Python
-------

キーフォーマット処理用のライブラリーを使用する必要があります。作業の開始前に、以下のコマンドを実行してライブラリーをインストールしてください。

.. code:: python

   pip install https://github.com/warner/python-ecdsa/archive/master.zip


PHP
-------

PHPの例を使用するには、PHP OpenSSLライブラリがインストールされている必要があります。以下のようにkeycheck.inc.php、test.phpファイルが同じパス上に位置するように設定し、作業を進めてください。


C# (.NET)
-----------------------

暗号化に関する処理のため、BouncyCastleライブラリをインストールする必要があります。(NuGetパッケージでもインストール可、MIT License)



各言語のコード作成例
-----------------------

各言語のコード作成例をご紹介します。


.. note:: 

   execution_timeはlong型です。そのため、execution_timeを入力する際はAccess Token 発行時に確認した時間の後ろに「L」を追記してください。


.. note:: 

   署名を生成するには秘密鍵が必要です。
   秘密鍵を確認する方法は `API Keyの発行と確認 <#apikey>`__\ をご参照ください。



.. tabs::

    .. code-tab:: java
        :title: Java

        import java.security.KeyFactory;
        import java.security.spec.PKCS8EncodedKeySpec;
        import java.security.PrivateKey;
        import java.security.Signature;
         
        //private key
        String privateKeyHexStr = "発行したprivate keyを入力(String型)";
        KeyFactory keyFact = KeyFactory.getInstance("EC");
        PKCS8EncodedKeySpec psks8KeySpec = new PKCS8EncodedKeySpec(new BigInteger(privateKeyHexStr,16).toByteArray());
        PrivateKey privateKey = keyFact.generatePrivate(psks8KeySpec);
         
        //execution_time サーバー時刻
        long execution_time = new Date().getTime();
        //long execution_time = 1611537340731L;     //long型のため、作成された時間の後ろにLを追記     
        String execution_time_str = String.valueOf(execution_time);
         
        //eformsign_signatureの生成
        Signature ecdsa = Signature.getInstance("SHA256withECDSA");
        ecdsa.initSign(privateKey);
        ecdsa.update(execution_time_str.getBytes("UTF-8"));
        String eformsign_signature = new BigInteger(ecdsa.sign()).toString(16);
         
         
        //現在時刻と現在時刻の署名値
        System.out.print("execution_time : "+execution_time);
        System.out.print("eformsign_signature : "+eformsign_signature);

    .. code-tab:: javascript
        :title: Javascript(Node.JS)

        const rs = require('jsrsasign');


        // User-Data-Here
        const execution_time  = Date.now()+"";
        const privateKeyHex = "発行したprivate key(String型)";

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
        privateKeyHex = "発行したprivate key(String型)"
        privateKey = SigningKey.from_der(binascii.unhexlify(privateKeyHex))
         
        # execution_time - サーバー時刻
        execution_time_int = int(time() * 1000)
        execution_time = str(execution_time_int)
          
        # eformsign_signature作成
        eformsign_signature = privateKey.sign(execution_time.encode('utf-8'), hashfunc=hashlib.sha256, sigencode=sigencode_der)
          
        # 現在時刻と現在時刻の署名値
        print("execution_time : " + execution_time)
        print("eformsign_signature : " + binascii.hexlify(eformsign_signature).decode('utf-8'))


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
         
         
        define('PRIVATE_KEY', '発行したprivate key(String型)');
         
         
        //private key設定
        $privateKey = new PrivateKey(PRIVATE_KEY);
         
         
        //execution_time - サーバー時刻
        $execution_time = eformsignECDSA\getNowMillisecond();
         
         
        //eformsign_signatureの生成
        $signature = eformsignECDSA\Sign($execution_time, $privateKey);
         
         
        //現在時刻および現在時刻の署名値
        print 'execution_time : ' . $execution_time . PHP_EOL;
        print 'eformsign_signature : ' . bin2hex($signature) . PHP_EOL;
        ?>

    .. code-tab:: C#
        :title: C#(.NET) - Program.cs

        using System;
        using System.Text;
        using Org.BouncyCastle.Crypto;
        using Org.BouncyCastle.Security;
        using Org.BouncyCastle.Crypto.Parameters;
         
         
        namespace eformsign_signature
        {
            class Program
            {
                private static readonly string HASH_ENCRYPTION_ALGORITHM = "SHA256withECDSA";
         
         
                static void Main(string[] args)
                {
                    byte[] privateKeyBytes = HexStringToByteArray("ここに発行した秘密鍵を入力(String型)");
                     
                    DateTime unixEpoch = new DateTime(1970, 1, 1);
                    DateTime currentTime = DateTime.UtcNow;
                    TimeSpan timeDiff = currentTime.Subtract(unixEpoch);
                    long unixCurrentTime = (long)timeDiff.TotalMilliseconds;
                    string execution_time = unixCurrentTime.ToString();
         
         
                    ISigner signer = SignerUtilities.GetSigner("SHA256withECDSA");
                    signer.Init(true, (ECPrivateKeyParameters)PrivateKeyFactory.CreateKey(privateKeyBytes));
         
         
                    byte[] dataBytes = Encoding.UTF8.GetBytes(execution_time);
         
         
                    signer.BlockUpdate(dataBytes, 0, dataBytes.Length);
                    byte[] signatureBytes = signer.GenerateSignature();
         
         
                    Console.WriteLine("execution_time : {0}", execution_time);
                    Console.WriteLine("eformsign_signature : {0}", ByteArrayToHexString(signatureBytes));
                }
         
         
                public static string ByteArrayToHexString(byte[] data)
                {
                    StringBuilder builder = new StringBuilder();
                    for (int i = 0; i < data.Length; i++)
                    {
                        builder.Append(data[i].ToString("x2"));
                    }
                    return builder.ToString();
                }
         
         
                public static byte[] HexStringToByteArray(string hexString)
                {
                    if (hexString.Length % 2 != 0)
                    {
                        throw new ArgumentException("{0} must have an even length", nameof(hexString));
                    }
                    byte[] bytes = new byte[hexString.Length / 2];
                    for (int i = 0; i < bytes.Length; i++)
                    {
                        string currentHex = hexString.Substring(i * 2, 2);
                        bytes[i] = Convert.ToByte(currentHex, 16);
                    }
                    return bytes;
                }
            }
        }





API提供リスト
======================

eformsign APIは、署名を作成するためのAPIと文書の作成・処理のためのAPI、メンバーおよびグループ管理のためのAPIの3つのAPIで構成されています。


.. _access_token_api:

トークンの発行と更新のAPI
---------------------------

APIを使用するためには、まずAccess Tokenを発行する必要があります。発行したトークンには有効期限があり、有効期間経過後は更新または再発行が必要です。

**トークンの発行**
~~~~~~~~~~~~~~~~~~

API KeyによりAccess Tokenを発行するAPIです。

``POST``: `Access Tokenの発行 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_\


Access Token APIに関するより詳しい説明は
`こちら <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/token>`__\ からご確認いただけます。
 
.. Note:: 


   トークンを発行するには、以下の各内容を入力してください。

   - Authorize: eformsignから発行されたAPIキーをBase64でエンコードした値を入力します。
   - Header: 生成した署名値(eformsign_signature) (* `参考 <#apikey>`__\: APIキー発行の際に設定した検証タイプによって異なります)
   - Request body: 署名生成時間(execution_time)とアカウントID(member_id)を入力します。

   |image5| 


   メンバーの情報なしでトークンを発行することもできます。この場合、Request bodyに署名生成時間(execution_time)のみを入力してください。
   ただし、このトークンでは、新規文書作成などのメンバー情報が必要なAPIを利用することはできません。

   |image6| 



   必要な値を入力してAPIを呼び出すとAccess Tokenが発行され、以下のような形式の応答を受け取ることができます。APIの利用に必要なoauth_token.access_token、トークンの更新に必要なoauth_token.refresh_token、トークンの有効時間を表すoauth_token.expires_in、API呼び出しサーバーを表すapi_key.company.api_urlをご確認ください。

   .. code:: JSON

      { "api_key": { "name": "アプリケーション_", "alias": "テスト用", "company": { "company_id": "dec5418e58694d90a65d6c38e3d226db", "name": "サンプル デモ", "api_url": "https://jp-api.eformsign.com" } }, "oauth_token": { "expires_in": 3600, "token_type": "JWT", "refresh_token": "8fd0a3c1-44dc-4a03-96ad-01fa34cd159c", "access_token": "eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJlZm9ybXNpZ24uaWFtIiwiY29udGV4dCI6eyJjbGllbnRJZCI6IjY4MDk0ZWVhMjVhZjRhNjI5ZTI4ZGU5Y2ZlYzRlYmZjIiwiY2xpZW50S2V5IjoiZTNiM2IzZTUtMGEzMS00NTE1LWE5NzEtN2M4Y2FlNDI4NzZmIiwibWFuYWdlbWVudElkIjoiMzRhYWI4MDBjMmEwNDQwNThmZDRlZjc5OGFlY2RlY2EiLCJzY29wZXMiOiJzbWFydF9lZm9ybV9zY29wZSIsInR5cGUiOiJ1c2VyIiwidXNlck5hbWUiOiIzMmIzZDRmOC00MjdkLTRjZjQtOTZiYS1mYzAxNjIxNWRkNDciLCJ1c2VySWQiOiJhNTEyNGVkNmU2M2Y0OTMzOGJlOTA0MjVhNjFkYjlmNSIsInJlZnJlc2hUb2tlbiI6IjhmZDBhM2MxLTQ0ZGMtNGEwMy05NmFkLTAxZmEzNGNkMTU5YyJ9LCJjbGFpbSI6eyJjb21wYW55X2lkIjoiZGVjNTQxOGU1ODY5NGQ5MGE2NWQ2YzM4ZTNkMjI2ZGIiLCJhY2Nlc3Nfa2V5IjoiMzJiM2Q0ZjgtNDI3ZC00Y2Y0LTk2YmEtZmMwMTYyMTVkZDQ3In0sImV4cCI6MTYxMTU0MjIzNiwiaWF0IjoxNjExNTM4NjM2fQ.BltoXXBSabjXfpyLsZik9OZTE5XtLqe9lguMmJ_qfwZN1NyoVoxDqA5y1-_TLis7FvvNjfI1eegOroCZDZPFyXRaBxAj0CW8TijVjbhliJBuccHFyKXaJxmo_GMmTHYtxNNB1SUgLeFIrYROnpQndU8J7ZkfPDgYGwh1YSx-5s4" } }



.. caution:: 
   
   署名の作成には30秒の時間制限があります。30秒以内に署名を作成し、トークンを取得する必要があります。 
   また、サーバー時刻と現在時刻が一致しない場合があります。Access Token APIを呼び出し、受信した応答メッセージの "execution_time" を確認してください。

   .. code:: JSON

      { "code": "4000002", "ErrorMessage": "The validation time has expired.",     "execution_time": 1611538409405 }


**トークンの更新**
~~~~~~~~~~~~~~~~~~~~

トークンの有効期限は3600秒（1時間）となっています。
トークンの有効期限が切れると、トークンによりAPIを利用できなくなります。以下のAPIを使用してトークンを更新してください。

``POST``: `Access Tokenの更新 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/token/post-api_auth-refresh_token>`_\



.. note:: 

   トークン更新のためにはAccess TokenをAuthorizeに、Refresh TokenをQueryに入力する必要があります。

.. caution:: 

   トークンの有効期限が切れてから一定時間が経過すると、refresh_tokenが無効であるという応答を受け取ることがあります。この場合、`Access Tokenの発行API <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_\ によりトークンを再発行してください。


------------


文書の作成と処理のAPI
----------------------------------

署名の生成後は、以下の文書APIを使用して新規文書の作成、文書情報の照会、完了文書ファイル(文書PDF、監査証跡証明書)のダウンロード、文書の添付ファイルのダウンロードを行うことができます。


.. caution:: 

 本文書のAPIを使用するためには、Access Tokenを先に発行する必要があります。`Access Token API <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/token/post-api_auth-access_token>`_\ で発行したAccess Tokenを `こちら <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/>`__\ にある **Authorize** ボタン(|image4|)を押して登録してください。


.. note:: 
  
   Access Token APIの **Authorize** ボタンにはAPIキー値の入力が必要です。


現在提供中の `文書API <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document>`_\ は以下のとおりです。



``POST``: `文書の新規作成_内部受信者 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/post-api-documents>`_\ 

``POST``: `文書の新規作成_外部受信者 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/post-api-documents-external>`_\ 



``GET``: `文書情報の照会 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/get-api-documents-DOCUMENT_ID>`_\

``GET``: `文書ファイルのダウンロード_文書PDFおよび監査証跡証明書 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/get-api-documents-DOCUMENT_ID-download_files>`_\

``GET``: `添付ファイルのダウンロード <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/get-api-documents-DOCUMENT_ID-download_attach_files>`_\ 

``GET``: `文書リストの照会 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/get-api-documents>`_\ 

``POST``: `文書リストの照会(POST) <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/post-api-list-documents>`_\ 

``DELETE``: `文書の削除 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/delete-api-documents>`_\ 

``DELETE``: `文書の無効化 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/post-api-documents-cancel>`_\ 

``POST``: `外部受信者に対する再依頼 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/post-api-documents-document_id-re_request_outsider>`_\ 


``POST``: `文書の一括生成 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/post-api-forms-mass_documents%3Ftemplate_id%3D-form_id>`_\

``POST``: `文書の一括作成_複数テンプレート <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/post-api-forms%2Fmass_multi_documents>`_\  

``POST``: `利用状況の照会 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/get-api-companies-COMPANY_ID-use_status>`_\  

``POST``: `内部受信者の差戻し <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/post-api-document-DOCUMENT_ID-decline>`_\  

``POST``: `外部受信者の差戻し <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/post-api-document-DOCUMENT_ID-external_decline>`_\  

``GET``: `作成可能なテンプレートリストの照会 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/get-api-forms>`_\  

``DELETE``: `テンプレートの削除 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/document/delete-forms>`_\  


------------------------


メンバーとグループ管理のAPI
---------------------------------------------

APIを使用してメンバーとグループを管理することができます。メンバーとグループのリストの照会、メンバーとグループの修正、削除を行うことができます。


.. caution:: 

   本文書のAPIを使用するには、Access Tokenの取得が必要です。`Access Token API <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0_auth>`_\ で取得した Access Tokenを `こちら <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/>`__\ にある **Authorize** ボタン (|image4|) を押して登録してください。 


.. note:: 
  
   APIの **Authorize** ボタンにはAPIキー値の入力が必要です。 


現在提供している `メンバーとグループ管理のAPI <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/members>`_\は以下のとおりです。


メンバー管理 API
~~~~~~~~~~~~~~~~~~~


``GET``: `メンバーリストの照会 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/members/get-api-members>`_\   

``PATCH``: `メンバー修正 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/members/patch-api-members-member_id>`_\  

``DELETE``: `メンバー削除 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/members/delete-api-members-member_id>`_\  


グループ管理 API
~~~~~~~~~~~~~~~~~~~


``GET``: `グループリストの照会 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/groups/get-api-groups>`_\  

``POST``: `グループ追加 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/groups/post-api-groups>`_\  

``PATCH``: `グループ修正 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/groups/patch-api-groups>`_\  

``DELETE``: `グループ削除 <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/groups/delete-api-groups>`_\  



.. note:: 

    各eformsign APIについての詳しい説明は `こちら <https://app.swaggerhub.com/apis-docs/eformsign_api.jp/eformsign_API_2.0/2.0#/>`__\ をご覧ください。






APIコード
===================

Open APIを使用する際は、以下のコードをご参照ください。


APIステータスコード
---------------------

APIステータスコードは、以下の通りです。

200
~~~~~~~~~~~~~~~~~~

===========  ===============  ===================================
Code         説明              備考
===========  ===============  ===================================
200          成功              成功
===========  ===============  ===================================


202
~~~~~~~~~~~~~~~~~~

===========  ===============  ===================================
Code         説明              備考
===========  ===============  ===================================
2020001      PDF生成中          - PDFファイルとしてダウンロードする際、非同期で作成されるため、文書保存後にPDFファイルの生成まで時間が必要 
                                - 数秒から数分後に再度リクエストすることでダウンロード可能
===========  ===============  ===================================


400
~~~~~~~~~~~~~~~~~~

===========  =====================  ======================================================================
Code         説明                    備考
===========  =====================  ======================================================================
4000001      必須入力値漏れ           APIの必須入力値（ヘッダー値またはパラメーター）が入力されていない場合                        
4000002      時間切れ                 API認証のリクエストが時間切れとなった場合
4000003      APIキーが存在しない      削除されたAPIキーまたは入力ミスの場合
4000004      文書が存在しない         間違った文書IDを入力した場合
4000005      会社が存在しない         会社が削除された場合
===========  =====================  ======================================================================


403
~~~~~~~~~~~~~~~~~~

===========  =========================  ==========================================
Code         説明                        備考
===========  =========================  ==========================================
4030001      アクセス権限なし             APIが非活性状態の場合
4030002      Access Token認証エラー       Access Tokenが正しくない場合
4030003      Refresh Token認証エラー      Refresh Tokenが正しくない場合
4030004      署名値検証エラー             署名値が正しくない場合
4030005      サポートされていないAPI      サポートされていないAPIを呼び出した場合
===========  =========================  ==========================================

405
~~~~~~~~~~~~~~~~~~

===========  =============================  ===================================
Code         説明                            備考
===========  =============================  ===================================
4050001      サポートされていないmethod        サポートされていないmethodを呼び出した場合
===========  =============================  ===================================


500
~~~~~~~~~~~~~~~~~~

===================  ===============  ===================================
Code                 説明              備考
===================  ===============  ===================================
5000001~5000003      サーバーエラー       サーバーエラーが発生した場合
===================  ===============  ===================================


---------


ユーザータイプ(user_type)
-------------------------

==============  ===============  ===================================
Type             Code             説明
==============  ===============  ===================================
内部受信者       01                会社に所属する内部メンバーの場合
外部受信者       02                会社に所属しない外部メンバーの場合
==============  ===============  ===================================


ステップタイプ
--------------

===========  ===============  ===================================
Type         Code             説明
===========  ===============  ===================================
Start         00               開始ステップ
Complete      01               完了ステップ
Approval      02               決裁ステップ(旧)
External      03               外部受信者ステップ(旧)
Accept        04               内部受信者ステップ(旧)
Participant   05               参加者
Reviewer      06               検討者
Viewer        07               閲覧者
===========  ===============  ===================================


文書の現在のステータスタイプ (current_status.status_type)
-----------------------------------------------------------------

==========================  ===============  ================================================
Type                         Code             説明
==========================  ===============  ================================================
doc_tempsave                 001              下書き保存 (作成者が下書きとして保存した状態)
doc_create                   002              文書の作成
doc_complete                 003              文書の完了
doc_update                   043              文書の修正
doc_request_delete           047              文書の削除依頼
doc_delete                   049              文書の削除
doc_request_revoke           040              文書の無効化依頼
doc_revoke                   042              文書の無効化
doc_request_reject           045              文書の差戻し依頼
doc_request_participant      060              参加者に依頼
doc_accept_participant       062              参加者の承認
doc_reject_participant       061              参加者の差戻し
doc_request_reviewer         070              検討者に依頼
doc_accept_reviewer          072              検討者の承認 
doc_reject_reviewer          071              検討者の差戻し
==========================  ===============  ================================================


文書の次のステータスタイプ(next_status.status_type)
------------------------------------------------------

===========  ===============  ==========================================================
Type          Code             説明
===========  ===============  ==========================================================
下書き保存     00               開始ステップで下書きとして保存した状態
進行中　       01               決裁依頼、外部受信者に依頼、閲覧、内部受信者に依頼
修正中　       02               文書の修正中 (内部受信者、作成者)
完了           03               文書が完了した状態
差戻し         04               決済の差戻し、内部受信者の差戻し、外部受信者の差戻し
無効化         05               文書が無効化された状態
無効化依頼     06               内部受信者のステップ
===========  ===============  ==========================================================



アクションタイプ(action_type)
------------------------------

==========================  ===============  ===================================
Type                         Code             説明
==========================  ===============  ===================================
doc_tempsave                 001              文書の下書き保存
doc_create                   002              文書の生成
doc_complete                 003              文書の完了
doc_request_approval         010              決裁の依頼
doc_reject_approval          011              決裁の差戻し
doc_accept_approval          012              決裁の承認
doc_cancel                   013              決裁依頼のキャンセル
doc_request_reception        020              内部受信者に依頼
doc_reject_reception         021              内部受信者の差戻し
doc_accept_reception         022              内部受信者の承認
doc_accept_tempsave          023              内部受信者の下書き保存
doc_request_outsider         030              外部受信者に依頼
doc_reject_outsider          031              外部受信者の差戻し
doc_accept_outsider          032              外部受信者の承認
doc_rerequest_outsider       033              外部受信者に再依頼
doc_open_outsider            034              外部受信者の閲覧
doc_outsider_tempsave        035              外部受信者の下書き保存
doc_request_revoke           040              無効化の依頼
doc_refuse_revoke            041              無効化依頼の依頼拒否
doc_revoke                   042              無効化
doc_update                   043              文書の修正
doc_cancel_update            044              文書修正のキャンセル
doc_request_reject           045              文書差戻しの依頼
doc_refuse_reject            046              文書差戻し依頼の拒否
doc_request_delete           047              文書削除の依頼
doc_refuse_delete            048              文書削除依頼の拒否
doc_delete                   049              文書の削除
doc_complete_send_pdf        050              完了文書をPDFファイルで送信
doc_transfer                 051              文書の移管
doc_request_participant      060              参加者に依頼
doc_reject_participant       061              参加者の差戻し
doc_accept_participant       062              参加者の承認
doc_rerequest_participant    063              参加者に再依頼(外部受信者)
doc_open_participant         064              参加者の閲覧(外部受信者)
doc_request_reviewer         070              検討者に依頼
doc_reject_reviewer          071              検討者の差戻し
doc_request_reviewer         072              検討者の承認
doc_rerequest_reviewer       073              検討者に再依頼(外部受信者)
doc_open_review              074              検討者の閲覧(外部受信者)
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