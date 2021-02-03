--------------------------
eformsign API 使用하기
--------------------------

eformsign が提供する API を使用し、eformsign の機能を顧客のシステム・サービスで呼び出して使用できる機能です。


시작하기 
=========

eformsign API を使用するためには、次のような準備作業が必要です。

- 会社 ID と 文書 ID の確認
- API キーの作成および暗号化キーの確認
- 署名の登録

.. caution:: 
   
   署名の登録には30秒の時間制限があります。30秒以内に署名を登録し、トークンを取得してください。 



会社 ID と 文書 ID の確認
---------------------------

eformsign APIを使用するためには、소속 会社のIDと照会したい文書のIDが必要です。 

eformsign サービスにログインし、会社 ID と文書 ID を確認してください。

.. note:: 

   会社 ID は、左のメニューツリーの会社管理 > 会社情報 メニューの"基本情報" タブで確認することができます。

   |image1|

   文書 ID は、各文書トレイの右上にある文書アイコン(|image2|)をクリックし、文書 ID コラムを追加すると、照会したい文書のIDを確認することができます。 

   |image3|



.. _apikey:

API キーの作成および暗号化キーの確認
------------------------------

1. eformsign に代表管理者としてログインし、メニューツリーで**[コネクト] > [API / Webhook]**に移動します。 

.. image:: resources/apikey1.PNG
    :width: 700
    :alt: コネクト > API/Webhook メニューの位置


2. **[API キー管理]**タブを選択し、**API キーの作成**ボタンをクリックします。

.. image:: resources/apikey2.PNG
    :width: 700
    :alt: API キー登録ボタン


3. API キー作成ポップアップにエイリアスとアプリケーション名を入力し、保存ボタンをクリックします。

.. image:: resources/apikey3.PNG
    :width: 700
    :alt: API キー登録ポップアップ


4. 生成されたキーリストから**キーを表示**ボタンをクリックし、API キーと暗号化キーを確認できます。

.. image:: resources/apikey4.PNG
    :width: 700
    :alt: API キーを表示 ボタンの位置

.. image:: resources/apikey5.PNG
    :width: 700
    :alt: API キーおよび暗号化キーの確認 



.. note:: **API キーを修正する方法**

    생성されたキーリスト에서 **修正** ボタンをクリックし、エイリアスと어플리케이션 이름を修正することができます。
  	있습니다. また、ステータス영역をクリックし、ステータスを활성/비활성に変更することもできます。

.. note:: **API キーを削除する方法**

    생성されたキーリスト에서 **削除** ボタンをクリックし、API キーを削除することができます。


署名 생성하기 
==============

eformsign_signature は、비대칭 キー방식と타원곡선 暗号化(Elliptic curve cryptography)を使用しています。

.. tip:: 
   
   타원곡선 暗号化は、공개キー暗号化 방식の一つで、デー暗号化デジタル認証など現在가장 많이 쓰이는 암호방식です。 


署名の登録方法については、Java, Python, PHP 언어별로 説明합니다.

Java
-------

サーバーの現在時刻を String(UTF-8) に変換し、`API Key 발급하기 <#apikey>`__\にて발급された private key で署名すると、署名したデータを hex string に変換します。

.. note:: 

   署名アルゴリズムは SHA256withECDSA を使用します。




Python
-------

キー포맷 처리용 ライブラリーを使用해야 합니다. 작업전 次の명령어を통해 해당 ライブラリーを설치해 주십시오.

.. code:: python

   pip install https://github.com/warner/python-ecdsa/archive/master.zip


PHP
-------

以下の예제の keycheck.inc.php, test.php ファイルを同じパスに保存してから예제を行ってください。


각 언어별 예제
---------------------

以下は각 언어별 예제です。


.. note:: 

   execution_time は long タイプを使用しています。そのため、execution_time を入力する際は Access Token 발급の際に確認した時刻の次に L を追加してください。  



.. code-tabs::

    .. code-tab:: java
        :title: Java

        import java.security.KeyFactory;
        import java.security.spec.PKCS8EncodedKeySpec;
        import java.security.PrivateKey;
        import java.security.Signature;
         
        //private key
        String privateKeyHexStr = "발급 받은 private key(String)";    //会社コネクト > API Key の Private key 値を入力
        KeyFactory keyFact = KeyFactory.getInstance("EC");
        PKCS8EncodedKeySpec psks8KeySpec = new PKCS8EncodedKeySpec(new BigInteger(privateKeyHexStr,16).toByteArray());
        PrivateKey privateKey = keyFact.generatePrivate(psks8KeySpec);
         
        //execution_time - サーバーの現在時刻
        //long execution_time = new Date().getTime();
        long execution_time = 1611537340731L;     //Access_token 발급の際に취득した execute_time をこちらに入力。 long タイプのため、발급された時刻の次に L 追加     
        String execution_time_str = String.valueOf(execution_time);
         
        //eformsign_signature 생성
        Signature ecdsa = Signature.getInstance("SHA256withECDSA");
        ecdsa.initSign(privateKey);
        ecdsa.update(execution_time_str.getBytes("UTF-8"));
        String eformsign_signature = new BigInteger(ecdsa.sign()).toString(16);
         
         
        //現在時刻および現在時刻署名値
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
        privateKeyHex = "발급された private key(String)"
        privateKey = SigningKey.from_der(binascii.unhexlify(privateKeyHex))
         
        # execution_time - サーバーの 現在時刻
        execution_time = int(time() * 1000)
          
        # eformsign_signature 생성
        eformsign_signature = privateKey.sign(execution_time.encode('utf-8'), hashfunc=hashlib.sha256, sigencode=sigencode_der)
          
        # 現在時刻および現在時刻署名値
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
         
         
        define('PRIVATE_KEY', '발급 받은 private key(String)');
         
         
        //private key 세팅
        $privateKey = new PrivateKey(PRIVATE_KEY);
         
         
        //execution_time - サーバーの現在時刻
        $execution_time = eformsignECDSA\getNowMillisecond();
         
         
        //eformsign_signature 생성
        $signature = eformsignECDSA\Sign(execution_time, $privateKey);
         
         
        //現在時刻および現在時刻署名値
        print 'execution_time : ' . execution_time . PHP_EOL;
        print 'eformsign_signature : ' . bin2hex($signature) . PHP_EOL;
        ?>







API 提供リスト
======================

eformsign API は、署名 생성のための API と文書の作成や処理のための API からなります。

署名 생성のための API
-------------------------

署名 생성のために、まず `Access Token API <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth>`_\を활용해 주십시오. 

``POST``: `/api_auth/access_token <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth#/eformsign/post-api_auth-access_token>`_\  Access Token 발급


Access Token API についての詳しい説明は 
`以下 <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth>`__\ で確認することができます。

.. caution:: 
   
   署名 생성には30秒の時間制限があります。30秒以内に署名を登録し、トークンを발급する必要があります。 
   また、サーバー上の時間と現在時刻が一致しない場合があります。Access Token API を呼び出し、수신한 응답 메시지の"execution_time"を確認してください。

   .. code:: JSON

      { "code": "4000002", "ErrorMessage": "The validation time has expired.",     "execution_time": 1611538409405 }

   `次 <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth>`__\ の예제 位置にも"execution_time"を入力してください。
   
   |image5| 

   Access Token は、멤버 권한에 대해서도 발급받을 수 있습니다. 멤버に対する Access Token を발급받으려면 次のように "execution_time" と一緒に "member_id" を入力してください。 
   
   |image6| 


   이후 해당 APIを실행すると、Access Token が발급되며, 次のような형태の응답을 수신할 수 있습니다.

   .. code:: JSON

      { "api_key": { "name": "애플리케이션_", "alias": "テスト용", "company": { "company_id": "dec5418e58694d90a65d6c38e3d226db", "name": "サンプルデモ", "api_url": "https://kr-api.eformsign.com" } }, "oauth_token": { "expires_in": 3600, "token_type": "JWT", "refresh_token": "8fd0a3c1-44dc-4a03-96ad-01fa34cd159c", "access_token": "eyJhbGciOiJSUzI1NiJ9.eyJpc3MiOiJlZm9ybXNpZ24uaWFtIiwiY29udGV4dCI6eyJjbGllbnRJZCI6IjY4MDk0ZWVhMjVhZjRhNjI5ZTI4ZGU5Y2ZlYzRlYmZjIiwiY2xpZW50S2V5IjoiZTNiM2IzZTUtMGEzMS00NTE1LWE5NzEtN2M4Y2FlNDI4NzZmIiwibWFuYWdlbWVudElkIjoiMzRhYWI4MDBjMmEwNDQwNThmZDRlZjc5OGFlY2RlY2EiLCJzY29wZXMiOiJzbWFydF9lZm9ybV9zY29wZSIsInR5cGUiOiJ1c2VyIiwidXNlck5hbWUiOiIzMmIzZDRmOC00MjdkLTRjZjQtOTZiYS1mYzAxNjIxNWRkNDciLCJ1c2VySWQiOiJhNTEyNGVkNmU2M2Y0OTMzOGJlOTA0MjVhNjFkYjlmNSIsInJlZnJlc2hUb2tlbiI6IjhmZDBhM2MxLTQ0ZGMtNGEwMy05NmFkLTAxZmEzNGNkMTU5YyJ9LCJjbGFpbSI6eyJjb21wYW55X2lkIjoiZGVjNTQxOGU1ODY5NGQ5MGE2NWQ2YzM4ZTNkMjI2ZGIiLCJhY2Nlc3Nfa2V5IjoiMzJiM2Q0ZjgtNDI3ZC00Y2Y0LTk2YmEtZmMwMTYyMTVkZDQ3In0sImV4cCI6MTYxMTU0MjIzNiwiaWF0IjoxNjExNTM4NjM2fQ.BltoXXBSabjXfpyLsZik9OZTE5XtLqe9lguMmJ_qfwZN1NyoVoxDqA5y1-_TLis7FvvNjfI1eegOroCZDZPFyXRaBxAj0CW8TijVjbhliJBuccHFyKXaJxmo_GMmTHYtxNNB1SUgLeFIrYROnpQndU8J7ZkfPDgYGwh1YSx-5s4" } }





.. caution:: 
   
   발급した API キーは、 `次 <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_auth>`__\ の位置にある **Authorize** ボタン (|image4|) をクリックして登録してください。ただし、API キー値には**必ず Base64**\ で인코딩한 문자열を入力しなければなりません。https://www.base64encode.org/ に접속し、발급された API キーを入力して인코딩된 텍스트を받아 삽입하세요.



文書の作成および처리のための API
----------------------------------

署名を登録すると、次の文書 API を使用して文書の新規作成や文書情報の照会ができ、完了文書ファイル (文書 PDF、감사추적증명서) や文書の添付ファイルをダウンロードすることができます。 


.. note:: 

   본 文書 API を使用するためには、先に Access Token の발급が必要です。

現在提供している `文書 API <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_general>`_\は以下のとおりです。


``POST``: `/api_auth/refresh_token <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_general#/eformsign/post-api_auth-refresh_token>`_\  Access Token 갱신

``POST``: `/api/documents <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_general#/eformsign/post-api-documents>`_\  文書の新規作成(내부 수신자)

``POST``: `/api/documents/external <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_general#/eformsign/post-api-documents-external>`_\  文書の新規作成(외부 수신자)

``GET``: `/api/documents/{document_id} <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_general#/eformsign/get-api-documents-DOCUMENT_ID>`_\  文書情報 照会

``GET``: `/api/docuemnts/{document_id}/download_files <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_general#/eformsign/get-api-documents-DOCUMENT_ID-download_files>`_\  文書ファイル 다운로드(文書 PDF, 감사추적증명서)

``GET``: `/api/doduments/{document_id}/download_attach_files <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_general#/eformsign/get-api-documents-DOCUMENT_ID-download_attach_files>`_\  文書 첨부 ファイル 다운로드


各 eformsign 文書 API についての詳しい説明は 
`次 <https://app.swaggerhub.com/apis/eformsign_api/eformsign_API_2.0/2.0_general>`__\ で確認することができます。



API 関連情報
===================

API ステータスコード
--------------

API ステータスコードは、以下の通りです。

200
'''''''

===========  ===============  ===================================
Code         説明              備考
===========  ===============  ===================================
200          성공              성공
===========  ===============  ===================================


202
'''''''

===========  ===============  =====================================================================================
Code         説明              備考
===========  ===============  =====================================================================================
2020001      PDF 생성中        - PDF ファイルでダウンロードする際、비동기で생성されるため 文書保存後にPDF생성まで時間が所要 
                              - 수초에서 수분 내에 재 要請시 다운로드 가능
===========  ===============  =====================================================================================


400
'''''''

===========  ===================  =======================================================
Code         説明                  備考
===========  ===================  =======================================================
4000001      必須入力値漏れ         API の必須入力値(ヘッダー値またはパラメーター)が入力されていない場合                        
4000002      時間切れ              API認証の要請時間が時間切れとなった場合
4000003      APIキーが存在しない      削除されたAPI キーか入力ミスの場合
4000004      文書が存在しない        間違った文書 IDを入力した場合
===========  ===================  =======================================================


403
'''''''

===========  =========================  ==========================================
Code         説明                        備考
===========  =========================  ==========================================
4030002      Access Token認証エラー        Access Tokenが正しくない場合
4030003      Refresh Token認証エラー       Refresh Tokenが正しくない場合
4030004      署名値検証エラー               署名値が正しくない場合
4030005      サポートしないAPI               サポートしないAPIを呼び出した場合
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
Start         00               スタート段階
Complete      01               完了段階
Approval      02               決裁段階
External      03               外部受信者段階
Accept        04               内部受信者段階
===========  ===============  ===================================


User タイプ
--------------

================  ===============  ===================================
Type               Code             説明
================  ===============  ===================================
内部メンバー          01               内部メンバー인지 여부
외부 使用자         02                内部メンバー가 아닌 외부 使用자 여부
================  ===============  ===================================

Status タイプ
--------------

======================  ===============  ===================================
Type                     Code             説明
======================  ===============  ===================================
doc_tempsave             001              下書き (作成者が下書き保存した状態)
doc_create               002              作成
doc_complete             003              完了
doc_update               043              修正
doc_request_delete       047              削除要請
doc_delete               049              削除
doc_request_revoke       040              キャンセル要請
doc_revoke               041              キャンセル
doc_request_reject       045              返戻要請
doc_request_approval     010              決裁要請
doc_accept_approval      012              決裁承認
doc_reject_approval      011              決裁返戻
doc_cancel               013              決裁キャンセル
doc_request_reception    020              内部メンバー要請
doc_accept_reception     022              内部メンバー承認
doc_reject_reception     021              内部メンバー返戻
doc_request_outsider     030              外部受信者要請
doc_accept_outsider      032              外部受信者承認
doc_reject_outsider      031              外部受信者返戻
======================  ===============  ===================================


」
Action タイプ
--------------

======================  ===============  ===================================
Type                     Code             説明
======================  ===============  ===================================
doc_tempsave             001              下書き保存
doc_create               002              文書 생성
doc_complete             003              完了
doc_request_approval     010              決裁要請
doc_reject_approval      011              決裁返戻
doc_accept_approval      012              決裁承認
doc_cancel               013              決裁キャンセル
doc_request_reception    020              内部メンバー要請
doc_reject_reception     021              内部メンバー返戻
doc_accept_reception     022              内部メンバー承認
doc_accept_tempsave      023              内部メンバーが下書きとして保存
doc_request_outsider     030              外部受信者要請
doc_reject_outsider      031              外部受信者返戻
doc_accept_outsider      032              外部受信者承認
doc_rerequest_outsider   033              外部受信者再要請
doc_open_outsider        034              外部受信者閲覧
doc_outsider_tempsave    035              外部受信者が下書きとして保存
doc_request_revoke       040              キャンセル要請
doc_refuse_revoke        041              キャンセル要請返戻拒否
doc_revoke               042              キャンセル
doc_update               043              修正
doc_cancel_update        044              修正キャンセル
doc_request_reject       045              返戻要請
doc_refuse_reject        046              返戻要請返戻
doc_request_delete       047              削除要請
doc_refuse_delete        048              削除要請返戻
doc_delete               049              削除
doc_complete_send_pdf    050              完了文書をPDFとして送信
doc_transfer             051              移管
======================  ===============  ===================================




.. |image1| image:: resources/company_id.png
   :width: 600px
.. |image2| image:: resources/column_icon.png
.. |image3| image:: resources/document_id.png
.. |image4| image:: resources/authorize_icon.png
.. |image5| image:: resources/execution_time.png
   :width: 450px
.. |image6| image:: resources/execution_time2.png
   :width: 450px