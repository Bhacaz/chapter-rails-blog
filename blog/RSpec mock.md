---
layout: default
title: RSpec mock
parent: Blog
nav_order: 6
---

# RSpec mock

Prémice, `allow_any_instance_of`, `expect_any_instance_of` et `receive_message_chain` sont peu recommendé. On peut les utilisé, mais souvant cela indique probablement un problème de design. Ou bien que stubber Grape ou ActiveRecord c'est compliqué.

https://relishapp.com/rspec/rspec-mocks/v/3-10/docs/working-with-legacy-code/any-instance

## Stub VS Mock

* **Stub:** a dummy piece of code that lets the test run, but you don't care what happens to it.
* **Mock:** a dummy piece of code, that you VERIFY is called correctly as part of the test.
* **Spy:** a dummy piece of code, that intercepts some calls to a real piece of code, allowing you to verify calls without replacing the entire original object.

https://stackoverflow.com/a/27136567/3123484

## Doubles

```ruby
require "rspec/mocks/standalone"
```

### Basic

`double` crée un simple object. Par défaut un `double` est _strict_ et ne répond a aucune méthode.

```ruby
dbl = double # => #<Double (anonymous)>
dbl = double('abc') # => #<Double "abc">
```

Le `'abc'` est facultatif, mais recommenter simplement pour la claireter.

### Allow

Permet de définir les méthodes qu'un `double` peut recevoir.

En bulk:

```ruby
dbl = double('abc', hello: 'world', to_a: []) # => #<Double "abc">
dbl.hello # => "world"
dbl.to_a # => []
```

Via `allow`:

```ruby
dbl = double('abc') # => #<Double "abc">
allow(dbl).to receive(:hello).and_return('world')
dbl.hello # => "world"
```

Via `expect`:

```ruby
dbl = double('abc') # => #<Double "abc">
expect(dbl).to receive(:hello).and_return('world')
dbl.hello # => "world"
```

Si on ne spécifie pas de `and_return` c'est `nil` qui est retourné.

### Verifiying doubles

https://relishapp.com/rspec/rspec-mocks/v/3-10/docs/verifying-doubles

Double encore plus strict, on peut seulement stubber des méthodes donc l'object réponds.

* `instance_double`
* `class_double`
* `object_double`

#### `instance_double`

Pour stubber les méthodes d'instance.

```ruby
instance_double(Account, name: 'Bob')
```

```ruby
class MyService
  def initialize(auth_account)
    @auth_account = auth_account
  end

  def greeting
    "Hi, my name is #{@auth_account.name}"
  end
end

describe MyService do
  let(:instance) { described_class.new(account_double) }
  let(:account_double) { instance_double(Account) }
  describe '#greeting' do
    subject { instance.greeting }

    it 'return a greeting string' do
      expect(account_double).to receive(:name).and_return('Bob')
      is_expected.to eq('Hi, my name is Bob')
    end
  end
end
```

#### `class_double`

Pour les méthodes de class.

```ruby
class_double(Account, where: [])
```

#### `object_double `

Pour stubber un object.

```ruby
params = {}
dbl_params = object_double(params, :[] => 'foo')
dbl_params[:xyz] # => "foo"
```

### Parial double

Si on reprend l'example avec `object_double`, on peut aussi passer par les allow/expect, pour stubber partiellement l'object.

```ruby
params = {}
allow(params).to receive(:[]).and_return('foo')
params[:xyz] # => "foo"
```

La différence c'est que avec le allow/expect `params` stub seulement une méthode tandis que avec le `object_double` seulement la méthode stubber peut être appeler.


## Return

Voir la documentation pour plus de détails. https://relishapp.com/rspec/rspec-mocks/v/3-10/docs/configuring-responses

* and_return
* and_raise
* and_throw
* and_yield
* and_call_original
* and_wrap_original

## Matching arguments

La documentation a un beau tableau très facile à lire. https://relishapp.com/rspec/rspec-mocks/v/3-10/docs/setting-constraints/matching-arguments

Mention spéciale aux _keywords_ `anything` et `any_args`.

Example:

```ruby
expect(MyService).to receive(:new).with(auth_account, any_args)
```

## Stub constant

https://relishapp.com/rspec/rspec-mocks/v/3-10/docs/mutating-constants

Peut-être pratique pour mocker des constante seulement définie dans un autre environnement de test ou en changer la valeur. 

Example: [spec/libs/error_handler_spec.rb:20](https://github.com/petalmd/petalmd.rails/blob/master/spec/libs/error_handler_spec.rb#L20)

## Example complet

```ruby
class MyService
  def initialize(auth_account)
    @auth_account = auth_account
  end

  def greeting
    "Hi, my name is #{@auth_account.name}"
  end
end

class GreetingSerializer
  include BrightSerializer::Serializer
end

class Api
  get '/' do
    greeting = MyService.new(auth_account).greeting
    GreetingSerializer.new(greeting)
  end
end

describe Api do
  describe 'GET /' do
    subject do
      get '/'
      response
    end

    let(:service_double) { instance_double(MyService) }
    let(:greeting_double) { double('greeting') }

    it 'call the service and the serializer' do
      expect(MyService).to receive(:new).and_return(service_double)
      expect(service_double).to receive(:greeting).and_return(greeting_double)
      expect(GreetingSerializer).to receive(:new).with(greeting_double)
      is_expected.to be_successful
    end
  end
end
```
