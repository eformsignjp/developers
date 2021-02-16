
======================================
eformsign 機能の組み込み
======================================


eformsign 機能を組み込んで連動すると、顧客が提供しているサイト/サービス内で顧客会社のユーザー（エンドユーザー）が eformsign サービスサイトを介さず自然に eformsign の電子文書を利用することができます。
例えば、ブログで特定の YouTube 動画を表示したいとき、ブログに YouTube 動画を直接挿入することで、その動画をブログ内ですぐ再生できるようにする方式と類似しています。


-------------
始めてみよう
-------------

eformsign embedding 機能を利用するためには、会社 ID とテンプレート ID が必要です。

会社 ID の確認
========================

**会社管理 > 会社情報 > 基本情報** で会社 ID を確認することができます。


メインメニュー
-------------------------

.. image:: resources/Dashboard_menu_icon.png
    :alt: メニュー
    :width: 700px



「会社情報」メニュー
--------------------------------

.. image:: resources/Dashboard_sidemenu_companyinfo.png
    :alt: メニュー > 会社情報
    :width: 700px



会社情報 > 基本情報
-------------------------

.. image:: resources/companyinfo_companyid.png
    :alt: 会社情報 > 基本情報
    :width: 700px



テンプレート ID の確認
===========================

**「テンプレート管理」** メニューに移動し、使用したいテンプレートの設定アイコンをクリックすると、そのテンプレートの URL から form_id を確認することができます。 


「テンプレート管理」メニュー
-----------------------------------

.. image:: resources/sidemenu_managetemplate.png
    :alt: メニュー > テンプレート管理
    :width: 700px



テンプレート管理画面
---------------------------------

.. image:: resources/managetemplate.png
    :alt: テンプレート管理画面
    :width: 700px



テンプレート ID の位置
-----------------------

.. image:: resources/templateURL_templateID.png
    :alt: テンプレート ID の位置
    :width: 700px





---------------
インストール
---------------

eformsign の機能を利用したい Web ページに次のスクリプトを追加します。

.. code-block:: javascript

   //jquery
   <script src="https://www.eformsign.com/plugins/jquery/jquery.min.js"/>
   //eformsign embedded script
   <script src="https://www.eformsign.com/lib/js/efs_embedded_v2.js"/>
   //eformsign redirect script
   <script src="https://www.eformsign.com/lib/js/efs_redirect_v2.js"/>


.. note::

   eformsign の機能を組み込みたいページに上記のスクリプトを追加すると、eformsign のオブジェクトをグローバル変数として使用することができます。


------------------------------------------
eformsign のオブジェクトについて
------------------------------------------

eformsign のオブジェクトは、 embedding と redirect の2つのタイプで構成されています。


+----------+-----------------------+--------------------------------------+
| Type     | Name                  | 説明                                 |
+==========+=======================+======================================+
| embedding| eformsign.document    | eformsignを組み込み、文書を作成  　  |
|          | (document_option,     | できる関数                          |
|          | iframe_id,            |                                      |
|          | success_callback,     | callbackパラメーターはオプション     |
|          | error_callback)       |                                      |
|          |                       | -  document_option, iframe_id: 必須  |
|          |                       |                                      |
|          |                       | -  success_callback: オプション      |
|          |                       |                                      |
|          |                       | -  error_callback: オプション        |
+----------+-----------------------+--------------------------------------+
| redirect | eformsign.document    | eformsignへのページ転換方式で        |
|          | (document_option)     | 文書を作成できるようにする関数       |
|          |                       |                                      |
|          |                       | -  document_option : 必須            |
+----------+-----------------------+--------------------------------------+



.. note::

   redirect 方式は、今後対応する予定です。 


.. code-block:: javascript

     var eformsign = new EformSign();
     
     var document_option = {
       "company" : {
          "id" : '', // company id 入力
          "country_code" : "", // 国コード入力 (ex: kr)
          "user_key": ""  // 顧客システムの固有なキー（顧客システムにログインしたユーザーの unique key）- option
       },
       "user" : {
            "type" : "01" ,
            "access_token" : "", // access Tokenを入力。openAPI accessTokenを参照
            "refresh_token" : "", // refresh Tokenを入力。openAPI accessTokenを参照
            "external_token" : "", // 外部処理の場合、external Tokenを入力。openAPI accessTokenを参照
            "external_user_info" : {
               "name" : "" // 外部処理の場合、外部受信者の名前を入力
            }
        },
        "mode" : {
            "type" : "02",
            "template_id" : "", // template idを入力
            "document_id" : ""  // document_idを入力
        },
        "prefill" : {
            "document_name": "", // 文書のタイトルを入力
            "fields": [ {
                "id" ; "顧客名",
                "value" : "田中太郎",
                "enabled" : true,
                "required" : true 
            }]
        },
        "return_fields" : ['顧客名']
     };
     
     //callback option
     var success_callback = function(response){ 
        console.log(response.code); 
        if( response.code == "-1"){
            //文書の作成に成功
            console.log(response.document_id);
            // return_fieldsに返したデータを照会することができる。fields とは、フォームを作成するときに作った入力コンポーネントのidを意味する。
            console.log(response.field_values["company_name"]);
            console.log(response.field_values["position"]);
        }
     };
      
     var error_callback = function(response){
        console.log(response.code); 
        //文書の作成に失敗
        alert(response.message);
         
     };
     
     eformsign.document(document_option , "eformsign_iframe" , success_callback , error_callback  );


embedding_document 関数
===========================

.. note::

   関数タイプ
   document(document_option, iframe_id, success_callback , error_callback)

eformsign を組み込み、顧客会社のサイト/サービスで文書を作成できるようにする関数です。 eformsign の document 関数を呼び出して使用してください。

大きく document_option と callback の2つのパラメーターを使用することができます。


===================  ===============  ==========  ==========================================================
 Paramter Name       Paramter Type    必須入力      説明 
===================  ===============  ==========  ==========================================================
 document_option      Json             O          組み込み後にeformsignを起動すると、document関連オプションを指定 
 iframe_id            String           O          組み込み後に表示されるiframe id 
 success_callback     function         X          eformsign文書の作成に成功した場合、呼び出されるcallback関数
 error_callback       function         X          eformsign文書の作成に失敗した場合、呼び出されるcallback関数 
===================  ===============  ==========  ==========================================================



.. code-block:: javascript

     var eformsign = new EformSign();
     var document_option = {
        "company": {
            "id": '', // company idを入力
            "country_code": "", // 国コードを入力 (ex: kr)
            "user_key": '' // 顧客システムの固有なKey (顧客システムにログインしたユーザーのunique key) - option
        },
        "user": {
            "type": "01",
            "access_token": "", // access Tokenを入力。openAPIのaccessTokenを参照
            "refresh_token": "", // refresh Tokenを入力。openAPIのaccessTokenを参照
            "external_token": "", // 外部処理の場合、external Tokenを入力。openAPIのaccessTokenを参照
            "external_user_info": {
                "name": "" // 外部処理の場合、外部受信者の名前を入力
            }
        },
        "mode": {
            "type": "02",
            "template_id": "", // template idを入力
            "document_id": "" // document_idを入力
        },
        "prefill": {
            "document_name": "", // 文書のタイトルを入力
            "fields": [{
                "id" : "",
                "顧客名" : "",
                "value": "田中太郎",
                "enabled": true,
                "required": true
            }]
        },
        "return_fields": ['顧客名']
     };
     
     //callback option
     var success_callback = function (response) {
        console.log(response.code);
        if (response.code == "-1") {
            //文書の作成に成功
            console.log(response.document_id);
            // return_fieldsに返したデータを照会することができる。fieldsとは、フォームを作成するときに作った入力コンポーネントのidを意味する。
            console.log(response.field_values["company_name"]);
            console.log(response.field_values["position"]);
        }
     };
     
     
     var error_callback = function (response) {
        console.log(response.code);
        //文書の作成に失敗
        alert(response.message);
     
     };
     
     eformsign.document(document_option, "eformsign_iframe", success_callback, error_callback);


パラメーターの説明：document-option
========================================


document-option では大きく次の5つの項目を設定することができます。 

- 会社情報
- ユーザー情報
- モード
- リターンフィールド
- 自動入力

.. note::

   会社情報とモードは必須入力情報です。 



1. 会社情報（必須）
-------------------------

.. code-block:: javascript

   var document_option = {
     "company" : {
         "id" : 'f9aec832efef4133a1e849efaf8a9aed',  // 会社管理 > 会社情報 の会社idを確認 (必須)
         "country_code" : "kr", // 必須ではないが、クィックオープンのため、指定することを推奨 (会社管理 > 会社情報 で国コードを指定)
         "user_key": "eformsign@forcs.com"
     }
 };


2. ユーザー情報（オプション）
----------------------------------

**メンバーログインによる新規作成**
    - ユーザー情報を指定しない場合に該当し、ユーザー情報を指定しません。	
    - この場合、eformsign ログインページが起動され、ログイン後に文書を作成することができます。


**メンバーのトークンを利用した作成（新規作成および受信した文書を含む）**	
    - 組み込むと、eformsign にログインせず、特定のアカウントの token を利用して文書を作成したり、受信した文書を作成することができます。
    - トークンの発行は、Open API の Access token の発行によって可能です。

.. code-block:: javascript

    var document_option = {
        "user":{
            "type" : "01" , // 01 - internal or  02 - external  (必須)
            "access_token" : "", // access Tokenを入力。openAPI accessTokenを参照
            "refresh_token" : "", // refresh Tokenを入力。openAPI accessTokenを参照
        }
    };


**メンバーではないユーザーが新規文書を作成**  
    - eformsign の会員ではないユーザーが文書を作成できるようにする方式

.. code-block:: javascript

    var document_option = {
        "user":{
            "type" : "02" , // 01 - internal or  02 - external  (必須)
            "external_user_info" : {
                "name" : "" // 外部処理の場合、外部受信者の名前を入力
            }
        }
    };

**メンバーではないユーザーが受信した文書を作成**
    - 組み込みのとき、eformsign の会員ではないユーザーが受信した文書を作成できるようにする方式

.. code-block:: javascript 

    var document_option = {
        "user":{
        "type" : "02" , // 01 - internal or  02 - external  (必須)
        "external_token" : "", // 外部処理の場合、external Tokenを入力。openAPI accessTokenを参照
        "external_user_info" : {
        "name" : "" // 外部処理の場合、外部受信者の名前を入力
            }
        }
    };

.. code-block:: javascript

    var document_option = {
        "user":{
            "type" : "01" , // 01 - internal or  02 - external  (必須)
            "access_token" : "", // access Tokenを入力。openAPI accessTokenを参照
            "refresh_token" : "", // refresh Tokenを入力。openAPI accessTokenを参照
            "external_token" : "", // 外部処理の場合、external Tokenを入力。openAPIのaccessTokenを参照
            "external_user_info" : {
               "name" : "" // 外部処理の場合、外部受信者の名前を入力
            }
        }
    };


3. モード（必須）
---------------------

**テンプレートを利用した新規作成** 
    - テンプレートを利用して文書を新規作成します。

.. code-block:: javascript

    var document_option = {
        "mode" : {
        "type" : "01" ,  // 01：文書の作成、02：文書の処理、03：プレビュー
        "template_id" : "" // template idを入力
        }
    }

**受信した文書に追記** 
    - 受信した文書に追記します。	

.. code-block:: javascript

    var document_option = {
        "mode" : {
        "type" : "02" ,  // 01：文書の作成、02：文書の処理、03：プレビュー
        "template_id" : "", // template idを入力
        "document_id" : ""  // document_idを入力
        }
    }

**特定の文書をプレビュー**
    - 作成した文書のプレビューを確認します。

.. code-block:: javascript

    var document_option = {
        "mode" : {
        "type" : "03" ,  // 01：文書の作成、02：文書の処理、03：プレビュー
        "template_id" : "", // template idを入力
        "document_id" : ""  // document_idを入力
        }
    }

.. code-block:: javascript

    var document_option = {
      "mode" : {
        "type" : "01" ,  //01：文書の作成、02：文書の処理、03：プレビュー
        "template_id" : "", // template idを入力
        "document_id" : ""  // document_idを入力
      }
    }


4. リターンフィールド（オプション）
-----------------------------------------

文書を作成または修正した後、ユーザーが作成したフィールドの内容のうち callback 関数でリターンされる項目を指定します。
    
.. note::

   指定しない場合、基本フィールドのみ提供します。詳しい内容は callBack パラメーターをご覧ください。

.. code-block:: javascript

    var document_option = {
       "return_fields" : ['顧客名']
    }

5. 自動入力（文書を作成するときに自動入力されるよう設定するときに利用）
--------------------------------------------------------------------------

**文書のタイトル**
    - document_name に作成したい文書のタイトルを指定します。

.. code-block:: javascript

    var document_option = {
        "prefill" : {
            "document_name": "休暇届"
        }
    }

**文書内のフィールド設定値の入力** 
    - フォームを作成する時に指定した入力コンポーネントの ID を基準に、フィールドの初期値、活性化、必須入力の設定を指定します。

  
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
            "id" ; "顧客名",
            "value" : "田中太郎",
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
                    "id": "顧客名",
                    "value": "田中太郎",
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
 success_callback    function         X           eformsign文書の作成に成功した場合、呼び出されるcallback関数 
 error_callback      function         X           eformsign文書の作成に失敗した場合、呼び出されるcallback関数
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
| code     | string | 送信に失敗した場合、結果 | -1 の場合、正常　　  |
|          |        | のエラーコードを返す　　 |                      |
+----------+--------+--------------------------+----------------------+
| document | string | 送信に成功した場合、作成 | ex)                  |
| _id      |        | した文書の document_idを | 910b8a965f9          |
|          |        | 返す　　                 | 402b82152f48c6da5a5c |
+----------+--------+--------------------------+----------------------+
| field    | object | document_optionに定義した| ex).                 |
| _values  |        | return_fields コラムに   | field_values["name"] |
|          |        | ユーザーが入力した値を   | // john              |
|          |        | インポートできる         |                      |
+----------+--------+--------------------------+----------------------+
| message  | string | 送信に失敗した場合、   | Nullの場合、正常　   |
|          |        | エラーメッセージを返す　    |                      |
+----------+--------+--------------------------+----------------------+
| title    | string | 送信に成功した場合、作成 | ex) 契約書           |
|          |        | した文書のタイトルを返す　　 |                      |
+----------+--------+--------------------------+----------------------+
