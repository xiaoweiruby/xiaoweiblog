
# Ruby on Rails 的快速学习的体系框架；

# 参考案例体系：
https://ruby-china.github.io/rails-guides/getting_started.html

https://www.techotopia.com/index.php/Ruby_Essentials

https://rails.guide/

Ruby on Rails 编程体系的技能体系的学习的关键所在在于：

# 新手阶段 10% ；

1、Ruby  的后端结构的体系的学习；

2、Rails 的前端框架的体系的学习；

3、Ruby on Rails 的案例体系的训练 ；

# 高级新手阶段 59% ；

4、针对于存在的案例给与项目的复现；

# 胜任者 20% ；精通者阶段 10% ；

5、针对于不存在的案例使用面板完成项目分解，从而打造自己的项目；

# 专家阶段 1% ；

6、针对于别人的项目的问题，快速的给与解决的方法和答疑


# 新建一个专栏

>rails new blog
cd blog
git init
git add .
git commit -m "commit initial"

# 构建一个首页

>bin/rails generate controller Welcome index
atom .
rails s

touch app/views/welcome/index.html.erb
```
<h1>Hello, Rails!</h1>
```
config/routes.rb
```
Rails.application.routes.draw do
  root 'articles#index'
  resources :articles
end

```

# 构建文章列表
 >git checkout -b add_articles
 bin/rails generate controller Articles
 bin/rails generate model Article title:string text:text
 bin/rails db:migrate
 rails s
 git add .
 git commit -m "add articles"

app/controllers/articles_controller.rb
```
class ArticlesController < ApplicationController
  def index
    @articles = Article.all
  end

  def show
    @article = Article.find(params[:id])
  end

  def new
    @article = Article.new
  end

  def edit
    @article = Article.find(params[:id])
  end

  def create
    @article = Article.new(article_params)

    if @article.save
      redirect_to @article
    else
      render 'new'
    end
  end

  def update
    @article = Article.find(params[:id])

    if @article.update(article_params)
      redirect_to @article
    else
      render 'edit'
    end
  end

  def destroy
    @article = Article.find(params[:id])
    @article.destroy

    redirect_to articles_path
  end

  private
    def article_params
      params.require(:article).permit(:title, :text)
    end
end

```
touch app/views/articles/index.html.erb
 ```
 <h1>Hello, Rails!</h1>
 <%= link_to 'My Blog', controller: 'articles' %>

 <h1>Listing Articles</h1>
 <%= link_to 'New article', new_article_path %>
 <table>
   <tr>
     <th>Title</th>
     <th>Text</th>
     <th colspan="3"></th>
   </tr>

   <% @articles.each do |article| %>
     <tr>
       <td><%= article.title %></td>
       <td><%= article.text %></td>
       <td><%= link_to 'Show', article_path(article) %></td>
       <td><%= link_to 'Edit', edit_article_path(article) %></td>
       <td><%= link_to 'Destroy', article_path(article),
               method: :delete,
               data: { confirm: 'Are you sure?' } %></td>
     </tr>
   <% end %>
 </table>

 ```
 app/views/articles/-form.html.erb
 ```
 <%= form_for @article do |f| %>

   <% if @article.errors.any? %>
     <div id="error_explanation">
       <h2>
         <%= pluralize(@article.errors.count, "error") %> prohibited
         this article from being saved:
       </h2>
       <ul>
         <% @article.errors.full_messages.each do |msg| %>
           <li><%= msg %></li>
         <% end %>
       </ul>
     </div>
   <% end %>

   <p>
     <%= f.label :title %><br>
     <%= f.text_field :title %>
   </p>

   <p>
     <%= f.label :text %><br>
     <%= f.text_area :text %>
   </p>

   <p>
     <%= f.submit %>
   </p>

 <% end %>
 ```
 app/views/articles/show.html.erb
 ```
 <p>
   <strong>Title:</strong>
   <%= @article.title %>
 </p>

 <p>
   <strong>Text:</strong>
   <%= @article.text %>
 </p>

 <h2>Comments</h2>
 <%= render @article.comments %>

 <h2>Add a comment:</h2>
 <%= render 'comments/form' %>

 <%= link_to 'Edit', edit_article_path(@article) %> |
 <%= link_to 'Back', articles_path %>

 ```

 app/views/articles/new.html.erb
 ```
 <h1>New article</h1>

 <%= render 'form' %>

 <%= link_to 'Back', articles_path %>

 ```
 app/views/articles/edit.html.erb
 ```
 <h1>Edit article</h1>

 <%= render 'form' %>

 <%= link_to 'Back', articles_path %>

 ```
# 构建评论页面
git checkout -b add_comments
bin/rails generate model Comment commenter:string body:text article:references
bin/rails db:migrate
bin/rails generate controller Comments
rails s

config/routes.rb
```
Rails.application.routes.draw do
  root 'articles#index'
  resources :articles do
  resources :comments
 end
end

```

app/models/article.rb
```
class Article < ApplicationRecord
  has_many :comments, dependent: :destroy
  validates :title, presence: true,
                      length: { minimum: 5 }

                    end

```
app/models/comment.rb
```
class Comment < ApplicationRecord
  belongs_to :article
end

```

app/controllers/comments_controller.rb
```
class CommentsController < ApplicationController
  def create
    @article = Article.find(params[:article_id])
    @comment = @article.comments.create(comment_params)
    redirect_to article_path(@article)
  end

  def destroy
     @article = Article.find(params[:article_id])
     @comment = @article.comments.find(params[:id])
     @comment.destroy
     redirect_to article_path(@article)
   end

  private
    def comment_params
      params.require(:comment).permit(:commenter, :body)
    end
end
```

app/views/comments/-comment.html.erb
```
<p>
  <strong>Commenter:</strong>
  <%= comment.commenter %>
</p>

<p>
  <strong>Comment:</strong>
  <%= comment.body %>
</p>

<p>
  <%= link_to 'Destroy Comment', [comment.article, comment],
               method: :delete,
               data: { confirm: 'Are you sure?' } %>
</p>

```

app/views/comments/-form.html.erb
```
<%= form_for([@article, @article.comments.build]) do |f| %>
  <p>
    <%= f.label :commenter %><br>
    <%= f.text_field :commenter %>
  </p>
  <p>
    <%= f.label :body %><br>
    <%= f.text_area :body %>
  </p>
  <p>
    <%= f.submit %>
  </p>
<% end %>

```

# 上传代码仓库和云端项目部署
git remote add origin https://github.com/shenzhoudance/xiaoweiblog.git
git push --all origin
