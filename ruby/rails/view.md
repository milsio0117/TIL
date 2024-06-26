## 部分テンプレートを作る
`<%= render partial: "部分テンプレートファイル名" %>`

* テンプレファイルは _ をファイル名の先頭につける
* localsというオプションを付けると部分テンプレート内でその変数を使えるようになる
```ruby
         <%= render partial: "sample", locals: { post: "hello!" } %>
```
post ="hello!" という変数が_sample.html.erbテンプレート内で使える状態
<br><br><br>

## 現在ログインしているユーザーをeachメソッドの処理から取り除く
`モデル名.where.not("条件")`
```ruby
         <% User.where.not(id: current_user.id).each do |user| %>
```

ログインしているユーザー以外のすべてのレコードを取得（all -current_user）
<br><br><br>


## form_with
データベースに保存しない場合
```ruby
         <%= form_with url: "パス" do |form| %>
            <!--内容 -->
         <% end %>
```
<br>

データベースに保存する場合
```ruby
         <%= form_with model: モデルクラスのインスタンス do |form| %>  #モデルクラスのインスタンス:コントローラーで定義
             <!--内容 -->
         <% end %>
```
<br>

例：投稿フォームの作成
```ruby
         <%= form_with model: @user do |form| %>
           <%= form.text_field :name %>  #form.htmlタグ名  :カラム名
           <%= form.submit %>
         <% end %>
```
<br><br><br>

## サインインの有無で条件分岐
`if user_signed_in?`  
例

```ruby
         <% if user_signed_in? %>
           <div class="user_nav grid-6">
              <%= link_to "ログアウト", destroy_user_session_path, data: { turbo_method: :delete } %>
              <%= link_to "投稿する", new_tweet_path, class: "post" %>
           </div>
         <% else %>
           <div class="grid-6">
              <%= link_to "ログイン", new_user_session_path, class: "post" %>
              <%= link_to "新規登録", new_user_registration_path, class: "post" %>
           </div>
         <% end %>
```
<br><br><br>

## 画像の表示方法
`<%= image_tag '画像へのパス' %>`  
例
```ruby
         <%= image_tag 'arrow_top.png' %>
```
<br><br><br>

## 画像のリンク
`<%= link_to image_tag('test.jpg'), 'パス' %>`  
スタイルもつけられる
```ruby
      <%= link_to image_tag('test.jpg', class: "contents"), 'パス' %>
```
<br><br><br>

## collection,member
  これは同じ処理
```ruby
    <% @hoges.each do |hoge|%>
      <%= render partial: 'hoge', locals: {hoge: hoge} %>
    <% end %>
```

```ruby
    <%= render partial: 'hoge', collection: @hoges %>
```
<br><br><br>

## エラー文を表示する
`pluralize`: エラーの数が1以外（0や2以上）なら"error"に適切な複数形の"エラー"という単語をつけるメソッド  
@model ←なんらかのmodelインスタンス  
```ruby
         <% if @model.errors.any? %>
           <div id="error_explanation">
             <h2><%= pluralize(@model.errors.count, "error") %> prohibited this model from being saved:</h2>
         
             <ul>
             <% @model.errors.full_messages.each do |message| %>
               <li><%= message %></li>
             <% end %>
             </ul>
           </div>
         <% end %>
```
<br><br><br>

## インスタンスが空かどうか判断
<% if @items.empty? %>  
→ 空の時の表示
<br><br><br>


## ドロップダウンリストを作成する
```ruby
collection_select(object, method, collection, value_method, text_method, options = {}, html_options = {})
```

```ruby
例：疑似モデル

         class PostagePayer < ActiveHash::Base
           self.data = [
             { id: 1, name: '---' },
             { id: 2, name: '着払い(購入者負担)' },
             { id: 3, name: '送料込み(出品者負担)' }
           ]
         
           include ActiveHash::Associations
           has_many :items
         end
の例
<%= f.collection_select(:postage_payer_id, PostagePayer.all, :id, :name, {}, {class:"select-box", id:"item-postage_payer"}) %>

```

object: フォームに関連づけられているオブジェクト名(formの場合はすでに決まっているので不要)  

method: ユーザが選択肢を選んだときにオブジェクト内で設定するべき属性の名前(≒カラム名)  

collection: ユーザが選択できる選択肢を提供するオブジェクトの配列  

value_method: 選択の値（通常はid）を提供するために各オブジェクトに送るメソッド名  

text_method: 選択の表示テキストを提供するために各オブジェクトに送るメソッド名  

optionsとhtml_options: 必須パラメータではないがカスタマイズを追加する時  


<br>
その他のoption例  

```ruby
{ selected: @item.shipping_day_id }

↑選択リストが表示されたときに@itemオブジェクトのshipping_day_idがデフォルトで選択されている
```

```ruby
{ prompt: 'Please select a shipping day' }

↑選択したリストが表示された初めに「Please select a shipping day」が表示される
```

```ruby
{ include_blank: '---' }

↑ドロップダウンリストの先頭に空の選択肢を追加する。この場合は「---」が空の選択肢となる。
```
<br><br><br>

## 日付を書式指定して文字列に変換
### lメソッドを使う

config/apprication.rb
```ruby
         class Application < Rails::Application
             # Initialize configuration defaults for originally generated Rails version.
             config.load_defaults 7.0
         
             config.autoload_paths << Rails.root.join('app/services')
         
             config.time_zone = "Asia/Tokyo"
             # デフォルトのロケールを日本（ja）に設定
             config.i18n.default_locale = :ja # 追記
```

config/locales/ja.yml
```ruby
         ja:
           time:
             formats:
               default: "%Y/%m/%d %H:%M:%S"　# デフォルト表記
               short: "%m/%d" #日付だけ
               time: "%H:%M" #時刻だけ

```

view
```ruby
         <td><%= l memo.created_at %></td> # → 投稿日時：2024/06/10 12:09:03
```
デフォルト表記

```ruby
         <td><%= l memo.created_at, format: :short %></td>  # → 投稿日時：06/10
```
ja.ymlで設定しておけばビューごとにフォーマットも選べる  
<br>
※Viewではlと書くだけでOKだが、lメソッドがヘルパーメソッドとして提供されていない場所（Modelなど）ではlだけではNoMethodErrorとなる。  
その場合は、lの代わりにI18n.lと書けば日時をフォーマットすることができる。
<br><br>
### strftimeを使う
```ruby
         <%= @message.created_at.strftime("%Y年%m月%d日%H時%M分") %>
```

長いのでmodelでメソッドを規定
```ruby
         def set_date
           created_at.strftime("%Y年%m月%d日%H時%M分")
         end
```

viewがすっきりする
```ruby
         <%= @message.set_date %>
```
<br><br><br>

## どのアクションから送られてきたかの分岐
1.  2つ以上の異なるアクションから1つのビューに送信され、ビューでその表示を分岐したい場合  
 → 遷移するリンクに元のアクション名（indexまたはhistory）を示すパラメータ（例えばorigin）を追加  
 例）indexとhistoryからshowに行き、そこで表示の分岐をする
```ruby
          # history.html.erb //historyから来たことを明記
         <%= link_to "#{stock.ticker} - #{stock.name}", stock_path(stock, origin: 'history') %>

         # index.html.erb //indexから来たことを明記
         <%= link_to "#{stock.ticker} - #{stock.name}", stock_path(stock, origin: 'index') %>
```

show.html.erb
```ruby
         <% if params[:origin] == 'history' %>
           # historyから来た場合の処理
         <% else %>
           # その他から来た場合の処理
         <% end %>
```
※他のアクションから遷移した場合の挙動は別途考慮する必要がある  
controller
```ruby
         def show
           if params[:origin] == 'history'
             @stocks = Stock.archived
           else
             @stocks = Stock.active
           end
         end
```
> [!NOTE]
> origin: 'history'という表記は、Rubyのハッシュというデータ型を利用したもの。  
> この場合はoriginがキー、'history'がその値。文字の規定はない。  
> この記述がpathに渡されると、生成されるURLの末尾に?origin=historyというクエリパラメータが付加される。  
> それにより、次にアクセスするページ側でparams[:origin]の形でこの値を取得することが可能になる。  
<br>

2. 元のアクション名をセッションに保持する  
controller
```ruby
         def index
           session[:origin_action] = 'index'
         end
         
         # historyアクション
         def history
           session[:origin_action] = 'history'
         end
```

view
```ruby
         <% if session[:origin_action] == 'history' %>
           # sessionがhistoryだった時の処理
         <% else %>
           # その他の時の処理
         <% end %>
```
>[!note]
>セッションを利用してorigin_actionというキーに'history'という値を保存している。  
>セッションはブラウザが閉じられるまでの間、サーバー側で状態を保持するための仕組み。  
>ここでは、どのアクションがユーザーによって最後に呼び出されたかを追跡する。  
