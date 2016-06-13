CuriositySoftware Objective-Cスタイルガイドライン

# はじめに

これは株式会社キュリオシティソフトウェアiOSアプリ開発チームのためのObjective-Cコーディング用のスタイルガイドです。

それぞれの項目には[スタイルに沿った例][スタイルに沿っていない例]およびそれらの[理由]が書かれています。[理由]に対して正当性がない場合においてはスタイルに沿った例のようなコーディングをする必要はありません。

また、これらすべて必須の項目ではなくプロジェクトごとに最適な選択肢があれば、メンバー内で議論しそれを加味する事が前提です。

## イントロダクション

このドキュメントで記載されていないコーディングスタイルについては、Appleが公開しているドキュメントを参考にして下さい。

* [Objective-Cプログラミング言語](https://developer.apple.com/jp/devcenter/ios/library/documentation/ObjC.pdf)
* [Cocoa基礎ガイド](https://developer.apple.com/jp/documentation/Cocoa/Conceptual/CocoaFundamentals/Introduction/chapter_1_section_1.html)
* [Cocoa向けコーディングガイドライン](https://developer.apple.com/jp/devcenter/ios/library/documentation/CodingGuidelines.pdf)
* [iOSアプリケーションプログラミングガイド](https://developer.apple.com/jp/devcenter/ios/library/documentation/iPhoneAppProgrammingGuide.pdf)

## 目次

- [フレームワークを呼び出す場合#importではなく@importを使おう](#フレームワークを呼び出す場合#importではなく@importを使おう)
- [Blocksを引数にとるメソッドの呼び出し時に、外部引数で改行を行うなら波括弧前で改行しインデント位置を調整する](#Blocksを引数にとるメソッドの呼び出し時に、外部引数で改行を行うなら波括弧前で改行しインデント位置を調整する)
- [積極的にドット記法を使おう](#積極的にドット記法を使おう)
- [インデントは4スペースにしよう](#インデントは4スペースにしよう)
- [if文の波括弧は省略しないように](#if文の波括弧は省略しないように)
- [三項演算子はなるべくシンプルに使う](#三項演算子はなるべくシンプルに使う)
- [エラーハンドリング](#エラーハンドリング)
- [メソッド名のスペース](#メソッド名のスペース)
- [変数](#変数)
- [定数](#定数)
- [命名](#命名)
- [init and dealloc](#init and dealloc)
- [リテラル](#リテラル)
- [CGRectGetMinX関数をview.frame.origin.xのつもりで使わない](#CGRectGetMinX関数をview.frame.origin.xのつもりで使わない)
- [enum](#enum)
- [Privateなプロパティは無名カテゴリに記述する](#Privateなプロパティは無名カテゴリに記述する)
- [画像の命名規則](#画像の命名規則)
- [ifによるオブジェクトのnil判定](#ifによるオブジェクトのnil判定)

## フレームワークを呼び出す場合#importではなく@importを使おう

**スタイルに沿った例**

```objc
#import "AppDelegate.h"
@import CoreLocation;
@import CoreBluetooth.CBPeripheralManager;
```

**スタイルに沿っていない例**

```objc
#import "AppDelegate.h"
#import <CoreLocation.h>
#import <CoreBluetooth/CBPeripheralManager.h>

```

**理由**

- コンパイル速度を上昇させるため

## Blocksを引数にとるメソッドの呼び出し時に、外部引数で改行を行うなら波括弧前で改行しインデント位置を調整する

Blocksを引数に取るメソッドがあり外部引数名で改行する場合、Xcodeエディタによりインデントされる位置は深くなり、手動でインデント位置を調整しても新しく作成する行のインデント位置は矯正されます。

**スタイルに沿っていない例**

```objc
[apiClient GET:path
    parameters:parameters
       success:^(NSURLSessionDataTask *task, id responseObject) {

           if (responseObject) {
             ....
           }
```

括弧の位置を改行するこれを回避できます

**スタイルに沿った例**

```objc
[apiClient GET:path
    parameters:parameters
       success:^(NSURLSessionDataTask *task, id responseObject)
{
    if (responseObject) {
      ...
    }
```

ただし、外部引数名の前で開業しなければ波括弧を改行しなくてもインデント位置が深くなることはありません

```objc
[apiClient GET:path parameters:parameters success:^(NSURLSessionDataTask *task, id responseObject) {

    if (responseObject) {
      ...
    }
```

## 積極的にドット記法を使おう

メンバ変数にsetterがある場合（プロパティが公開されている場合）、ドット記法は積極的に使いましょう。

**スタイルに沿った例**

```objc
view.backgroundColor = [UIColor orangeColor];
```

**スタイルに沿っていない例**

```objc
[view setBackgroundColor:[UIColor orangeColor]];
```

**理由**

* プロパティ名に直に代入しているような記述は（実際setterが動作していても）より直感的に見えます。


その他に、メソッドもドット記法により呼び出すことができますが、括弧でのアクセスするほうが好まれる場合もあります。

**スタイルに沿った例**

```objc
[UIApplication sharedApplication].delegate;
```

**スタイルに沿っていない例**

```objc
//+ (UIApplication *)sharedApplication;と定義されているので呼び出せるんだけど…
UIApplication.sharedApplication.delegate;
```

**理由**

* このようなメソッドをドット記法で呼び出せるのはプロパティの仕様上の副産物である気がします。大きなメリットもデメリットもないのであれば、慣習に従い括弧によるメソッド呼び出しのほうがチーム内では適しているでしょう。

**規約上の例外:1**

例えば60fpsが要求されるリアルタイムなゲームアプリを実装する場合、1秒間に60回動作するロジック内でメンバ変数へのアクセスはプロパティ経由ではsetterやgetterのメソッドを動作させる事になります。そのような場面ではあらかじめメンバ変数に直にアクセスしましょう。

**規約上の例外:2**

initメソッドやdeallocメソッドでは自分のメンバ変数に対してプロパティ経由でのアクセスを避けましょう。

## インデントは4スペースにしよう

* インデントは4スペースにしよう。ハードタブ(\t)は使わず、ソフトタブによる4スペースが好ましいです（Xcode5のデフォルトで問題ない）
* ifなどの制御構文後の開始波括弧後は改行しましょう（Xcode5のスニペットで問題ない）
* else ifやelseなどの開始前は改行しないように（Xcode5のスニペットで問題ない）


**スタイルに沿った例**

```objc
if (user.isHappy) {
    //Do something
} else if(user.isDead) {
    //Do something
} else {
    //Do something else
}
```

**スタイルに沿っていない例**

```objc
if (user.isHappy) {
    //Do something
}
else if(user.isDead) {
    //Do something
}
else {
    //Do something else
}
```

***理由***

* ハードタブを使うとエディタによって見た目が変わったり、GitHubなどウェブサービス上での表示で思いもよらない形になります
* else ifやelseの前を改行することでコードをコメントアウトしやすくなるメリットもあるので、そのメリットを享受したいのであれば改行することを検討する必要があるかもしれません

## if文の波括弧は省略しないように

if文では波括弧を使いましょう。

**スタイルに沿った例**

```objc
if (!error1) {
    goto fail;
}

if (!error2) {
    goto fail;
}

//goto文を推奨しているわけでは有りません
```

**スタイルに沿っていない例**

```objc
if (!error1)
    goto fail;

if (!error2)
    goto fail;
```

or

```objc
if (!error1) goto fail;
if (!error2) goto fail;
```

**理由**

* 波括弧を省略することにより処理が短く簡潔になりあなたは自分自身がスマートに見えるでしょうが、それは気のせいです。


### 三項演算子はなるべくシンプルに使う

もし三項演算子を使いたくても、三項演算子を続けて書くという事は避けましょう。

**スタイルに沿った例**

```objc
result = a > b ? x : y;
```

**スタイルに沿っていない例**

```objc
result = a > b ? x = c > d ? c : d : y;
```

**理由**

* 三項演算子を続けて記述することで一行での表現は高くなりますが読みづらさが増します
* 三項演算子を使うことにより、処理が短く簡潔になりあなたは自分自身が賢く見えるでしょうが、それも気のせいです。
* 三項演算子自体を本当に使わなくてはいけないかどうかを考えましょう。

## エラーハンドリング

戻り値によりエラー検出が出来る場合、エラー検出は引数に渡したerrorではなく、戻り値による検出をすればよいでしょう。

**スタイルに沿った例**

```objc
NSError *error;
if (![self trySomethingWithError:&error]) {
    // Handle Error
}
```

**スタイルに沿っていない例**

```objc
NSError *error;
[self trySomethingWithError:&error];
if (error) {
    // Handle Error
}
```

**理由**

理由として、NYTimesのスタイルガイドを引用すると

>>Some of Apple’s APIs write garbage values to the error parameter (if non-NULL) in successful cases, so switching on the error can cause false negatives (and subsequently crash).

「いくつかのAppleのAPIでは成功時にerrorにゴミデータをいれてくるので」とあります。この内容は具体的な発生パターンやメソッド名が不明で試せないので、戻り値で判断できる場合はそれで判断し、メソッドの設計を行う時は可能であれば戻り値をBOOLとする（NSErrorはあくまでエラーの内容を示す）としておくのが無難でしょう。


## メソッド名のスペース


インスタンスメソッドの-やクラスメソッドの+の後にスペースを一つ入れましょう。

**スタイルに沿った例**

```objc
- (void)setExampleText:(NSString *)text image:(UIImage *)image;
```

## 変数

変数のアスタリスクは変数名の前につけましょう。

可能ならばメンバ変数にはアクセサ経由でのアクセスを行いましょう。逆にプロパティによるアクセスを行うべきではない場合についての詳細は[Appleのリファレンスに詳しく書かれています](https://developer.apple.com/library/mac/documentation/Cocoa/Conceptual/MemoryMgmt/Articles/mmPractical.html#//apple_ref/doc/uid/TP40004447-SW6)。

**スタイルに沿った例**

```objc
@interface NYTSection: NSObject

@property (nonatomic) NSString *headline;

@end
```

**スタイルに沿っていない例**

```objc
@interface NYTSection : NSObject {
    NSString *headline;
}
```

**理由**

* C++などではポインタを意識させる必要があったり、メソッド宣言のしやすさからアスタリスクの位置がクラスのすぐ後ろにする方が記述が自然ですが、Objective-Cではポインタ変数を言語上で意識させないためでしょう。

## 定数

定数はstaticに定義し、#defineのようなプリプロセッサ命令を使わないようにしましょう。#defineはコンパイル前にソースコード上の文字列を任意の文字列を置き換えるときや#ifなどの条件のために使いましょう。

**スタイルに沿った例**

```objc
static NSString * const NYTAboutViewControllerCompanyName = @"The New York Times Company";

static const CGFloat NYTImageThumbnailHeight = 50.0;
```

**スタイルに沿っていない例**

```objc
#define CompanyName @"The New York Times Company"

#define thumbnailHeight 2
```

**理由**

- #defineを定数として使っても、さらに再定義をすることで上書きをすることが出来ます（Xcode上では警告表示される）。
- kを頭につけるのはAppleでも古いフレームワークに多く、最近はプリフィックスをつけるものが増えている
 - おそらくkというのは物理定数k(const)を示している（例:ばね定数 F[N]=kx[m]）
 - そもそも物理定数がcでなくkなのはドイツは世界的に物理学が盛んだったためドイツ語省略が物理定数として用いられているのではないか

## 命名

Appleの命名規則に従うべきです。長くなっても説明的な名前にしましょう。

**スタイルに沿った例**

```objc
UIButton *settingsButton;
```

**スタイルに沿っていない例**

```objc
UIButton *setBut;
```

次に、定数やクラス名には3文字のプレフィックスを付けましょう。

**スタイルに沿った例**

```objc
static const NSTimeInterval NYTArticleViewControllerNavigationFadeAnimationDuration = 0.3;
```

**スタイルに沿っていない例**

```objc
static const NSTimeInterval fadetime = 1.7;
```


**規約上の例外**

AppDelegateクラスにプレフィックスは必要ありません。これはApple自身のフレームワークや他のライブラリと衝突する危険性がないからです。


---

また、synthesizeを記述する場合、変数名の前にアンダースコアを付けましょう（現在のLLVMはこれを自動で行ってくれるためsynthesizeを記述する必要は有りません）。


**スタイルに沿った例**

```objc
@synthesize descriptiveVariableName = _descriptiveVariableName;
```

**スタイルに沿っていない例**

```objc
@synthesize descriptiveVariableName = descriptiveVariableName__;
```

## init and dealloc

`dealloc`メソッドはimplementationの直後に書くと良いでしょう。

```objc
@implementation EXCObject
@synthesize dummy = _dummy;

- (void)dealloc {
    //必要であればメンバ変数にnilを代入するなど
    //ARCでなければ[dummy release];
    //ARCでなければ[super dealloc];
}

- (instancetype)init {
    self = [super init]; // or call the designated initalizer
    if (self) {
        // Custom initialization
    }

    return self;
}
```

**理由**

* ARCでない場合、`@synthesize`や`@dynamic`のすぐ下のdeallocでそれらをreleaseすれば、release忘れが減ります。



## リテラル

`NSString`, `NSDictionary`, `NSArray`, `NSNumber`を使う場合は新しいリテラルを使いましょう。

**スタイルに沿った例**

```objc
NSArray *names = @[@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul"];
NSDictionary *productManagers = @{@"iPhone":@"Kate", @"iPad":@"Kamal", @"Mobile Web":@"Bill"};
NSNumber *shouldUseLiterals = @YES;
NSNumber *buildingZIPCode = @10018;
```

**スタイルに沿っていない例**

```objc
NSArray *names = [NSArray arrayWithObjects:@"Brian", @"Matt", @"Chris", @"Alex", @"Steve", @"Paul", nil];
NSDictionary *productManagers = [NSDictionary dictionaryWithObjectsAndKeys: @"Kate", @"iPhone", @"Kamal", @"iPad", @"Bill", @"Mobile Web", nil];
NSNumber *shouldUseLiterals = [NSNumber numberWithBool:YES];
NSNumber *buildingZIPCode = [NSNumber numberWithInteger:10018];
```

## CGRectGetMinX関数をview.frame.origin.xのつもりで使わない

CGRectGetMinX関数およびCGRectGetMinY関数はUIViewなどのframe.origin.xやyなどのように使うのは避けましょう。

**スタイルに沿った例**

```objc
CGRect frame = self.view.frame;

CGFloat originX = frame.origin.x;
CGFloat originY = frame.origin.y;
```

**スタイルに沿っていない例**

```objc
CGRect frame = self.view.frame;

CGFloat originX = CGRectGetMinX(frame);
CGFloat originY = CGRectGetMinY(frame);
```

**理由**

- 例えばframe = {{0, 0}, {-100, 100}}の場合、frame.origin.xは0だがCGRectGetMinX(frame)は-100になります。
- 例えば計算にてframeのX座標値の最小値が必要な場合などにCGRectGetMinXを使うのはコードを読む人にとって優しいやり方です。

## enum

enumを使う場合にはNS_ENUMを使いましょう。

**スタイルに沿った例**

```objc
typedef NS_ENUM(NSInteger, NYTAdRequestState) {
    NYTAdRequestStateInactive,
    NYTAdRequestStateLoading
};
```

**理由**

- enumはC++11の標準化によりコンパイラは基盤となる型を先行宣言出来るようになった
 - それによって型安全になった
- NS_ENUMはそのためのマクロで古いコンパイラにも対応している
- NS_ENUMはswitch文の記述漏れもチェックできる

--

ビットマスク定義用にはNS_OPTIONSを使いましょう。

**スタイルに沿った例**

```objc
typedef NS_OPTIONS(NSUInteger, CSIFlag) {
    CSIFlagNone                  = 0,
    CSIFlagRed                   = 1 << 0,
    CSIFlagBlack                 = 1 << 1,
    CSIFlagWhite                 = 1 << 2,
};
```

**理由**

- ビットマスクによりOR演算を行った場合、その結果は指定した基盤の型（例ではNSUInteger）になるが、代入時などはenum（例ではCSIFlag）になる。その時C++/Objective-C++であれば暗黙的なキャストが許されないのでNS_OPTIONSはマクロによりキャストしないで済むようにしている
 - NS_OPTIONSのマクロ参照
 - 参考：Effective Objective-C（p21）


## Privateなプロパティは無名カテゴリに記述する

Privateなプロパティは無名カテゴリに記述しましょう。

**スタイルに沿った例**

```objc
@interface NYTAdvertisement ()

@property (nonatomic, strong) GADBannerView *googleAdView;
@property (nonatomic, strong) ADBannerView *iAdView;
@property (nonatomic, strong) UIWebView *adXWebView;

@end
```

## 画像の命名規則

全て小文字でスネークケースし英単語を省略しないようにしましょう。

**スタイルに沿った例**

* `star_button.png`

**スタイルに沿っていない例**

* `star_btn.png`

**理由**

- 小文字か大文字かをコーディング時に迷わないようにする
- 変数名などの省略しない命名規則に併せる
- 素材を準備するデザイナはキャメルケースよりスネークケースに慣れている

## ifによるオブジェクトのnil判定

オブジェクトがnilかどうか評価したい時、if文での条件式内で `== nil`とせず、nilじゃないかどうかを評価したい場合も`!= nil`ともしないようにしましょう。

**スタイルに沿った例**

```objc
if (!someObject) {
}
```

**Not:**

```objc
if (someObject == nil) {
}
```

**理由**

- Objective-CではnilはNOとしてデザインされている
