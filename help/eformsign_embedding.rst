
======================================
eformsign 기능 직접 삽입하기
======================================


eformsign 기능을 직접 삽입해 연계하게 되면, 고객이 제공하고 있는 서비스(혹은 사이트) 내에서 고객사의 사용자(최종 사용자)가 eformsign 서비스 사이트를 통하지 않고 고객의 서비스를 통해 자연스럽게 eformsign 전자문서를 사용할 수 있습니다.
예로 들면, 내 블로그에서 특정 YouTube 동영상을 소개하려고 할 때, 블로그에서 YouTube 동영상을 직접 삽입함으로써 해당 동영상을 블로그 내에서 바로 플레이할 수 있도록 하는 방식과 유사합니다.

------------
시작하기
------------


설치하기
=============

eformsign의 기능을 사용하고자 하는 웹 페이지에 다음의 스크립트를 추가합니다.

.. code-block:: Javascript

   //jquery
   <script src="https://www.eformsign.com/plugins/jquery/jquery.min.js"/>
   //eformsign embedded script
   <script src="https://www.eformsign.com/lib/js/efs_embedded_v2.js"/>
   //eformsign redirect script
   <script src="https://www.eformsign.com/lib/js/efs_redirect_v2.js"/>


.. note::

   eformsign 기능을 삽입하고자 하는 페이지에 이 스크립트를 추가하면 eformsign 객체를 전역변수로 사용할 수 있습니다.


--------------------------
eformsign 객체에 대하여
--------------------------

eformsign 객체는 embedding과 redirect의 두 가지 타입으로 구성되어 있습니다.


+----------+--------------------+--------------------------------------+
| Type     | Name               | 설명                                 |
+==========+====================+======================================+
| embedding| eformsign.         | eformsign을 삽입해 문서를 작성할 수  |
|          | document(          | 있도록 해주는 함수                   |
|          | document_option,   |                                      |
|          | iframe_id,         | callback 파라미터는 옵션             |
|          | success_callback,  |                                      |
|          | error_callback)    | -  document_option, iframe_id: 필수  |
|          |                    |                                      |
|          |                    | -  success_callback: 옵션            |
|          |                    |                                      |
|          |                    | -  error_callback: 옵션              |
+----------+--------------------+--------------------------------------+
| redirect | eformsign.documen  | eformsign으로의 페이지 전환 방식으로 |
|          | t(document_option) | 문서를 작성할 수 있도록 해주는 함수  |
|          |                    |                                      |
|          |                    | -  document_option : 필수            |
+----------+--------------------+--------------------------------------+




.. note::

   redirect 방식은 추후 공개할 예정입니다. 

.. code-block::javascript

     var eformsign = new EformSign();
     
     var document_option = {
       "company" : {
          "id" : '', // company id 입력
          "country_code" : "", // 국가 코드 입력 (ex: kr)
          "user_key": ""  // 고객 시스템의 고유한 Key (고객시스템에 로그인한 사용자의 unique key) - option
       },
       "user" : {
            "type" : "01" ,
            "access_token" : "", // access Token 입력 openAPI accessToken 참조
            "refresh_token" : "", // refresh Token 입력 openAPI accessToken 참조
            "external_token" : "", // 외부자 처리 시 external Token 입력 openAPI accessToken 참조
            "external_user_info" : {
               "name" : "" // 외부자 처리 시 외부자 이름 입력
            }
        },
        "mode" : {
            "type" : "02",
            "template_id" : "", // template id 입력
            "document_id" : ""  // document_id 입력
        },
        "prefill" : {
            "document_name": "", // 문서 제목 입력
            "fields": [ {
                "id" ; "고객명",
                "value" : "홍길동",
                "enabled" : true,
                "required" : true 
            }]
        },
        "return_fields" : ['고객명']
     };
     
     //callback option
     var success_callback = function(response){ 
        console.log(response.code); 
        if( response.code == "-1"){
            //문서 작성 성공
            console.log(response.document_id);
            // return_fields에 넘긴 데이터를 조회 가능함. fields는 폼을 작성할 때 만든 입력 컴포넌트의 id를 의미함.
            console.log(response.field_values["company_name"]);
            console.log(response.field_values["position"]);
        }
     };
      
     var error_callback = function(response){
        console.log(response.code); 
        //문서 작성 실패
        alert(response.message);
         
     };
     
     eformsign.document(document_option , "eformsign_iframe" , success_callback , error_callback  );


embedding_document 함수
===========================


eformsign을 삽입해 고객사의 사이트/서비스에서 문서를 작성할 수 있도록 해주는 함수입니다. eformsign 내 document 함수를 호출해 사용하세요.

크게 document_option과 callback의 2가지 파라미터를 사용할 수 있습니다.

.. note::

   함수 형태
   document(document_option, iframe_id, success_callback , error_callback)

===================  ===============  ==========  ==========================================================
 Paramter Name       Paramter Type    필수여부      설명 
===================  ===============  ==========  ==========================================================
 document_option      Json             필수         임베딩하여 eformsign 구동시, document 관련된 옵션을 지정 
 iframe_id            String           필수         임베딩되어 표시될 iframe id 
 success_callback     function         비필수       eformsign 문서 작성 성공 시, 호출될 callback 함수
 error_callback       function         비필수       eformsign 문서 작성 실패 시, 호출될 callback 함수 
===================  ===============  ==========  ==========================================================



.. code-block:: javascript

     var eformsign = new EformSign();
     var document_option = {
        "company": {
            "id": '', // company id 입력
            "country_code": "", // 국가 코드 입력 (ex: kr)
            "user_key": '' // 고객 시스템의 고유한 Key (고객시스템에 로그인한 사용자의 unique key) - option
        },
        "user": {
            "type": "01",
            "access_token": "", // access Token 입력 openAPI accessToken 참조
            "refresh_token": "", // refresh Token 입력 openAPI accessToken 참조
            "external_token": "", // 외부자 처리 시 external Token 입력 openAPI accessToken 참조
            "external_user_info": {
                "name": "" // 외부자 처리 시 외부자 이름 입력
            }
        },
        "mode": {
            "type": "02",
            "template_id": "", // template id 입력
            "document_id": "" // document_id 입력
        },
        "prefill": {
            "document_name": "", // 문서 제목 입력
            "fields": [{
                "id" : "",
                "고객명" : "",
                "value": "홍길동",
                "enabled": true,
                "required": true
            }]
        },
        "return_fields": ['고객명']
     };
     
     //callback option
     var success_callback = function (response) {
        console.log(response.code);
        if (response.code == "-1") {
            //문서 작성 성공
            console.log(response.document_id);
            // return_fields에 넘긴 데이터를 조회 가능함. fields는 폼을 작성할 때 만든 입력 컴포넌트의 id를 의미함.
            console.log(response.field_values["company_name"]);
            console.log(response.field_values["position"]);
        }
     };
     
     
     var error_callback = function (response) {
        console.log(response.code);
        //문서 작성 실패
        alert(response.message);
     
     };
     
     eformsign.document(document_option, "eformsign_iframe", success_callback, error_callback);


파라미터 설명: document-option
================================


document-option에서는 크게 회사 정보, 유저 정보, 모드, 리턴 필드, 자동 기입에 대해 설정할 수 있습니다. 회사 정보와 모드는 필수 입력정보입니다. 

1. 회사 정보(필수)
-------------------------

.. code-block:: javascript

   var document_option = {
     "company" : {
         "id" : 'f9aec832efef4133a1e849efaf8a9aed',  // 회사의 id - 회사관리 - 회사 정보 에서 확인 - 필수
         "country_code" : "kr", // 비필수 이나, 지정해 주는 것이 좋음. ( 회사 관리의 회사 정보에서 국가에 대한 코드를 지정 ) - 빠른 open이 가능함.
         "user_key": "eformsign@forcs.com"
     }
 };


2. 유저 정보(비필수)
---------------------------

**회사 내 멤버 로그인을 통한 신규 작성**
    - 유저 정보를 지정하지 않을 경우에 해당하며, 유저 정보를 지정하지 않습니다.	
    - 이 경우, eformsign 로그인 페이지가 기동되며, 로그인 과정 이후에 문서를 작성할 수 있게 됩니다.

**회사 내 멤버의 토큰을 이용한 작성(신규 및 수신한 문서 포함)**	
    - 임베딩시, eformsign 로그인 과정 없이, 특정 계정의 token을 이용하여 문서를 작성 및 수신한 문서를 작성합니다.
    - 토큰 발급 방법은 Open API의 Access token 발급을 통해 가능합니다.

.. code-block:: javascript

    var document_option = {
        "user":{
            "type" : "01" , // 01 - internal or  02 - external  (필수)
            "access_token" : "", // access Token 입력 openAPI accessToken 참조
            "refresh_token" : "", // refresh Token 입력 openAPI accessToken 참조
        }
    };


**회사 내 멤버가 아닌 사용자가 신규 문서 작성**  
    - eformsign의 회원이 아닌 사용자로 하여금 문서를 작성하게 하는 방식

.. code-block:: javascript

    var document_option = {
        "user":{
            "type" : "02" , // 01 - internal or  02 - external  (필수)
            "external_user_info" : {
                "name" : "" // 외부자 처리 시 외부자 이름 입력
            }
        }
    };

**회사 내 멤버가 아닌 사용자가 수신한 문서를 작성**
    - 임베딩시, eformsign의 회원이 아닌 사용자가 수신한 문서를 작성하게 하는 방식

.. code-block:: javascript 

    var document_option = {
        "user":{
        "type" : "02" , // 01 - internal or  02 - external  (필수)
        "external_token" : "", // 외부자 처리 시 external Token 입력 openAPI accessToken 참조
        "external_user_info" : {
        "name" : "" // 외부자 처리 시 외부자 이름 입력
            }
        }
    };

.. code-block:: javascript

    var document_option = {
        "user":{
            "type" : "01" , // 01 - internal or  02 - external  (필수)
            "access_token" : "", // access Token 입력 openAPI accessToken 참조
            "refresh_token" : "", // refresh Token 입력 openAPI accessToken 참조
            "external_token" : "", // 외부자 처리 시 external Token 입력 openAPI accessToken 참조
            "external_user_info" : {
               "name" : "" // 외부자 처리 시 외부자 이름 입력
            }
        }
    };


3. 모드(필수)
---------------------

**템플릿을 이용한 신규 작성** 
    - 템플릿을 이용하여 문서를 새로 작성합니다.

.. code-block:: javascript

    var document_option = {
        "mode" : {
        "type" : "01" ,  // 01 : 문서 작성 , 02 : 문서 처리 , 03 : 미리 보기
        "template_id" : "" // template id 입력
        }
    }

**수신한 문서에 추가 작성** 
    - 수신한 문서에 대해 추가 작성합니다.	

.. code-block:: javascript

    var document_option = {
        "mode" : {
        "type" : "02" ,  // 01 : 문서 작성 , 02 : 문서 처리 , 03 : 미리 보기
        "template_id" : "", // template id 입력
        "document_id" : ""  // document_id 입력
        }
    }

**특정한 문서를 미리보기**
    - 작성된 문서를 미리보기합니다.

.. code-block:: javascript

    var document_option = {
        "mode" : {
        "type" : "03" ,  // 01 : 문서 작성 , 02 : 문서 처리 , 03 : 미리 보기
        "template_id" : "", // template id 입력
        "document_id" : ""  // document_id 입력
        }
    }

.. code-block:: javascript

    var document_option = {
      "mode" : {
        "type" : "01" ,  //01 : 문서 작성 , 02 : 문서 처리 , 03 : 미리 보기
        "template_id" : "", // template id 입력
        "document_id" : ""  // document_id 입력
      }
    }


4. 리턴 필드(비필수)
--------------------------

문서 작성 및 수정 후, 사용자가 작성한 필드의 내용 중 callback 함수를 통해 받을 수 있는 항목을 지정합니다.
    
.. note::

   미 지정시 기본 필드만 제공합니다. 관련 내용은 callBack 파라미터를 참고하세요.

.. code-block:: javascript

    var document_option = {
       "return_fields" : ['고객명']
    }

5. 자동 기입(문서 작성 과정 중에 자동으로 기입 처리시 사용)
-----------------------------------------------------------

**문서 제목**
    - document_name에 작성할 문서의 제목을 지정합니다.

.. code-block:: javascript

    var document_option = {
        "prefill" : {
            "document_name": "휴가신청서"
        }
    }

**문서내 필드 설정 기입** 
    - 폼 생성시에 지정한 입력 컴포넌트의 ID를 기준으로, 필드 초기값 및 활성 여부, 필수 여부를 지정합니다.

  
.. note::

   - enabled
     - 지정하지 않을 경우, 템플릿 설정의 항목 제어 옵션에 따른다
     - 지정할 경우, 템플릿 설정의 항목 제어 옵션보다 우선한다
   - required
     - 지정하지 않을 경우, 템플릿 설정의 항목 제어 옵션에 따른다
     - 지정할 경우, 템플릿 설정의 항목 제어 옵션보다 우선한다
   - value
     - 지정하지 않을 경우, 신규 작성 시에 템플릿 설정의 필드 설정 옵션을 따른다
     - 지정할 경우, 템플릿 설정의 필드 설정보다 우선한다

           
.. code:: javascript
    var document_option = {
        "prefill" : {
        "fields": [ {
            "id" ; "고객명",
            "value" : "홍길동",
            "enabled" : true,
            "required" : true 
        }]
    }
    }

.. code-block:: javascript
    var document_option = {
        "prefill": {
            "document_name": "",
            "fields": [
                {
                    "id": "고객명",
                    "value": "홍길동",
                "enabled": true,
                    "required": true
                }
            ]
        }
    };




파라미터 설명: Callback
============================

==================  ===============  ===========  ===================================================
 Paramter Name       Paramter Type    필수 여부     설명        
==================  ===============  ===========  ===================================================
 success_callback    function         비필수        eformsign 문서 작성 성공 시, 호출될 callback 함수 
 error_callback      function         비필수        eformsign 문서 작성 실패 시, 호출될 callback 함수
==================  ===============  ===========  ===================================================

Callback 함수는 다음과 같이 설정합니다. 

.. code-block:: javascript
   var eformsign = new eformsign(); // iframe document 함수 인자로 이동
 
 
   var document_option = {};
 
 
  var sucess_callback= funtion(response){
    console.log(response.document_id);
    console.log(response.title);
    console.log(response.field_values["name"]);
  };
 
 
  var error_callback= funtion(response){
    alert(response.message);
    console.log(response.code); 
    console.log(response.message);
  };
 
 
  eformsign.document(document_option , "eformsign_iframe" , sucess_callback , error_callback);

document 함수의 파라미터로 Callback 함수를 설정한 경우, Callback 함수 호출 시에 다음과 같은 값을 반환합니다. 

+----------+--------+--------------------------+----------------------+
| Callback | Type   | 설명                     | 비고                 |
+==========+========+==========================+======================+
| code     | string | 문서 제출 실패시 결과의  | -1 일 경우, 정상     |
|          |        | 오류 코드를 반환한다     |                      |
+----------+--------+--------------------------+----------------------+
| doc      | string | 문서 제출 성공시, 작성한 | ex)                  |
| ument_id |        | 문서의 document_id를     | 910b8a965f9          |
|          |        | 반환한다                 | 402b82152f48c6da5a5c |
+----------+--------+--------------------------+----------------------+
| fiel     | object | document_option에 정의한 | ex).                 |
| d_values |        | return_fields 컬럼에     | field_values["name"] |
|          |        | 사용자가 입력한 값을     | // john              |
|          |        | 가져올 수 있다           |                      |
+----------+--------+--------------------------+----------------------+
| message  | string | 문서 제출 실패시, 오류   | 빈 값일 경우, 정상   |
|          |        | 메시지를 반환한다        |                      |
+----------+--------+--------------------------+----------------------+
| title    | string | 문서 제출 성공시, 작성한 | ex) 계약서           |
|          |        | 문서의 제목을 반환한다   |                      |
+----------+--------+--------------------------+----------------------+

