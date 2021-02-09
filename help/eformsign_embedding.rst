
======================================
eformsign 機能の直接挿入
======================================


eformsign 機能を直接挿入して연계することになると、고객が提供しているサービス（またはサイト）内で고객사のユーザー(최종 ユーザー)が eformsign サービスサイトを介さず自然に eformsign の電子文書を利用することができます。
例えば、ブログで特定のYouTube動画を表示したいとき、ブログに YouTube 動画を直接挿入することで、その動画をブログ内ですぐ再生できるようにする方式と유사합니다.

------------
はじめに
------------


設置
=============

eformsign の機能を使用したい Web ページに次のスクリプトを追加します。

.. code-block:: javascript

   //jquery
   <script src="https://www.eformsign.com/plugins/jquery/jquery.min.js"/>
   //eformsign embedded script
   <script src="https://www.eformsign.com/lib/js/efs_embedded_v2.js"/>
   //eformsign redirect script
   <script src="https://www.eformsign.com/lib/js/efs_redirect_v2.js"/>


.. note::

   eformsign 機能を挿入したいページに上記のスクリプトを追加すると、 eformsign オブジェクトを 전역변수として使用することができます。


-------------------------------
eformsign のオブジェクトについての説明
-------------------------------

eformsign のオブジェクトは、 embedding と redirect の2つのタイプで構成されています。


+----------+--------------------+--------------------------------------+
| Type     | Name               | 説明                                 |
+==========+====================+======================================+
| embedding| eformsign.         | eformsign を挿入し、文書を作成できるように  |
|          | document(          | する関数                              |
|          | document_option,   |                                      |
|          | iframe_id,         | callback パラメーターはオプション            |
|          | success_callback,  |                                      |
|          | error_callback)    | -  document_option, iframe_id：必須  |
|          |                    |                                      |
|          |                    | -  success_callback：オプション         |
|          |                    |                                      |
|          |                    | -  error_callback：オプション           |
+----------+--------------------+--------------------------------------+
| redirect | eformsign.documen  | eformsign へのページ転換方式で           |
|          | t(document_option) | 文書を作成できるようにする関数              |
|          |                    |                                      |
|          |                    | -  document_option：必須             |
+----------+--------------------+--------------------------------------+




.. note::

   redirect 方式は、추후 공개할 예정です。 


.. code-block:: javascript

     var eformsign = new EformSign();
     
     var document_option = {
       "company" : {
          "id" : '', // company id 入力
          "country_code" : "", // 国コード 入力 (ex: kr)
          "user_key": ""  // 고객 システムの固有な Key (고객システムにログインしたユーザーの unique key) - option
       },
       "user" : {
            "type" : "01" ,
            "access_token" : "", // access Token 入力 openAPI accessToken 参照
            "refresh_token" : "", // refresh Token 入力 openAPI accessToken 参照
            "external_token" : "", // 外部処理 시 external Token 入力 openAPI accessToken 参照
            "external_user_info" : {
               "name" : "" // 外部処理 시 外部受信者名を入力
            }
        },
        "mode" : {
            "type" : "02",
            "template_id" : "", // template id 入力
            "document_id" : ""  // document_id 入力
        },
        "prefill" : {
            "document_name": "", // 文書タイトルを入力
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
            //文書作成成功
            console.log(response.document_id);
            // return_fields에 넘긴 データを照会することができる。fieldsとは、フォームを作成するときに作った入力コンポーネントの id を意味する。
            console.log(response.field_values["company_name"]);
            console.log(response.field_values["position"]);
        }
     };
      
     var error_callback = function(response){
        console.log(response.code); 
        //文書作成失敗
        alert(response.message);
         
     };
     
     eformsign.document(document_option , "eformsign_iframe" , success_callback , error_callback  );


embedding_document 関数
===========================

.. note::

   関数タイプ
   document(document_option, iframe_id, success_callback , error_callback)

eformsign を挿入し、고객사のサイト/サービスで文書を作成できるようにする関数です。 eformsign 内の document 関数を呼び出して使用してください。

大きく document_option と callback の2つのパラメーターを使用することができます。


===================  ===============  ==========  ==========================================================
 Paramter Name       Paramter Type    必須入力      説明 
===================  ===============  ==========  ==========================================================
 document_option      Json             O          임베딩하여 eformsign 구동시, document 関連オプションを指定 
 iframe_id            String           O          임베딩되어 표시될 iframe id 
 success_callback     function         X          eformsign 文書作成に成功した場合、呼び出される callback 関数
 error_callback       function         X          eformsign 文書作成に失敗した場合、呼び出される callback 関数 
===================  ===============  ==========  ==========================================================



.. code-block:: javascript

     var eformsign = new EformSign();
     var document_option = {
        "company": {
            "id": '', // company id 入力
            "country_code": "", // 国コード入力 (ex: kr)
            "user_key": '' // 고객 システムの고유한 Key (고객システムにログインしたユーザーの unique key) - option
        },
        "user": {
            "type": "01",
            "access_token": "", // access Token 入力 openAPI accessToken 参照
            "refresh_token": "", // refresh Token 入力 openAPI accessToken 参照
            "external_token": "", // 外部処理時の external Token 入力 openAPI accessToken 参照
            "external_user_info": {
                "name": "" // 外部処理の場合、外部受信者名を入力
            }
        },
        "mode": {
            "type": "02",
            "template_id": "", // template id 入力
            "document_id": "" // document_id 入力
        },
        "prefill": {
            "document_name": "", // 文書タイトルを入力
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
            //文書作成成功
            console.log(response.document_id);
            // return_fields에 넘긴 データを照会することができる。fieldsとは、フォームを作成するときに作った入力コンポーネントの id を意味する。
            console.log(response.field_values["company_name"]);
            console.log(response.field_values["position"]);
        }
     };
     
     
     var error_callback = function (response) {
        console.log(response.code);
        //文書作成失敗
        alert(response.message);
     
     };
     
     eformsign.document(document_option, "eformsign_iframe", success_callback, error_callback);


パラメーターの説明: document-option
================================


document-option では大きく次の5項目について設定することができます。 

- 会社情報
- ユーザー情報
- モード
- リターンフィールド
- 자동 기입

.. note::

   会社情報とモードは必須入力情報です。 



1. 会社情報（必須）
-------------------------

.. code-block:: javascript

   var document_option = {
     "company" : {
         "id" : 'f9aec832efef4133a1e849efaf8a9aed',  // 会社の id - 会社管理 - 会社情報 で確認 - 必須
         "country_code" : "kr", // 必須ではないが、指定することを推奨。（会社管理の 会社情報で国コードを指定） - クィックな빠른 openが可能
         "user_key": "eformsign@forcs.com"
     }
 };


2. ユーザー情報（非必須）
---------------------------

**内部メンバー 로그인을 통한 新規作成**
    - ユーザー情報를 지정하지 않을 경우에 해당하며, ユーザー情報를 지정하지 않습니다.	
    - この場合、eformsign 로그인 ページ가 기동되며, 로그인 과정 이후에 文書를 作成할 수 있게 됩니다.


**内部メンバー의 トークン을 이용한 作成（新規作成および受信した文書を含む）**	
    - 임베딩시, eformsign 로그인 과정 없이, 특정 계정의 token을 이용하여 文書를 作成 및 受信した文書를 作成합니다.
    - トークン 발급 방법은 Open API의 Access token 발급을 통해 가능합니다.

.. code-block:: javascript

    var document_option = {
        "user":{
            "type" : "01" , // 01 - internal or  02 - external  (必須)
            "access_token" : "", // access Token 入力 openAPI accessToken 参照
            "refresh_token" : "", // refresh Token 入力 openAPI accessToken 参照
        }
    };


**内部メンバーではないユーザーが新規文書を作成**  
    - eformsign の会員ではないユーザーが文書を作成できるようにする方式

.. code-block:: javascript

    var document_option = {
        "user":{
            "type" : "02" , // 01 - internal or  02 - external  (必須)
            "external_user_info" : {
                "name" : "" // 外部処理 시 外部受信者名を入力
            }
        }
    };

**内部メンバーではないユーザーが受信した文書を作成**
    - 임베딩시、eformsign の会員ではないユーザーが受信した文書を作成できるようにする方式

.. code-block:: javascript 

    var document_option = {
        "user":{
        "type" : "02" , // 01 - internal or  02 - external  (必須)
        "external_token" : "", // 外部処理 시 external Token 入力 openAPI accessToken 参照
        "external_user_info" : {
        "name" : "" // 外部処理 시 外部受信者名を入力
            }
        }
    };

.. code-block:: javascript

    var document_option = {
        "user":{
            "type" : "01" , // 01 - internal or  02 - external  (必須)
            "access_token" : "", // access Token 入力 openAPI accessToken 参照
            "refresh_token" : "", // refresh Token 入力 openAPI accessToken 参照
            "external_token" : "", // 外部処理 시 external Token 入力 openAPI accessToken 参照
            "external_user_info" : {
               "name" : "" // 外部処理 시 外部受信者名を入力
            }
        }
    };


3. モード(必須)
---------------------

**テンプレートを利用した新規作成** 
    - テンプレートを利用して文書を新規作成します。

.. code-block:: javascript

    var document_option = {
        "mode" : {
        "type" : "01" ,  // 01 : 文書 作成 , 02 : 文書 처리 , 03 : プレビュー
        "template_id" : "" // template id 入力
        }
    }

**受信した文書に追記** 
    - 受信した文書に追記します。	

.. code-block:: javascript

    var document_option = {
        "mode" : {
        "type" : "02" ,  // 01 : 文書 作成 , 02 : 文書 처리 , 03 : プレビュー
        "template_id" : "", // template id 入力
        "document_id" : ""  // document_id 入力
        }
    }

**特定の文書をプレビュー**
    - 作成した文書のプレビューを確認します。

.. code-block:: javascript

    var document_option = {
        "mode" : {
        "type" : "03" ,  // 01 : 文書 作成 , 02 : 文書 처리 , 03 : プレビュー
        "template_id" : "", // template id 入力
        "document_id" : ""  // document_id 入力
        }
    }

.. code-block:: javascript

    var document_option = {
      "mode" : {
        "type" : "01" ,  //01 : 文書 作成 , 02 : 文書 처리 , 03 : プレビュー
        "template_id" : "", // template id 入力
        "document_id" : ""  // document_id 入力
      }
    }


4. リターンフィールド(X)
--------------------------

文書の作成や修正後、ユーザーが作成したフィールドの内容のうち callback 関数でリターンされる項目を指定します。
    
.. note::

   指定しない場合、基本フィールドのみ提供します。詳しい内容は callBack パラメーターをご覧ください。

.. code-block:: javascript

    var document_option = {
       "return_fields" : ['고객명']
    }

5. 자동 기입(文書作成中に自動入力されるよう設定するときに使用)
-----------------------------------------------------------

**文書タイトル**
    - document_name に作成したい文書タイトルを指定します。

.. code-block:: javascript

    var document_option = {
        "prefill" : {
            "document_name": "休暇届"
        }
    }

**文書내 フィールド 設定 기입** 
    - フォーム생성時に指定した入力コンポーネントの ID を基準に、フィールド初期値および활성 여부、必須入力を指定します。

  
.. note::

   - enabled
     - 指定しない場合、テンプレート設定の項目制御オプションに従う
     - 指定する場合、テンプレート設定の項目制御オプションに優先する
   - required
     - 指定しない場合、テンプレート設定の項目制御オプションに従う
     - 指定する場合、テンプレート設定の項目制御オプションに優先する
   - value
     - 指定しない場合、新規作成時にテンプレート設定のフィールド設定オプションに従う
     - 指定する場合、テンプレート設定のフィールド設定に優先する


           
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




パラメーターの説明: Callback
============================

==================  ===============  ===========  =============================================================
 Paramter Name       Paramter Type    必須入力     説明        
==================  ===============  ===========  =============================================================
 success_callback    function         X           eformsign 文書作成に成功した場合、呼び出される callback 関数 
 error_callback      function         X           eformsign 文書作成に失敗した場合、呼び出される callback 関数
==================  ===============  ===========  =============================================================

Callback 関数は、次のように設定します。

.. code-block:: javascript

   var eformsign = new eformsign(); // iframe document 関数因子に移動
 
 
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


document 関数のパラメーターとして Callback 関数を設定した場合、Callback 関数を呼び出す際、次のような値を返します。 

+----------+--------+--------------------------+----------------------+
| Callback | Type   | 説明                     | 備考                 |
+==========+========+==========================+======================+
| code     | string | 文書제출に失敗した場合、     | -1 の場合、正常        |
|          |        | 結果のエラーコードを返す       |                      |
+----------+--------+--------------------------+----------------------+
| doc      | string | 文書제출に成功した場合、     | ex)                  |
| ument_id |        | 作成した文書の document_id | 910b8a965f9          |
|          |        | を返す                    | 402b82152f48c6da5a5c |
+----------+--------+--------------------------+----------------------+
| fiel     | object | document_optionに定義した  | ex).                 |
| d_values |        | return_fields コラムに      | field_values["name"] |
|          |        | ユーザーが入力した値を        | // john              |
|          |        | 가져올 수 있다            |                      |
+----------+--------+--------------------------+----------------------+
| message  | string | 文書제출に失敗した場合、     | Nullの場合、正常       |
|          |        | エラーメッセージを返す          |                      |
+----------+--------+--------------------------+----------------------+
| title    | string | 文書제출に成功した場合、     | ex) 契約書           |
|          |        | 作成した文書タイトルを返す     |                      |
+----------+--------+--------------------------+----------------------+

