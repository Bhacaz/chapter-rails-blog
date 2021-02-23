---
layout: default
title: RSpec
parent: Bon examples
nav_order: 3
grand_parent: Référence
---

# RSpec

## Simple class

```ruby
class Printer
  def upcase(text)
    text.upcase
  end

  def self.greeting(name)
    "Greeting, #{name}"
  end
end
```

Élément à avoir:

1. Un `describe` avec la class à tester.
2. Une variable `instance` au sommet (si ça s'applique)
3. Un `describe` par méthode

```ruby

describe Printer do
  let(:instance) { described_class.new }

  describe '#upcase' do
    subject { instance.upcase(text) }

    let(:text) { Faker::Lorem.word }

    it 'will be upcase' do
      expect(subject).to eq text.upcase
    end
  end

  describe '.greeting' do
    subject { described_class.greeting(name) }

    let(:name) { Faker::Name.name }

    it 'had greeting before the name' do
      expect(subject).to eq "Greeting, #{name}"
    end
  end
end
```

## Exemple models et `let!`

`let!` permet d'évaluer le block automatiquement avant d'exécuter le `it`.
Pratique pour générer du data avant un teste.

```ruby
describe Account do
  let(:instance) { FactoryBot.create :account }

  describe '#memberships' do
    subject { instance.memberships }

    context 'no memberships' do
      it { is_expected.to be_empty }
    end

    context 'with memberships' do
      let!(:membership) { FactoryBot.create :membership, account: instance }

      it 'have one membership' do
        expect(subject.count).to eq 1
        expect(subject.first).to eq membership
      end
    end
  end
end
```
