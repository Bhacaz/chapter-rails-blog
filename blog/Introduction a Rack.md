---
layout: default
title: Introduction a Rack
parent: Blog
nav_order: 7
---

# Introduction a Rack

> Rack provides a minimal, modular, and adaptable interface for developing web applications in Ruby. By wrapping HTTP requests and responses in the simplest way possible.
> 
> [rack/rack](https://github.com/rack/rack)

**Le plus simple possible!**

1. Un object qui répond a `call` comme un proc.
    ```ruby
    app = -> { }
    ```
2. Qui prend en paramètre un object `env` (un Hash)
    ```ruby
    app = -> (env) { }
    ```
3. Qui retourne un Array avec 3 items
    1. Status (Integer)
    2. Header (Hash)
    3. Body (Array)
    
    ```ruby
    app = -> (env) { [200, { 'Content-Type' => 'text/plain' }, ['Hello World']] }
    ```

Nous avons maintenant une app Rack (comme Rails)

## Server web

Rack a besoin d'un serveur web qui va écouter les requests dans une `loop`. Plusieurs nom connue: Puma, Passenger, Unicorn, WEBrick, Nginx...

1. Web server recoit une requête
2. Le Rack::Handler parse la requête dans un Hash (`env`)
3. Appel l'app en passant `env` en paramètre 

```ruby
app = -> (env) { [200, { 'Content-Type' => 'text/plain' }, ['Hello World']] }

require 'rack/handler/puma'
Rack::Handler::Puma.run(app)
# OR
Rack::Handler::WEBrick.run(app)
```

## `env`

Variable qui contient l'information du call HTTP.

```ruby
{"GATEWAY_INTERFACE"=>"CGI/1.1",
 "PATH_INFO"=>"/",
 "QUERY_STRING"=>"",
 "REMOTE_ADDR"=>"::1",
 "REMOTE_HOST"=>"::1",
 "REQUEST_METHOD"=>"GET",
 "REQUEST_URI"=>"http://localhost:3001/",
 "SCRIPT_NAME"=>"",
 "SERVER_NAME"=>"localhost",
 "SERVER_PORT"=>"3001",
 "SERVER_PROTOCOL"=>"HTTP/1.1",
 "SERVER_SOFTWARE"=>"WEBrick/1.4.2 (Ruby/2.6.6/2020-03-31)",
 "HTTP_HOST"=>"localhost:3001",
 "HTTP_CONNECTION"=>"keep-alive",
 "HTTP_CACHE_CONTROL"=>"max-age=0",
 "HTTP_UPGRADE_INSECURE_REQUESTS"=>"1",
 "HTTP_USER_AGENT"=>
  "Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_5) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36",
 "HTTP_ACCEPT"=>
  "text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.9",
 "HTTP_SEC_FETCH_SITE"=>"cross-site",
 "HTTP_SEC_FETCH_MODE"=>"navigate",
 "HTTP_SEC_FETCH_USER"=>"?1",
 "HTTP_SEC_FETCH_DEST"=>"document",
 "HTTP_ACCEPT_ENCODING"=>"gzip, deflate, br",
 "HTTP_ACCEPT_LANGUAGE"=>"en-US,en;q=0.9",
 "HTTP_COOKIE"=>"_development_sid=606d05c4fb3b84113d276ff27b143cba",
 "rack.version"=>[1, 3],
 "rack.input"=>#<StringIO:0x00007f8fa7a8de80>,
 "rack.errors"=>#<IO:<STDERR>>,
 "rack.multithread"=>true,
 "rack.multiprocess"=>false,
 "rack.run_once"=>false,
 "rack.url_scheme"=>"http",
 "rack.hijack?"=>true,
 "rack.hijack"=>
  #<Proc:0x00007f8fa7a8de08@/Users/bhacaz/.rbenv/versions/2.6.6/lib/ruby/gems/2.6.0/gems/rack-2.2.3/lib/rack/handler/webrick.rb:83 (lambda)>,
 "rack.hijack_io"=>nil,
 "HTTP_VERSION"=>"HTTP/1.1",
 "REQUEST_PATH"=>"/"}
```
Les plus intéressant

```ruby
REQUEST_PATH
QUERY_STRING
REQUEST_METHOD
```

A partir de cela on peut commencer a construire une application, API, framework...

```ruby
app = -> (env) do
  case env['REQUEST_METHOD']
    when 'GET'
      [ 200, { "Content-Type" => "text/plain" }, ["Hello World"] ]
    when 'POST'
      [ 201, { "Content-Type" => "text/plain" }, ["Created"] ]
    else
      [ 404, { "Content-Type" => "text/plain" }, ["Not found"] ]
    end
end
```

Pour l'expérience dans une console `rails`

```ruby
>> PetalMD::Application.call({})
# => ActionController::UnknownHttpMethod (, accepted HTTP methods are OPTIONS, GET, HEAD, POST, PUT, DELETE, ..., and PATCH)
```

## Middleware

1. Un `initialize` qui prend `app` en paramètre
2. Une méthode `call` qui prend `env` en paramètre et qui retourne un array qui contient
    1. Status (Integer)
    2. Header (Hash)
    3. Body (Array)
 
 ```ruby
app = -> (env) { [200, { 'Content-Type' => 'text/plain' }, ['Hello World']] }

class DurationMiddleware
  def initialize(app)
    @app = app
  end
  
  def call(env)
    t = Time.now
    # Before
    status, headers, body = @app.call(env)
    # After
    puts Time.now - t
    [status, headers, body]
  end
end

Rack::Handler::WEBrick.run(DurationMiddleware.new(app))
```

On peut les enchainés

```ruby
DurationMiddleware.new(
  SecondMiddleware.new(
    app
  )
)
```

C'est pour cela que dans une stacktrace d'un call a l'app rails on voit une bonne dizaine de méthodes `call` qui s'enchaine.

## Builder

Permet de simplifier la définition de l'application

```ruby
my_proc_app = -> (env) { [200, { 'Content-Type' => 'text/plain' }, ['Hello World']] }

app = Rack::Builder.new do
  use DurationMiddleware
  use SecondMiddleware
  run(my_proc_app)
end

Rack::Handler::WEBrick.run(app)
```

## config.ru

La configuration d'une app Rack peux être fait dans un fichier `config.ru` (pour Rackup). Remarquer que l'application *petalmd.rails*
a ce fichier. 

Dans se fichier on peut simplement mettre le contenu du block de `Rack::Builder` et les différents web server font se charger du reste.

```ruby
# config.ru

use DurationMiddleware
use SecondMiddleware
run(app)
```

Dans un terminal (avec la gem puma déjà installer):

```shell
$ puma
```

## À quoi sa peut me servire?

### Spec

#### Original

```ruby
describe Middlewares::AuthenticatePartnerMiddleware do
  subject { response }
  class API < Grape::API
    get('/') do
    end
  end

  def app
    Rack::Builder.app do
      use Middlewares::AuthenticatePartnerMiddleware
      run API.new
    end
  end
  
  it 'raises an error' do
    expect(Rails.env).to receive(:production?) { true }
    expect { get '/' }.to raise_error ApiErrors::UnauthorizedError
  end
end
```

#### Avec des class anonymes et `Rack::Builder`

* Voir chapter passé: [Comment définir une class dans les specs](https://petalmd.github.io/chapter-rails-blog/blog/Nouvelle%20class%20pour%20un%20teste.html#comment-d%C3%A9finir-une-class)

```ruby
describe Middlewares::AuthenticatePartnerMiddleware do
  subject { response }

  let(:api_class) do
    Class.new(Grape::API) do
      use Middlewares::AuthenticatePartnerMiddleware
      get('/') { }
    end
  end

  let(:app) { Rack::Builder.new(api_class) }

  it 'raises an error' do
    expect(Rails.env).to receive(:production?) { true }
    expect { get '/' }.to raise_error ApiErrors::UnauthorizedError
  end
end
```

#### On oublie `Grape::API` un middleware Rack reste un middleware Rack

```ruby
describe Middlewares::AuthenticatePartnerMiddleware do
  subject { response }

  let(:app) do
    proc_app = -> (_env) { [200, { 'Content-Type' => 'text/plain' }, ['Hello world']] }
    described_class.new(proc_app)
  end

  it 'raises an error' do
    expect(Rails.env).to receive(:production?) { true }
    expect { get '/' }.to raise_error ApiErrors::UnauthorizedError
  end
end
```
