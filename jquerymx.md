@page jquerymx jQueryMX
@parent index 0
@description jQuery Model View Controller and extensions.

jQueryMXはjQueryに不足している、
大規模なjQueryアプリケーションの実装・構築に必要な機能を提供するライブラリです。

jQueryMXの各部品はあなたのアプリを軽量に保つため、それぞれ独立して使えます。
$.Model、$.View、そして$.Controllerをあわせても、ミニファイしてgzip化すれば
わずか7kbしかありません。
これらには$.Stringや$.Class、destroyedイベントなどの依存ライブラリも含まれています。

依存管理の仕組みである [steal] を使っているのであれば、
下記のように書けば必要とするプラグインだけを利用できます。

    steal('jquery/controller', function(){
      $.Controller('Tabs');
    })
    
または、[http://javascriptmvc.com/builder.html download builder]を用いて
必要なファイルだけを選択してダウンロードすることも出来ます。

jQueryMXは次の４つの主要部分で成り立っています。

  - DOM Helpers
  - Language Helpers
  - Special events
  - Model, View, Controller and Class ([mvc the walkthrough]参照 )

 
## DOM Helpers

[dom DOM helpers] は、DOM操作のためにjQueryを拡張します。
例えば, HTMLエレメントの横・縦幅を指定するには以下の書き方ができます。

    $('#foo').outerWidth(500);
    
<!--[dom DOM helpers] extend jQuery with extra functionality for 
manipulating the DOM. For example, [dimensions] lets you set the 
outer width and height of elements like:

    $('#foo').outerWidth(500);
    
-->
The following are the other dom plugins:

  - [jQuery.cookie Cookie] - Set and get cookie values.
  - [jQuery.fixture Fixture] - Simulate Ajax responses.
  - [jQuery.fn.closest Closest] - Use the open child selector in event delegation.
  - [jQuery.fn.compare Compare] - Compare the location of two elements rapidly.
  - [jQuery.fn.curStyles CurStyles] - Get multiple css properties quickly.
  - [jQuery.fn.formParams FormParams] - Serializes a form into a JSON-like object.
  - [jQuery.fn.selection Selection] - Gets or sets the current text selection.
  - [jQuery.fn.within Within] - Returns elements that have a point within their boundaries.
  - [jQuery.Range Range] - Text range utilities.
  - [jQuery.route] - Routes for history-enabled ajax apps.


## Special Events

jQueryMX comes packed with jQuery [specialevents special events] and event helpers.

  - [jQuery.Drag Drag] - Delegatable drag events.
  - [jQuery.Drop Drop] - Delegatable drop events.
  - [jQuery.Hover Hover] - Delegatable hover events.
  - [jQuery.event.special.destroyed Destroyed] - Know when an element is removed from the page.
  - [jQuery.event.special.resize Resize] - Listen to resize events on any element.
  - [jQuery.event.swipe Swipe] - Delegatable swipe events.
  - [jQuery.Event.prototype.key Key] - Get the character from a key event.
  - [jQuery.event.special.default Default] - Provide default behaviors for events.
  - [jquery.event.pause Pause-Resume] - Pause and resume event propagation.


## Language Helpers

Language helpers make it easy to perform various functions on
JavaScript data.  

  - [jQuery.Object Object] - compare objects and sets
  - [jQuery.Observe Observe] - Listen to changes in JS Objects and Arrays
  - [jQuery.String String] - String helpers
  - [jQuery.toJSON toJSON] - create and convert JSON strings
  - [jQuery.Vector Vector] - vector math

## $.Class

[jQuery.Class $.Class]はシンプルなプロトタイプ継承を提供します。
[jQuery.Controller $.Controller]と[jQuery.Model $.Model]でも利用されています。

    // 怪獣クラスの作成
	$.Class("Monster",
	// static methods 
	{
	
	  // 怪獣のリスト
	  monsters : []
	},
	// prototype methods
	{
	
	  // インスタンス生成時に呼ばれる
	  init : function(name){
	  
	    // name値の保持
	    this.name = name;
	    
	    // 怪獣リストに自身を追加
	    this.Class.monsters.push(this);
	  },
	  
	  // メソッド定義
	  speak : function(){
	    alert(this.name + " says hello.");
	  }
	});
	
	// 怪獣インスタンス生成
    var hydra = new Monster("hydra");	

    // speakメソッドの呼び出し
	hydra.speak();

## $.Model

[jQuery.Model $.Model]はサービス・データレイヤをカプセル化します。
下記ではTaskクラスを定義し、
JSON RESTサービスへの接続、および削除可能か問い合わせるヘルパーメソッドを定義しています。

    $.Model("Task",{
      findAll : "GET /tasks.json",
      findOne : "GET /tasks/{id}.json",
      create  : "POST /tasks.json",
      update  : "PUT /tasks/{id}.json",
      destroy : "DELETE /tasks/{id}.json"
    },{
      canDestroy : function(){
        return this.acl.indexOf('w') > -1
      }
    });

'/tasks.json'は次のようなJSON配列を返すものとしておきます。

    [{
      "id"       : 1,
      "name"     : "take out trash",
      "acl"      : "rw",
      "createdAt": 1303000731164 // April 16 2011
    },
    {
      "id"       : 2,
      "name"     : "do the dishes",
      "acl"      : "r" ,
      "createdAt": 1303087131164 // April 17 2011
    }]

下記の例はサーバからすべてのTaskを取得し、
削除可能であれば削除を行っています。
※canDestroyメソッドはaclプロパティに'w'が含まれていればtrueですね！

    Task.findAll({}, function(tasks){
      for(var i =0; i < tasks.length; i++){
       
        var task = tasks[i];
        
        if( task.canDestroy() ){
          task.destroy();
        }
      }
    });

Modelはその他にも次のような有用な特徴を持っています。

  - イベントのリスニング [jquery.model.events events]

		// taskインスタンスのname変更イベントをリッスン
		task.bind("name", function(ev, newName){
			alert('task name = '+newName);
		});

		//taskインスタンスのname変更
    	task.attr('name', "laundry");

    	//Task生成時のイベントをリッスン
    	Task.bind("created", function(ev, newTask){
       		// newTaskを使ってhtml生成したり出来るよ
    	});


  - 生データを使いやすいオブジェクトへ変換 [jquery.model.typeconversion Converting]

		$.Model('Task', {
		  convert  : {
		    'date' : function(raw){
		      return new Date(raw)
		    }
		  },
		  attributes : {
		    'createdAt' : 'date' 
		  }
		},{});

		var task = new Task({ createdAt : 1303087131164});

		// createdAtはDateオブジェクトとして使える
		task.createdAt.getFullYear() // -> 2011

  - [jQuery.Model.List lists]コレクションクラスで使えるメソッド+ユーティリティ

		// Taskモデルのコレクションクラスを定義
		$.Model.List('Task.List',{

		  // ヘルパーメソッドの定義
		  canDestroyAll : function(){
    
		    return this.grep(function(task){
		      return task.canDestroy();
		    }).length === this.length
		  }
		});

		Task.findAll({}, function(tasks){

		  //引数のtasksはTask.Listインスタンス
		  tasks.canDestroyAll() //-> boolean
		})

  - [http://api.jquery.com/category/deferred-object/ Deferreds]

		// ２つのリクエストを生成し、両方が完了した時点で処理を行う
		
		$.when( Task.findAll(), People.findAll() )
		  .done(function(tasks, people){

		  // do 何らかの処理 cool!だね！
		})


<!--<ul>
  <li><p>Listening to [jquery.model.events events].</p>
@codestart
// listen to name changes on a task
task.bind("name", function(ev, newName){
   alert('task name = '+newName);
});

//change the task's name
task.attr('name', "laundry");

//listen for Tasks being created:
Task.bind("created", function(ev, newTask){
   // create newTask's html and add it to the page
});
@codeend
</li>
<li><p>[jquery.model.typeconversion Converting] raw data into more useful objects.</p>
@codestart
$.Model('Task', {
  convert  : {
    'date' : function(raw){
      return new Date(raw)
    }
  },
  attributes : {
    'createdAt' : 'date' 
  }
},{});

var task = new Task({ createdAt : 1303087131164});

// createdAt is now a date.
task.createdAt.getFullYear() // -> 2011
@codeend
</li>
<li><p>Methods and utilities on [jQuery.Model.List lists] of instances.</p>
@codestart
// define a task list
$.Model.List('Task.List',{

  // add a helper method to a collection of tasks
  canDestroyAll : function(){
    
    return this.grep(function(task){
      return task.canDestroy();
    }).length === this.length
  }
});

Task.findAll({}, function(tasks){

  //tasks is a Task.List
  tasks.canDestroyAll() //-> boolean
})
@codeend
</li>
<li><p>[http://api.jquery.com/category/deferred-object/ Deferreds]</p>
@codestart
// make 2 requests, and do something when they are 
// both complete

$.when( Task.findAll(), People.findAll() )
  .done(function(tasks, people){

  // do something cool!
})
@codeend
</li>
</ul>
-->    
## $.View

[jQuery.View $.View] is a template framework.  It allows 
you to use different template engines in the same way.  

The following requests tasks from the model, then
loads a template at <code>"task/views/tasks.ejs"</code>, 
renders it with tasks, and 
inserts the result in the <code>#tasks</code> element.

    Task.findAll( {}, function(tasks){
      
      $('#tasks').html( 'task/views/tasks.ejs', tasks );
    });

<code>tasks.ejs</code> might look like:

    <% $.each(this, function(task){  %>
      <li><%= task.name %></li>
    <% }) %>

$.View understands [http://api.jquery.com/category/deferred-object/ deferreds] so the following does the exact same thing!

     $('#tasks').html( 'task/views/tasks.ejs', Task.findAll() );

Any template engine can be used with $.View.  JavaScriptMVC comes with:

  - [jQuery.EJS]
  - [Jaml]
  - [Micro]
  - [jQuery.tmpl]

## $.Controller

[jQuery.Controller $.Controller] is a jQuery widget factory. The 
following creates a <code>$.fn.list</code> [jquery.controller.plugin plugin] that writes 
a message into an element:

    $.Controller( "List", {
      init: function( ) {
        this.element.text( this.options.message );
      }
    });

	// create the list
	$('#list').list({message: "Hello World"});

$.Controller lets you define [jQuery.Controller.static.defaults default options]:

    $.Controller( "List", {
      defaults: {
        message : "I am list"
      }
    },{
      init: function( ) {
        this.element.text( this.options.message );
      }
    });

    // create's a list that writes "I am list"
	$('#list').list();

Controller's best feature is that it organizes your event handlers, and 
makes [jquery.controller.listening binding and unbinding] event 
handlers extremely easy. The following listens for clicks on an
<code>LI</codE> elements and alerts the element's text:

    $.Controller( "TaskList", {
      init: function(){
        // uses a view to render tasks
        this.element.html( "tasks.ejs", Task.findAll() );
      },
      "li click": function(el){
        alert( el.text() );
      }
    });

Controller makes it easy to parameterize event binding.  The following 
listens for tasks being created and inserts them into the list:

    $.Controller( "TaskList", {
      init: function( ) {
        // uses a view to render tasks
        this.element.html("tasks.ejs", Task.findAll());
      },
      "{Task} created": function( Task, ev, newTask ) {
        this.element.append( "tasks.ejs", [newTask] );
      }
    });

Finally, this makes it very easy to create widgets that work with any model:

    $.Controller( "List", {
      init: function(){
        // uses a view to render tasks
        this.element.html( this.options.view, 
                           this.options.model.findAll( ));
      },
      "{model} created": function( Model, ev, instance ){
        this.element.append( this.options.view, [instance] );
      }
    });
    
    $("#tasks").list({ model: Task, view: 'tasks.ejs' });
    $("#people").list({model: Person, view: 'people.ejs' });




  
  