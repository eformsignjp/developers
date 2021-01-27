--------------------------
eformsign API 사용하기
--------------------------

eformsign이 제공하는 API를 사용해 eformsign의 기능을 고객의 시스템/서비스에서 호출해 사용할 수 있도록 해주는 기능입니다.


시작하기 
=========

.. _apikey:

API 키 발급하기
-----------------

1. eformsign에 대표 관리자로 로그인 후, 메뉴 트리에서 **[커넥트] > [API / Webhook]** 페이지로 이동합니다. 

.. image:: resources/apikey1.PNG
    :width: 700
    :alt: 커넥트 > API/Webhook 메뉴 위치


2. **[API 키 관리]** 탭을 선택하고 **API 키 생성** 버튼을 클릭합니다.

.. image:: resources/apikey2.PNG
    :width: 700
    :alt: API 키 생성 버튼


3. API 키 생성 팝업창에 별칭과 애플리케이션 이름을 입력하고 저장 버튼을 클릭합니다.

.. image:: resources/apikey3.PNG
    :width: 700
    :alt: API 키 생성 팝업창


4. 생성된 키 목록에서 **키보기** 버튼을 클릭하여 API 키와 비밀 키를 확인합니다.

.. image:: resources/apikey4.PNG
    :width: 700
    :alt: API 키보기 버튼 위치

.. image:: resources/apikey5.PNG
    :width: 700
    :alt: API 키 및 비밀 키 확인 



.. note:: **API 키 수정 방법**

    생성된 키 목록에서 **수정** 버튼을 클릭하면 별칭과 어플리케이션 이름을 수정할 수
  	있습니다. 또한, 상태 영역을 클릭하여 활성/비활성 상태로 변경할 수도 있습니다.

.. note:: **API 키 삭제 방법**

    생성된 키 목록에서 **삭제** 버튼을 클릭하여 API 키를 삭제할 수 있습니다.


서명 생성하기 
==============

eformsign_signature는 비대칭 키 방식과 타원곡선 암호화(Elliptic curve cryptography)를 사용하고 있습니다.

.. tip:: 
   
   타원곡선 암호화는 공개키 암호화 방식 중 하나로, 데이터 암호화 디지털 인증 등 현재 가장 많이 쓰이는 암호방식입니다. 


서명 생성 방법에 대해서는  Java, Python, PHP 언어별로 설명합니다.

Java
-------

서버의 현재 시간을 String(UTF-8)으로 변환하고, `API Key 발급하기 <#apikey>`__\에서 발급받은 private key로 서명한 후, 서명한 데이터를 hex string으로 변환합니다.

.. note:: 
  서명 알고리즘은 SHA256withECDSA을 사용합니다.


Python
-------

키 포맷 처리용 라이브러리를 사용해야 합니다. 작업전 다음의 명령어를 통해 해당 라이브러리를 설치해 주십시오.

.. code:: python

   pip install https://github.com/warner/python-ecdsa/archive/master.zip


PHP
-------

다음 예제의 keycheck.inc.php, test.php 파일이 동일한 패스에 위치하게 한 후에 진행해야 합니다.

다음은 각 언어별 예제입니다.

.. code-tabs::

    .. code-tab:: java
        :title: Java

        import java.security.KeyFactory;
        import java.security.spec.PKCS8EncodedKeySpec;
        import java.security.PrivateKey;
        import java.security.Signature;
         
        //private key
        String privateKeyHexStr = "발급 받은 private key(String)";
        KeyFactory keyFact = KeyFactory.getInstance("EC");
        PKCS8EncodedKeySpec psks8KeySpec = new PKCS8EncodedKeySpec(new BigInteger(privateKeyHexStr,16).toByteArray());
        PrivateKey privateKey = keyFact.generatePrivate(psks8KeySpec);
         
        //execution_time - 서버 현재 시간
        long execution_time = new Date().getTime();
        String execution_time_str = String.valueOf(execution_time);
         
        //eformsign_signature 생성
        Signature ecdsa = Signature.getInstance("SHA256withECDSA");
        ecdsa.initSign(privateKey);
        ecdsa.update(execution_time_str.getBytes("UTF-8"));
        String eformsign_signature = new BigInteger(ecdsa.sign()).toString(16);
         
         
        //현재 시간 및 현재 시간 서명값
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
        privateKeyHex = "발급 받은 private key(String)"
        privateKey = SigningKey.from_der(binascii.unhexlify(privateKeyHex))
         
        # execution_time - 서버 현재 시간
        execution_time = int(time() * 1000)
          
        # eformsign_signature 생성
        eformsign_signature = privateKey.sign(execution_time.encode('utf-8'), hashfunc=hashlib.sha256, sigencode=sigencode_der)
          
        # 현재 시간 및 현재 시간 서명값
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
         
         
        //execution_time - 서버 현재 시간
        $execution_time = eformsignECDSA\getNowMillisecond();
         
         
        //eformsign_signature 생성
        $signature = eformsignECDSA\Sign(execution_time, $privateKey);
         
         
        //현재 시간 및 현재 시간 서명값
        print 'execution_time : ' . execution_time . PHP_EOL;
        print 'eformsign_signature : ' . bin2hex($signature) . PHP_EOL;
        ?>



API 테스트해보기
======================================

생성한 eformsign_signature를 테스트해 봅니다. 

다음의 eformsign_signature 생성 및 검증용 샘플은 Open API 또는 Webhook의 서명값을 생성 및 검증하는 테스트 샘플 소스코드 입니다.

.. note::

   샘플 키를 사용하고 있어 실 사용시에는 정상 동작 하지 않습니다. 생성하신 서명 값의 검증용으로만 사용해 주세요.


Java
-------

다음의 샘플 키로 서명 및 검틍 테스트를 해보시기 바랍니다.


Python
-------

키 포맷 처리용 라이브러리를 사용해야 합니다. 작업전 다음의 명령어를 통해 해당 라이브러리를 설치해 주십시오.

.. code:: python

   pip install https://github.com/warner/python-ecdsa/archive/master.zip


PHP
-------

다음 예제의 keycheck.inc.php, test.php 파일이 동일한 패스에 위치하게 한 후에 진행해야 합니다.

다음은 각 언어별 테스트 키와 예제입니다.

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




Open API 리스트
=================

API 상태 코드
--------------

API 상태 코드는 다음과 같습니다.

200
'''''''

===========  ===============  ===================================
Code         설명              비고
===========  ===============  ===================================
200          성공              성공
===========  ===============  ===================================


202
'''''''

===========  ===============  ===================================
Code         설명              비고
===========  ===============  ===================================
2020001      PDF 생성 중        - PDF 파일 다운로드 시 파일은 비동기로 생성되기 때문에 문서 저장 후 PDF 생성까지 추가 시간 필요. 
                                - 수초에서 수분 내에 재 요청시 다운로드 가능.
===========  ===============  ===================================


400
'''''''

===========  ===================  ==========================================
Code         설명                  비고
===========  ===================  ==========================================
4000001      필수 입력값 누락      API 의 필수 입력값(헤더 값 또는 페러미터)이 누락 했을 경우                        
4000002      인증 시간 만료        API 인증 요청 시간이 만료 되었을 경우
4000003      API 키 없음           API 키가 삭제 되었거나 잘못 입력했을 경우
4000004      문서가 없음           문서 ID를 잘못 입력 했을 경우
===========  ===================  ==========================================


403
'''''''

===========  =========================  ==========================================
Code         설명                        비고
===========  =========================  ==========================================
4030002      Access Token 인증 오류      Access Token이 올바르지 않을 경우
4030003      Refresh Token 인증 오류     Refresh Token이 올바르지 않을 경우
4030004      서명값 검증 실패            서명값이 올바르지 않을 경우
4030005      지원하지 않는 API           지원하지 않는 API 호출 시
===========  =========================  ==========================================

405
'''''''

===========  =====================  ===================================
Code         설명                    비고
===========  =====================  ===================================
4050001      지원하지 않는 method     지원하지 않는 method 호출 시
===========  =====================  ===================================


500
'''''''

===================  ===============  ===================================
Code                 설명              비고
===================  ===============  ===================================
5000001~5000003      서버 오류         서버에 오류가 발생 했을 경우
===================  ===============  ===================================




Step 타입
--------------

===========  ===============  ===================================
Type         Code             설명
===========  ===============  ===================================
Start         00               시작단계
Complete      01               완료단계
Approval      02               결재단계
External      03               외부수신자단계
Accept        04               내부수신자단계
===========  ===============  ===================================


User 타입
--------------

===========  ===============  ===================================
Type         Code             설명
===========  ===============  ===================================
내부 멤버     01                내부 멤버인지 여부
외부 사용자   02                내부 멤버가 아닌 외부 사용자 여부
===========  ===============  ===================================

Status 타입
--------------

======================  ===============  ===================================
Type                     Code             설명
======================  ===============  ===================================
doc_tempsave             001              초안 (최초 작성자 임시 저장 상태)
doc_create               002              문서 작성
doc_complete             003              문서 완료
doc_update               043              문서 수정
doc_request_delete       047              문서 삭제 요청
doc_delete               049              문서 삭제
doc_request_revoke       040              문서 취소 요청
doc_revoke               041              문서 취소
doc_request_reject       045              문서 반려 요청
doc_request_approval     010              문서 결재 요청
doc_accept_approval      012              문서 결재 승인
doc_reject_approval      011              문서 결재 반려
doc_cancel               013              문서 결재 취소
doc_request_reception    020              문서 내부자 요청
doc_accept_reception     022              문서 내부자 승인
doc_reject_reception     021              문서 내부자 반려
doc_request_outsider     030              문서 외부자 요청
doc_accept_outsider      032              문서 외부자 승인
doc_reject_outsider      031              문서 외부자 반려
======================  ===============  ===================================



Action 타입
--------------

======================  ===============  ===================================
Type                     Code             설명
======================  ===============  ===================================
doc_tempsave             001              문서 임시 저장
doc_create               002              문서 생성
doc_complete             003              문서 최종 완료
doc_request_approval     010              문서 결재 요청
doc_reject_approval      011              문서 결재 반려
doc_accept_approval      012              문서 결재 승인
doc_cancel               013              문서 결재 취소
doc_request_reception    020              문서 내부자 요청
doc_reject_reception     021              문서 내부자 반려
doc_accept_reception     022              문서 내부자 승인
doc_accept_tempsave      023              내부자 임시 저장
doc_request_outsider     030              문서 외부자 요청
doc_reject_outsider      031              문서 외부자 반려
doc_accept_outsider      032              문서 외부자 승인
doc_rerequest_outsider   033              외부자 재요청
doc_open_outsider        034              외부자 열람
doc_outsider_tempsave    035              외부자 임시 저장
doc_request_revoke       040              문서 취소 요청
doc_refuse_revoke        041              문서 취소 요청 거절
doc_revoke               042              문서 취소
doc_update               043              문서 수정
doc_cancel_update        044              문서 수정 취소
doc_request_reject       045              문서 반려 요청
doc_refuse_reject        046              문서 반려 요청 거절
doc_request_delete       047              문서 삭제 요청
doc_refuse_delete        048              문서 삭제 요청 거절
doc_delete               049              문서 삭제
doc_complete_send_pdf    050              완료 문서 pdf 전송
doc_transfer             051              문서 이관
======================  ===============  ===================================


swagger 링크


