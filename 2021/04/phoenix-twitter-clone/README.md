# Build a real-time Twitter clone

## Tutorials

Build a real-time Twitter clone in 15 minutes with LiveView and Phoenix 1.5
[[Video]](https://www.phoenixframework.org/blog/build-a-real-time-twitter-clone-in-15-minutes-with-live-view-and-phoenix-1-5), [[Code]](https://github.com/dersnek/chirp)

### Phoenix v1.5
Phoenix LiveDashboard
* Real-time performance monitoring and debugging tool (focus on production data)
* Streaming request logger

Phoenix LiveView generators
* `mix phx.new [project_name] --live` : include everything you need to get up and running developing with LiveView
*  `phx.gen.live` generator for boostrapping CRUD LiveView context/interfaces similar to `phx.gen.html/phx.gen.json`. 

### Hands-On

Create project with `mix phx.new chirp --live`.

Create LiveView context with `mix phx.gen.live Timeline Post posts username body likes_count:integer reposts_count:integer`
3. Copy router content in console and paste in `router.ex`.

```elixir
  live "/posts", PostLive.Index, :index
  live "/posts/new", PostLive.Index, :new
  live "/posts/:id/edit", PostLive.Index, :edit

  live "/posts/:id", PostLive.Show, :show
  live "/posts/:id/show/edit", PostLive.Show, :edit
```

With ecto you get some new mix tasks to your project to deal with the database. They might help you:

* `mix ecto.create` - create the database which is used by your repository as backend
* `mix ecto.migrate` - runs the pending migrations for your repository
* `mix ecto.drop` - drops the database

Go to `chirp_web/live/post_live/form_component.html.leex` and change form element.

```elixir
<h2><%= @title %></h2>

<%= f = form_for @changeset, "#",
  id: "post-form",
  phx_target: @myself,
  phx_change: "validate",
  phx_submit: "save" %>

  <%= textarea f, :body %>
  <%= error_tag f, :body %>

  <%= submit "Save", phx_disable_with: "Saving..." %>
</form>
```

Set data schema in `lib/chirp/timeline/post.ex`

```elixir
defmodule Chirp.Timeline.Post do
  # ....

  schema "posts" do
    field :body, :string
    field :likes_count, :integer, default: 0
    field :reposts_count, :integer, default: 0
    field :username, :string, default: "anonymous"

    timestamps()
  end

  # ...
end
```

Go to `chirp_web/live/post_live/post_component.ex` and change HTML template.

```elixir
<div id="posts">
  <%= for post <- @posts do %>
    <%= live_component @socket, ChirpWeb.PostLive.PostComponent, id: post.id, post: post %>
  <% end %>
</div>
```

Create `post_component.ex` and make `ChirpWeb.PostLive.PostComponent `.

```elixir
defmodule ChirpWeb.PostLive.PostComponent do
  use ChirpWeb, :live_component

  def render(assigns) do
    ~L"""
    <div id="post-<%= @post.id %>" class="post">
      <div class="row">
        <div class="column column-10">
          <div class="post-avatar"></div>
        </div>
        <div class="column column-90 post-body">
          <b>@<%= @post.username %></b>
          <br/>
          <%= @post.body %>
        </div>
      </div>

      <div class="row">
        <div class="column post-button-column">
          <a href="#">
          <i class="far fa-heart"></i> <%= @post.likes_count %>
        </div>
        <div class="column post-button-column">
          <a href="#">
          <i class="far fa-hand-peace"></i> <%= @post.reposts_count %>
        </div>
        <div class="column post-button-column">
          <%= live_patch to: Routes.post_index_path(@socket, :edit, @post.id) do %>
            <i class="far fa-edit"></i>
          <% end %>
          <%= link to: "#", phx_click: "delete", phx_value_id: @post.id do %>
            <i class="far fa-trash-alt"></i>
          <% end %>
        </div>
      </div>
    </div>
    """
  end
end
```

Install `fontawesome` and `file-loader` dependencies.

Go to `assets` and install dependencies with `yarn add @fortawesome/fontawesome-free file-loader`. Check dependencies in `package.json`.

Create `_custom.scss` file under `assets/css` and add some styling.

```css
/* Fontawesome 5 config */

$fa-font-path: "~@fortawesome/fontawesome-free/webfonts";

.posts {
  align-items: center;
}

.post {
  margin: 10px 100px;
  padding: 10px 20px;
  box-shadow: 0px 0px 2px 2px rgba(0,0,0,0.1);
}

.post-avatar {
  background: gray;
  height: 40px;
  width: 40px;
  border-radius: 50%;
}

.post-button-column {
  text-align: center;
  margin: 15px 0px 0px 0px;
}
```

Import `_custom.scss` file and fontawesome-free libs in `app.scss`.

```scss
@import "custom";
@import "~@fortawesome/fontawesome-free/scss/fontawesome";
@import "~@fortawesome/fontawesome-free/scss/regular";
@import "~@fortawesome/fontawesome-free/scss/solid";
@import "~@fortawesome/fontawesome-free/scss/brands";
```

Go to `webpack.config.js` add file loader config.

```js
        // Load fonts
        {
          test: /\.(woff(2)?|ttf|eot|svg)(\?v=\d+\.\d+\.\d+)?$/,
          use: [{
            loader: 'file-loader',
            options: {
              name: '[name].[ext]',
              outputPath: '../fonts'
            },
          }],
        },
```

Next we need to broadcast events if the new post is created/changed.

Go to `lib/chirp/timeline.ex` and update `create_post` and `update_post` with `broadcast(:post_created)`.

```elixir
  def create_post(attrs \\ %{}) do
    %Post{} |> Post.changeset(attrs) |> Repo.insert() |> broadcast(:post_created)
  end
```

```elixir
def update_post(%Post{} = post, attrs) do
  post
  |> Post.changeset(attrs)
  |> Repo.update()
  |> broadcast(:post_created)
end
```

Create `subscribe` and `broadcast` function under the end of module.

```elixir
def subscribe do
  Phoenix.PubSub.subscribe(Chirp.PubSub, "posts")
end

defp broadcast({:error, _reason} = error, _event), do: error

defp broadcast({:ok, post}, event) do
  Phoenix.PubSub.broadcast(Chirp.PubSub, "posts", {event, post})
end
```

Open `live/post_live/index.ex` and update `mount` callback to subscribe events if socket is connected.

```elixir
@impl true
  def mount(_params, _session, socket) do
    if connected?(socket), do: Timeline.subscribe()
    {:ok, assign(socket, :posts, list_posts())}
  end
```

In the same file, update socket and posts, add new post in existing posts.

```elixir
  def handle_info({:post_created, post}, socket) do
    {:noreply, update(socket, :posts, fn posts -> [post | posts] end)}
  end
```

Open two browsers and create a post. Check the other browser update post list instantly.

Now, it needs to sort post list.

Go to `lib/chirp/timeline.ex` and modify `list_posts` to sort list in descending order by `id`.

```elixir
  def list_posts do
    Repo.all(from p in Post, order_by: [desc: p.id])
  end
```

To receive a post update, add `handle_info`.

```elixir
  def handle_info({:post_updated, post}, socket) do
    {:noreply, update(socket, :posts, fn posts -> [post | posts] end)}
  end
```

There's no reason to hold all posts and memory on the server and it renders all of them. Phoenix has a collection optimization which allows us to assign some of them are temporary.

```elixir
  def mount(_params, _session, socket) do
    if connected?(socket), do: Timeline.subscribe()
    {:ok, assign(socket, :posts, list_posts()), temporary_assigns: [posts: []]}
  end
```

It is to mark which assigns are temporary and what values they should be reset to on mount.

See: https://hexdocs.pm/phoenix_live_view/dom-patching.html

Go to `live/post_live/index.html.leex` and tag a post container with `phx-update`.

```elixir
<div id="posts" phx-update="prepend" >
  <%= for post <- @posts do %>
    <%= live_component @socket, ChirpWeb.PostLive.PostComponent, id: post.id, post: post %>
  <% end %>
</div>
```

The following `phx-update` values are supported:

* `replace` - the default operation. Replaces the element with the contents
* `ignore` - ignores updates to the DOM regardless of new content changes
* `append` - append the new DOM contents instead of replacing
* `prepend` - prepend the new DOM contents instead of replacing

To implement like and repost click event, open `post_component.ex` and add `phx-click` and `phx-target` tags in all button elements.

```html
        <div class="column post-button-column">
          <a href="#" phx-click="like" phx-target="<%= @myself %>">
          <i class="far fa-heart"></i> <%= @post.likes_count %>
        </div>
        <div class="column post-button-column">
          <a href="#" phx-click="repost" phx-target="<%= @myself %>">
          <i class="far fa-hand-peace"></i> <%= @post.reposts_count %>
        </div>
```

And add event handlers in the below.

```elixir
  def handle_event("like", _, socket) do
    Chirp.Timeline.inc_likes(socket.assigns.post)
    {:noreply, socket}
  end

  def handle_event("repost", _, socket) do
    Chirp.Timeline.inc_reposts(socket.assigns.post)
    {:noreply, socket}
  end
```

Add `inc_likes` and `inc_reposts` in `timeline.ex`.

```elixir
  def inc_likes(%Post{id: id}) do
    {1, [post]} =
      from(p in Post, where: p.id == ^id, select: p)
      |> Repo.update_all(inc: [likes_count: 1])

    broadcast({:ok, post}, :post_updated)
  end

  def inc_reposts(%Post{id: id}) do
    {1, [post]} =
      from(p in Post, where: p.id == ^id, select: p)
      |> Repo.update_all(inc: [reposts_count: 1])

    broadcast({:ok, post}, :post_updated)
  end
  ```

## More tutorials

[Phoenix LiveView Counter Tutorial](https://github.com/dwyl/phoenix-liveview-counter-tutorial)
