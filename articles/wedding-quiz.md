---
title: "自分の結婚式用に自作アプリを作成した話"
emoji: "💍"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: [
  "railway",
  "laravel",
  "vue",
  "Inertia",
  "firebase",
]
published: true
---
# きっかけ
2024年12月に自分の結婚式をしました 💍

演出として、新郎新婦にまつわるクイズ大会を企画したのですが、いろいろ調べてもクイズを実施するツールで自分の望むものが見つからなかったので、自作アプリを作成することにしました。

# アプリの概要
FirebaseのRealtime Databaseを使用し、リアルタイムで画面が描画されるクイズアプリを作成しました。
Repositoryは[こちら](https://github.com/tactical-k/party-quiz)です。

## デモ
![](/images/quiz_movie.gif)

## 使用技術
- フロントエンド: Vue.js(Inertia.js)
- デザイン: TailwindCSS, DaisyUI
- バックエンド: Laravel
- データベース: MySQL, Firebase Realtime Database
- デプロイ先: Railway

使用技術はこのようになりました。
開発期間があまり取れなかった（2週間ほど）ので、自分が一番得意なLaravelをベースに選定しています。
UIは以前調べて興味のあった、[DaisyUI](https://daisyui.com/)を使用しました。
フロントエンドは、LaravelのBlade Templateを使用するか迷いましたが、VueでのSPA経験があったのでInertia.jsを使用しました。

## インフラ構成
Railway上にLaravelアプリケーションとMySQLをホスティングしています。
FirebaseのRealtime DatabaseはFirebaseのSDK（[laravel-firebase](https://github.com/kreait/laravel-firebase)）を経由して、Laravelからアクセスしています。
![インフラ構成](/images/railway_architecture_quiz.png)

# やってみて
## Laravel & Inertia.js
Laravel & Inertia.jsの組み合わせは結論から言うと正解でした。

今回、出題者側にはログイン機能が必要でしたが、SPAでの開発において認証機能はある程度の工数がかかるかと思います。
SPAでの開発において認証機能は多く頭を悩ませるものですが、Inertia.jsからLaravel Breezeの認証機能を呼び出すことができるので、容易に認証機能を実装できました。


## Realtime Database
今回Realtime Databaseは出題する問題を格納し、vue.jsから変更を監視する方法で実装しました。
データ永続化のためのDBとしては私はRDBが好きなのですが、このような使い方を試すことができたのではいい経験でした。

### Laravel側実装イメージ
```php
use Kreait\Firebase\Factory;

public function __construct()
{
    $firebase = (new Factory)->withServiceAccount([
        'type' => env('FIREBASE_TYPE'),
        'project_id' => env('FIREBASE_PROJECT_ID'),
        'private_key_id' => env('FIREBASE_PRIVATE_KEY_ID'),
        'private_key' => str_replace('\\n', "\n", env('FIREBASE_PRIVATE_KEY')),
        'client_email' => env('FIREBASE_CLIENT_EMAIL'),
        'client_id' => env('FIREBASE_CLIENT_ID'),
        'auth_uri' => env('FIREBASE_AUTH_URI'),
        'token_uri' => env('FIREBASE_TOKEN_URI'),
        'auth_provider_x509_cert_url' => env('FIREBASE_AUTH_PROVIDER_CERT_URL'),
        'client_x509_cert_url' => env('FIREBASE_CLIENT_CERT_URL'),
        'universe_domain' => env('FIREBASE_UNIVERSE_DOMAIN'),
    ]);
    
    $this->database = $firebase->withDatabaseUri(env('VITE_FIREBASE_DATABASE_URL'))->createDatabase();
}

public function syncQuestion($question)
{
  $this->database->getReference("{$question->event_id}/currentDisplay")->set([
      'type' => 'question',
      'question_id' => $question->id,
      'text' => $question->text,
      'choices' => $question->choices->pluck('text'),
  ]);
  return ['status' => 'success'];
}
```

### Vue.js側実装イメージ
```js
<script setup>
import { ref as dbRef, onValue, push } from 'firebase/database';

{/* 略 */}

// 質問をリアルタイムで取得
onMounted(() => {
    // 問題の取得元はFirebase
    const questionRef = dbRef(database, `${props.event_id}/currentDisplay`);
    onValue(questionRef, (snapshot) => {
        const data = snapshot.val();
        if (data) {
            question.value = data;
            // Firebaseイベント発火時にanswerをリセット
            answer.value = '';
        }
    });
});
</script>
```

# つまづいた点
## Railwayへのデプロイ
Laravel+Inertia.js+TailwindCSSのアプリケーションの環境構築を行う記事は大量にあるのですが、どれもローカル環境で立ち上げるまでが大半で、実際に本番環境へデプロイするまでの記事はあまり見つかりませんでした。
その結果、Railway上でLaravelアプリケーションを立ち上げるまでに一番時間がかかりました。

中でもInertia.jsが悪さをしていそうで、migrationなどは通っていてLaravelは立ち上がっていそうなのに、画面に何も映らないと言う症状にも遭遇しました。
（これは一晩寝たら解決しました。睡眠大事🛌）

### 解消したコミット
HTTPSを強制しないといけない
https://github.com/tactical-k/party-quiz/commit/6c9f9655532f1cd6be1ef0162935e8e3083a4c24

また、RailwayのBuilderにはNixpacksが使用されているのですが、これがよくわからず原因を特定するのに時間がかかってしまいました。
:::message
DockerFileを使用してbuildすることもできるようなので、Dockerへの理解がある方はDockerFileを使用してbuildすることをおすすめします。
:::

## 時間が足りない
開発期間が2週間ほどしかなかったので、あまり時間が取れなかったと言うのと、一度の結婚式のために作り始めたアプリなので簡単に妥協することができず、テストコードをサボり始めたり、RoutesにDBアクセスを書き始めたりしてしまいました。

# 終わりに
5年くらいWebエンジニアをやってきてますが、技術選定・インフラ構築・デプロイまで自分でやることはあまりなかったので、このような経験を積むことができてよかったです。（結婚式の私物化ともいう）

また、今回で `Laravel + Inertia + tailwind + Railway` の環境構築のノウハウがかなり溜まったので、また記事にしたいと思います。

