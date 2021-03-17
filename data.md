# データの保存、集約、操作
## jsPsychのデータ：永久保存データと非永久保存データ。

データの保存には、メモリ上に保存されるデータと永続的に保存されるデータという非常に異なる2つの種類があります。永続的に保存されるデータは、jsPsychを実行しているブラウザが終了した後も存在し、通常はデータベースやサーバー上のファイルに保存されます。メモリに保存されたデータは、jsPsychを実行しているブラウザのウィンドウが開いている間だけ存在します。

jsPsychには、メモリに保存されたデータを操作するための多くの機能がありますが、データを永続的に保存するための機能はほとんどありません。データを恒久的に保存する方法は何十通りもあるので、これは意図的に選択されたものです。jsPsychはユーザを特定のソリューションに拘束することはありません。このページの後半では、データを永久保存するための方法をいくつか紹介します。

## jsPsychのデータ構造にデータを格納する

jsPsychには、実験の進行に合わせて構築されるデータの一元的なコレクションがあります。データにアクセスするには、jsPsych.data.get()などのさまざまな関数を使用します。

ほとんどの場合、データの収集は自動的に行われ、非表示になります。プラグインは勝手にデータを保存するので、データを扱うのは実験の最後に恒久的に保存するときだけ、ということも珍しくありません（その方法については以下のセクションを参照してください）。しかし、データを操作したい場合もあるでしょう。特に、被験者の識別子や条件の割り当てなど、プラグインが記録していない追加データを保存したい場合があります。また、試験ごとにデータを追加したい場合もあります。例えば、Stroopパラダイムでは、どの試行が一致していて、どの試行が不一致であるかをラベル付けしたいと思います。これらのシナリオについては、以下で説明します。

## すべての試行にデータを追加する

実験中のすべての試行にあるデータを追加することは、しばしば有用である。例えば、被験者のIDを各試行に追加するなどです。これはjsPsych.data.addProperties()関数で行うことができます。以下にその例を示します。


```
// generate a random subject ID with 15 characters
var subject_id = jsPsych.randomization.randomID(15);

// pick a random condition for the subject at the start of the experiment
var condition_assignment = jsPsych.randomization.sampleWithoutReplacement(['conditionA', 'conditionB', 'conditionC'], 1)[0];

// record the condition assignment in the jsPsych data
// this adds a property called 'subject' and a property called 'condition' to every trial
jsPsych.data.addProperties({
  subject: subject_id,
  condition: condition_assignment
});
```

## 特定のトライアルやトライアルセットへのデータの追加

特定のトライアルにデータを追加するには、そのトライアルのデータパラメータを設定する。data parameterはkey-valueペアのオブジェクトであり、各ペアはそのトライアルのデータに追加される。

```
var trial = {
  type: 'image-keyboard-response',
  stimulus: 'imgA.jpg',
  data: { image_type: 'A' }
}
```

このように宣言されたデータは、入れ子になっているタイムラインのトライアルにも保存されます。

```
var block = {
  type: 'image-keyboard-response',
  data: { image_type: 'A' },
  timeline: [
    {stimulus: 'imgA1.jpg'},
    {stimulus: 'imgA2.jpg'}
  ]
}
```

トライアルのデータオブジェクトは、on_finishイベントハンドラでも更新できます。プロパティを上書きしたり、新しいプロパティを追加することができます。これは、値がトライアル中に起こったことに依存するような場合に特に有効です。

```
var trial = {
  type: 'image-keyboard-response',
  stimulus: 'imgA.jpg',
  on_finish: function(data){
    if(jsPsych.pluginAPI.compareKeys(data.response, 'j')){
      data.correct = true;
    } else {
      data.correct = false;
    }
  }
}
```


