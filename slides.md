---
theme: seriph
background: https://source.unsplash.com/collection/94734566/1920x1080
class: text-center
highlighter: shiki
lineNumbers: false
info: 東京.dart Dart/Flutter のOSS 活動・pub パッケージ開発のすすめ
drawings:
  persist: false
transition: slide-left
title: 東京.dart (KosukeSaigusa)
---

# Dart/Flutter の<br>OSS 活動・pub パッケージ<br>開発のすすめ

<div class="flex items-center justify-center">
  <img border="rounded" src="/images/slidev_qr_code.png" class="pb-1">
</div>

東京.dart (2023-06-09)

---

# 自己紹介

<div grid="~ cols-2 gap-2" m="-t-2">

<ul>
  <li>Flutter/Dart 歴 3 年くらい</li>
  <li>本業：Web エンジニア（atama plus株式会社、Django, Python, React, Angular, TypeScript など）</li>
  <li>副業・個人事業：主に Flutter, Dart（某ガソリンスタンドのアプリなど）</li>
  <li>趣味・個人開発：Flutter, Dart が大好き、Firebase も好き</li>
  <li>好きな技術・よく使う技術：Flutter, Dart, Firebase, Django, Python, React, Vue, Angular, Next, Nuxt, TypeScript, Chakra UI, Tailwind CSS, LINE API, LIFF, ...</li>
</ul>

<div class="flex items-center justify-center">
  <div>
    <img border="rounded" src="/images/profile_qr_code.png" class="pb-1">
    <div class="text-center">@KosukeSaigusa</div>
  </div>
</div>

</div>


---
layout: default
---

# 本日の発表について...

<div style="margin-top: 4rem;">
  <h3 style="font-weight: bold;">「OSS 活動のすすめ」</h3>
  <p style="font-size: 1.25rem; line-height: 2rem;">と、銘打っていますが、私も全然 OSS 初心者です！</p>
  <p style="font-size: 1.25rem; line-height: 2rem;">が、これまでの私の OSS との関り方や作ったものを紹介しながら、その楽しさを共有したり、皆さんと繋がるきっかけになったりすれば嬉しいです！🙌</p>
</div>

---

# これまでの（広義の）OSS 活動

- 個人開発のリポジトリをたくさん作って公開する
  - Flutter や Dart の気になる機能やパッケージを使ったレファレンスコード・アプリ
  - ハッカソンで開発したアプリ、その他の遊びで作ったアプリ
- 既存の OSS パッケージに（ちょっとでも）Contribute する
  - バグっぽい現象を見つけたら Issue で報告
  - README のタイポや翻訳関係でも貢献
  - Flutter/Dart SDK の新しい機能が出たらそれを使ってリファクタ
- これまで（小さい）Contribute したことがある主なパッケージ
  - [geoflutterfire](https://pub.dev/packages/geoflutterfire)
  - [wechat_assets_picker](https://pub.dev/packages/wechat_assets_picker)
  - [supabase_auth_ui](https://pub.dev/packages/supabase_auth_ui)
  - ... など

---

# 公開した pub パッケージ

最近公開したパッケージをかんたんに紹介します！

<div grid="~ cols-2 gap-2" m="-t-2">

[geoflutterfire_plus](https://pub.dev/packages/geoflutterfire_plus)

[flutterfire_json_converters](https://pub.dev/packages/flutterfire_json_converters)

<img border="rounded" src="/images/geoflutterfire_plus.png">

<img border="rounded" src="/images/flutterfire_json_converters.png">

</div>

---

# geoflutterfire_plus

<div class="grid grid-cols-4">
  <div class="col-span-3 p-4 overflow-auto">
    <ul>
      <li>Cloud Firestore の Geo query をかんたんに書くためのパッケージ</li>
      <li>既存の geoflutterfire パッケージの後継として開発</li>
      <li>現時点で 15 ヶ月間、依存パッケージの更新（メンテナンス）がされなくなって、使用できなくなっていた</li>
      <li>設計や内部実装にいろいろ問題があり、自分で大部分を書き換えてフォーク版として使っていた</li>
      <li>設計を根本から見直して、同じ機能を維持しつつ、いくつかの機能や重要なユニットテストの追加をした</li>
    </ul>
  </div>
  <div class="col-span-1">
    <img src="/images/geoflutterfire_plus.gif" class="object-cover h-full w-full">
  </div>
</div>

---

# geoflutterfire_plus

Geo query, Geohash について

- Cloud Firestore では複合クエリごとに 1 つの range 句のみしか使用できない
- よって、緯度・経度を別々のフィールドに保存して `minLat < latitude < maxLat` かつ `minLng < longitude < maxLng` のようなクエリは書けない
- そこで、緯度経度の値の組を文字列に変換して扱う仕組みである Geohash が用いられる
  - ある地点の緯度・経度をひとつの文字列として扱う
  - 物理的地点が近ければ、Geohash 文字列も近くなる（アルゴリズムを追えば理由は分かる）
    - 例）東京駅: `xn76urx6`, 新宿駅: `xn774cnf`, 博多駅: `wvuxpfc6`

---

# Geohash の計算

`GeoFirePoint` というクラスを導入。緯度・経度を与えるだけで、勝手に Geohash を計算してくれる。

```dart
/// 緯度・経度を与えて [GeoFirePoint] を定義する。
final GeoFirePoint geoFirePoint = GeoFirePoint(GeoPoint(35.681236, 139.767125));

/// GeoFirePoint.data で、[GeoPoint] インスタンスと Geohash 文字列が得られる。
final Map<String, dynamic> data = geoFirePoint.data;

// {geopoint: Instance of 'GeoPoint', geohash: xn76urx66}
print(data);

// 'xn76urx66'
print(data['geohash']);
```

---

# 位置情報データを Cloud Firestore に保存する

`CollectionReference` を与えて得られる `GeoCollectionReference` を `CollectionReference` と（ほぼ）同様に扱える。

```dart
/// 緯度・経度を与えて [GeoFirePoint] を定義する。
final geoFirePoint = GeoFirePoint(GeoPoint(35.681236, 139.767125));

/// [CollectionReference] の定義。
final collectionReference = FirebaseFirestore.instance.collection('locations');

/// [CollectionReference] を与えて [GeoCollectionReference] を定義する。
final geoCollectionReference = GeoCollectionReference<Map<String, dynamic>>(collectionReference);

/// [CollectionReference<T>] に [T] 型がついていれば [GeoCollectionReference<T>] も型付きで扱える。
final typedGeoCollectionReference = GeoCollectionReference<Location>(typedCollectionReference);

/// GeoCollectionReference.add メソッドで位置情報ドキュメントを作成する。
/// GeoFirePoint.data を任意のフィールド名で含める。
/// その他のフィールドも自由に設定して良い。
geoCollectionReference.add(<String, dynamic>{
  'geo': geoFirePoint.data,
  'name': name,
  'isVisible': true,
});
```

---

# 位置情報データを取得する

それぞれ `Stream`, `Future` を返す 2 種類のメソッドを提供している。

検出中心や半径 (km) を変更するたびに結果を返す `subscribeWithin` メソッド:

```dart
final Stream<List<DocumentSnapshot<Location>> stream = 
  typedGeoCollectionReference.subscribeWithin(
    center: GeoFirePoint(GeoPoint(35.681236, 139.767125)),
    radiusInKm: 50,
    field: 'geo',
    geopointFrom: location.geo.geopoint,
  );
```

 任意のリクエストしたタイミングで結果を返す `fetchWithin` メソッド:

```dart
final Future<List<DocumentSnapshot<Location>> future = 
  typedGeoCollectionReference.fetchWithin(
    center: GeoFirePoint(GeoPoint(35.681236, 139.767125)),
    radiusInKm: 50,
    field: 'geo',
    geopointFrom: location.geo.geopoint,
  );
```

---

# flutterfire_json_converters

<div grid="~ cols-2 gap-2" m="-t-2">

- Dart 3 に実装された `sealed` クラスを早速活用
- Dart の `DateTime` 型と Cloud Firestore の `Timestamp` 型を一緒に扱うための `JsonConverter` を定義
- さらに、書き込み (`create`/`update`) 時には `FieldValue.serverTimestamp()` を自動で付加する
- Dart 3 以前は freezed パッケージの Union/Sealed を使用していた
- 参考：[json_converter_helper](https://pub.dev/packages/json_converter_helper)

<img border="rounded" src="/images/freezed_union_sealed.png">

</div>

---

# 実装内容

sealed クラスの活用

<div grid="~ cols-2 gap-2" m="-t-2">

```dart
/// sealed クラスで定義したタイムスタンプ。
sealed class SealedTimestamp {
  const SealedTimestamp();

  DateTime? get dateTime {
    return switch (this) {
      ClientDateTime(
        clientDateTime: final clientDateTime
      ) =>
        clientDateTime,
      ServerTimestamp() => null,
    };
  }
}
```

```dart
/// クライアントの Dart の [DateTime].
class ClientDateTime extends SealedTimestamp {
  const ClientDateTime(this.clientDateTime);

  final DateTime clientDateTime;
}

/// Cloud Firestore の `FieldValue.serverTimestamp`.
class ServerTimestamp extends SealedTimestamp {
  const ServerTimestamp();
}
```

</div>

---

# 実装内容

<div grid="~ cols-2 gap-2" m="-t-2">

```dart
const sealedTimestampConverter =
    _SealedTimestampConverter();

const alwaysUseServerTimestampSealedTimestampConverter =
    _SealedTimestampConverter(
  alwaysUseServerTimestamp: true,
);

class _SealedTimestampConverter
    implements JsonConverter<SealedTimestamp, Object> {
  const _SealedTimestampConverter({
    this.alwaysUseServerTimestamp = false,
  });

  final bool alwaysUseServerTimestamp;

  // ... 省略（右側参照）
}
```

```dart
{
  // ... 省略

  @override
  SealedTimestamp fromJson(Object json) {
    final timestamp = json as Timestamp;
    return ClientDateTime(timestamp.toDate());
  }

  @override
  Object toJson(SealedTimestamp sealedTimestamp) {
    if (alwaysUseServerTimestamp) {
      return FieldValue.serverTimestamp();
    }
    return switch (sealedTimestamp) {
      ClientDateTime(
        clientDateTime: final clientDateTime
      ) =>
        Timestamp.fromDate(clientDateTime),
      ServerTimestamp() => FieldValue.serverTimestamp(),
    };
  }
}
```

</div>

---
layout: two-cols
---

# 使い方

<div style="padding-right: 4rem;">
  <ul>
    <li>json_serializable や freezed のフィールド定義時の json_converter として使用</li>
    <li>読み込み時: Cloud Firestore の <code>TimeStamp</code> を Dart の <code>DateTime</code> に変換</li>
    <li>書き込み時
      <ul>
        <li><code>createdAt</code>: クライアントから Dart の <code>DateTime</code> を与えればそれを <code>Timestamp</code> に変換。与えられなければ自動で <code>FieldValue.serverTimestamp()</code></li>
        <li><code>updatedAt</code>: 自動で常に <code>FieldValue.serverTimestamp()</code></li>
      </ul>
    </li>
  </ul>
</div>

::right::

```dart
@JsonSerializable()
class Entity {
  Entity({
    this.createdAt = const ServerTimestamp(),
    this.updatedAt = const ServerTimestamp(),
  });

  factory Entity.fromJson(Map<String, dynamic> json) =>
      _$EntityFromJson(json);

  @sealedTimestampConverter
  final SealedTimestamp createdAt;

  @alwaysUseServerTimestampSealedTimestampConverter
  final SealedTimestamp updatedAt;

  Map<String, dynamic> toJson() => _$EntityToJson(this);
}
```

```dart
test('test', () {
  final epoch = DateTime(1970);
  final entity = Entity(createdAt: ClientDateTime(epoch));
  final json = entity.toJson();
  expect(json['createdAt'], Timestamp.fromDate(epoch));
  expect(json['updatedAt'], isA<FieldValue>());
});
```

---

# 今後やってみたいこと

Cloud Firestore のドキュメントのクラス定義と `CollectionReference`, `DocumentReference` を自動生成する仕組みを作りたい（freezed の FlutterFire 特化、Cloud Firestore ODM よりもシンプルで柔軟なもの）

<div grid="~ cols-2 gap-2" m="-t-2">

```dart
@freezed
class Foo with _$Foo {
  const factory Foo({
    @Default('') String fooId,
    @Default('') String someField,
    @sealedTimestampConverter
    @Default(ServerTimestamp()) SealedTimestamp createdAt,
  }) = _Foo;

  factory Foo.fromJson(Map<String, dynamic> json) =>
      _$FooFromJson(json);

  factory Foo.fromDocumentSnapshot(DocumentSnapshot ds) {
    final data = ds.data()! as Map<String, dynamic>;
    return Foo.fromJson(<String, dynamic>{
      ...data,
      'fooId': ds.id,
    });
  }
}
```

```dart
final _db = FirebaseFirestore.instance;

/// foos コレクションの参照。
final foosRef = _db.collection('foos').withConverter(
  fromFirestore: (ds, _) {
    return Foo.fromDocumentSnapshot(ds);
  },
  toFirestore: (obj, _) {
    final json = obj.toJson();
    return json;
  },
);

/// foo ドキュメントの参照。
DocumentReference<Foo> fooRef({required String fooId}) =>
    foosRef.doc(fooId);
```

</div>

---

# まとめ

<!--
- ⚡なぜ OSS をやるの？
  - 私たちエンジニアにとっての自己表現のひとつ
  - 自分が作りたいもの、役に立つものを作るのは楽しい
  - （まだ大きな実績はないけど）それが世界中の人に使われ得るのも楽しい
- 💡 今日からできる（広義の）OSS 活動
  - 機会を見つけて、普段使っている pub パッケージのコードを読んだり、Issue や PR を送ったりしてみる
  - アイディアがまとまれば開発してパッケージをリリースしてみる
  - （今日のような）オフライン・オンラインの勉強会にも参加してみる
- ✌️ ぜひ Twitter や GitHub で繋がりましょう！！
-->

<div grid="~ cols-2 gap-2" m="-t-2">

<ul>
  <li>
    ⚡なぜ OSS をやるの？
    <ul>
      <li>私たちエンジニアにとっての自己表現のひとつ</li>
      <li>自分が作りたいもの、役に立つものを作るのは楽しい</li>
      <li>（まだ大きな実績はないけど）それが世界中の人に使われ得るのも楽しい</li>
    </ul>
  </li>
  <li>
    💡 今日からできる（広義の）OSS 活動
    <ul>
      <li>普段使っている pub パッケージのコードを読んでみる、Issue や PR の作成にも気軽に取り組んでみる</li>
      <li>オフライン・オンラインの勉強会にも参加してみる</li>
    </ul>
  </li>
  <li>✌️ ぜひ Twitter や GitHub で繋がりましょう！！</li>
</ul>

<div class="flex items-center justify-center">
  <div>
    <img border="rounded" src="/images/profile_qr_code.png" class="pb-1">
    <div class="text-center">@KosukeSaigusa</div>
  </div>
</div>

</div>

