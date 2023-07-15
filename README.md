# AviUtl エイリアス - FPS カウンタ
AviUtl の FPS カウンタになる拡張編集のテキストオブジェクトのエイリアスです．パフォーマンスモニタなどにお使いください．動画出力時には自動的に非表示になります．

##	使い方
`FPSカウンタ.exa` をタイムラインにドラッグドロップでテキストオブジェクトが生成され，そのまま FPS カウンタとして機能します．適宜位置やフォントなどを好みに調整してください．調整した後はエイリアス出力で上書きなどすれば次に使うとき手間が省けます．

また，AviUtl の1階層下のフォルダに配置するとタイムラインの右クリックメニューに登録されるようになるのでそこから配置してもいいでしょう．

![表示例](https://github.com/sigma-axis/AviUtl-Alias-FPS-Counter/assets/132639613/14a31e06-1135-4abc-b9b9-eebfd7d02453)

##	機能
次のテキストがセットされています：

```lua
<?local f,N,k="FPS: %2d",4,"**frame counter";
if not obj.getinfo"saving"then
local F,t,s,Q=_G[k]if obj.time<=0 or F==t then Q={}t=-N
F=function(T)if T>t then t,T=T,T-t;s=0
for i=T,N do s=s+Q[i]Q[i-T]=Q[i]end for i=1,math.min(T,N)do Q[N+1-i]=0 end
s=f:format(s)end Q[N]=1+Q[N]mes(s)end;_G[k]=F;end F(math.ceil(N*os.clock()))end?>
```

1行目の `=` より右の部分を変えて挙動をカスタマイズできます．

1.	`"FPS: %2d"` を変更することで書式を指定できます．
	-	書式指定については Lua の [`string.format()`](https://www.lua.org/manual/5.1/manual.html#pdf-string.format) 関数や C言語の [`printf()`](https://learn.microsoft.com/ja-jp/cpp/c-runtime-library/format-specification-syntax-printf-and-wprintf-functions?view=msvc-170) 関数を参照．

1.	次の `4` は1秒間に何回 FPS 表示を更新するかの回数を指定します．
1.	`"**frame counter"` は描画回数などをグローバル変数テーブル `_G` に保存しておくためのキーです．
Lua では変数名として使用できない文字が入っているため他のスクリプトと競合することはまずありません．必要なら適宜変更してください．

現在フレームをオブジェクトの先頭に持ってくると状態がリセットされます．上記の設定などでスクリプトに変更を加えたあとは，リセットをかけて変更を反映させてください．

##	スクリプトについて
なるべく文字数が小さくなるように ~~遊んで~~ 工夫しています．その影響で読みにくくなっていますが普通に書くと次のスクリプトと等価です．

```lua
local f, N, k = "FPS: %2d", 4, "**frame counter"; -- 冒頭はむしろ見やすさ重視．
if not obj.getinfo("saving") then -- 動画出力中はスキップ．
    local Func = _G[k]; -- グローバル変数テーブルから更新関数を取得．

    -- 更新関数がなかったり，オブジェクトの冒頭ならリセット．
    if obj.time <= 0 or Func == nil then -- 実際は未初期化変数の nil 値と比較．
        local Queue = {}; -- 描画回数履歴．
        local tick = -N; -- 最終更新時刻 x N．
        local s; -- 出力文字列を格納．描画回数の部分和も兼任. (`string`/`sum`)

        -- 更新関数を作成. T は現在時刻 x N.
        Func = function(T)
            if T > tick then -- FPS 表示の更新が必要．
                -- 時刻を更新, T は最後の描画からの時間差に再設定．
                tick,T = T,T-tick;

                -- 履歴をずらしながら描画回数の和をとっていく．
                s = 0; 
                for i = T, N do
                    s = s + Queue[i];
                    Queue[i-T] = Queue[i];
                end

                -- ずらしてできた「空白」を 0 で埋める．
                for i=1,math.min(T,N) do Queue[N+1-i] = 0 end

                -- 書式指定 f で s を文字列化．
                s = f:format(s);
            end

            -- カウンタをインクリメント．
            Queue[N] = Queue[N] + 1;

            -- 文字列 s を出力．
            mes(s);
        end

        -- 作成した関数をグローバル変数テーブルに登録．
        _G[k] = Func;
    end

    -- 現在時刻 x N を取得して更新関数を実行．
    Func(math.ceil(N*os.clock()));
end
```

### その他
~~もしもっと文字数を省けるなら省いてみてください．私に見せたら悔しがると思います．~~

...とかなんとかカッコつけて書いてたけど計4回の自己更新．何やってるのだろうか，私は...?

##	動作環境
-	AviUtl (1.10 で確認)
-	拡張編集プラグイン (0.92 で確認)
-	http://spring-fragrance.mints.ne.jp/aviutl

##	免責事項
このソフトウェアは現状有姿で提供されるものとし，いかなる保証もありません．このソフトウェアを使用したことによるいかなる影響や損害等についても私 sigma-axis は責任を取らないものとします．

##	製作者情報・連絡先
**sigma-axis**

-	GitHub: https://github.com/sigma-axis
-	Twitter: https://twitter.com/sigma_axis
-	nicovideo: https://www.nicovideo.jp/user/51492481
