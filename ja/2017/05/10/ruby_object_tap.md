# `Object#tap` についての雑感 [Ruby]

仕事で `Object#tap` について話すことがなんどかあったので、まとめてみた。


## `Object#tap` とは

> self を引数としてブロックを評価し、self を返します。
> メソッドチェインの途中で直ちに操作結果を表示するために メソッドチェインに "入り込む" ことが、このメソッドの主目的です。
>
> https://docs.ruby-lang.org/ja/latest/class/Object.html#I_TAP

つまり、以下のような動作をする。

```ruby
a = "hoge"
c = a.tap do |b|
  puts b # => hoge
end
puts c # => hoge
```

`tap`に渡されたブロック内では`tap`のレシーバーを引数として参照することが出来る。
また、`tap`に渡されたブロックの戻り値には関係なく、`tap`の戻り値はレシーバーになる。

そのため、上記ドキュメントでは以下のような使い方が想定されている。

```ruby
# select した直後の様子を見たい
array
  .select{|foo| cond(foo)}
  .tap{|foos| p foos}
  .map{|foo| foo.to_s}
```


## 他に tap の便利な使い方

また、`tap`には上記以外にも便利な使い方がある。

### 値に変更を加えてから返す

```ruby
def some_user(id)
  user = User.find(id)
  user.attr_1 = "some value1"
  user.attr_2 = "some value2"
  user
end
```

このような、`user`変数の値に変更を加えてから返すメソッドを定義することはよくある。
最後に`user`と変数名だけを書かなければいけないことに微妙な気持ち悪さを感じるが、このコードは`tap`を使うと以下の様にリファクタリング出来る。

```ruby
def some_user(id)
  User.find(id).tap do |user|
    user.attr_1 = "some value1"
    user.attr_2 = "some value2"
  end
end
```


### 変数のスコープを作る

```ruby
def some_user(id)
  user = User.find(id)
  user.attr_1 = "some value1"
  user.attr_1 = "some value2"
  user.save!

  # 以降そこそこ処理が続くが `user` 変数は登場しない
  # ...
end
```

上記のような場合、(そもそも設計がおかしいのではという話もあるが)一部にしか登場しない`user`変数がメソッドが終わるまで生き続けるのが気持ち悪い、と感じることもあるだろう。
これも`tap`を使用することでリファクタリングを行うことが出来る。

```ruby
def some_user(id)
  User.find(id).tap do |user|
    user.attr_1 = "some value1"
    user.attr_1 = "some value2"
    user.save!
  end

  # 以降`user`変数は参照できない
  # ...
end
```

もっとも、私はこのようなリファクタリングを行ったことは無いと思う。メソッドを分割するなりいい感じにする方法は他にあるだろう。


### 感情的にまとめたいものをまとめる

これは完全に感情的な理由で、私は「感情的には分かるけどうーん」という気分なので良いコードなのかはよくわからない。

```ruby
def something(params)
  someuser = User.find_or_initialize_by(name: params[:name]).tap do |user|
    user.foo = params[:foo]
    user.bar = params[:bar]
  end

  hogehoge(someuser)
end
```

このコードは、以下と同じ動きをする。

```ruby
def something(params)
  someuser = User.find_or_initialize_by(name: params[:name])
  someuser.foo = params[:foo]
  someuser.bar = params[:bar]

  hogehoge(someuser)
end
```

この例は先程までの2例とは違い、`someuser`の値は後から使用されるし、`someuser`の値がメソッドの戻り値として使用されるわけでもない。
そのため、`tap`を使うことのメリットは無いように思える。

だがこの手法には`tap`を使うことで「ここが初期化プロセスである感」を醸し出すことが出来る、という感情的な利点が存在する。
この利点は「`tap`に慣れていない人がぱっと見でよくわからない可能性」などに比べれば小さいものだと考えている為、私はこのような`tap`の使用方法は微妙だと思う。
ただ、言いたいことの理解は出来る。
