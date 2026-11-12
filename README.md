# AI MUSIC

AIで生まれた音楽だけを集めた、スマホ向けのストリーミングアプリです。聴く・集める（お気に入り／プレイリスト）・フォローする・投稿する、ができます。年末のイベント「AI紅白歌合戦」は、このアプリの中の1つの機能（「紅白」タブ）です。

## フォルダの構成

| 場所 | 中身 |
| --- | --- |
| public/ | 公開されるサイト本体（index.html、config.js など） |
| supabase/ | データベースの定義（schema.sql）、審査員評価の追加分（schema_judges.sql）、音楽アプリ化の追加分（schema_music.sql）、サンプル曲（seed.sql）。自動では反映されません |
| terms.md | 利用規約などの文面。terms.html の元になる、編集用の原稿です |
| wrangler.jsonc | Cloudflareに「public の中身を公開する」と伝える設定 |

## 自動で公開に反映する手順

1. GitHubで新しいリポジトリを作る（Privateで構いません）。
2. このフォルダの中身を、リポジトリにアップロードする（「Add file」>「Upload files」に、フォルダごとドラッグ）。
3. Cloudflareの Workers & Pages で、今のアプリ（black-rain-abc0）を開き、Settings > Builds から GitHub を接続して、作ったリポジトリを選ぶ。
4. ビルドコマンドは空のまま、デプロイコマンドは `npx wrangler deploy` にする。

以後は、GitHubの main ブランチにファイルを更新（コミット）するたびに、自動で公開が更新されます。

## 注意

- `public/config.js` の Supabase のキーは、公開しても問題ない publishable key です。service_role キーは絶対に入れないでください。
- データベースを変える場合は、`supabase/schema.sql` を Supabase の SQL Editor で実行します。
- `wrangler.jsonc` の `name` は、Cloudflare上のアプリ名と同じにしてください。違うとビルドが失敗します。

## 音楽アプリ機能（プレイリスト・フォロー・再生回数）を使うとき

1. Supabaseの SQL Editor で `supabase/schema_music.sql` を実行します（`schema.sql` の実行後に、1回だけ。何度実行しても壊れません）。
2. 実行前は、曲を聴く機能は動きますが、プレイリスト・フォロー・再生回数・「紅白に出ない曲」の投稿は使えません。

## パスワードを忘れたとき

ログイン画面の「パスワードを忘れた方はこちら」からメールを送ると、新しいパスワードを決める画面に戻ってきます。次の設定が必要です。

- Supabase の Authentication > URL Configuration の Site URL と Redirect URLs に、公開しているURLを入れる。
- メールは Supabase 標準の送信だと1時間あたりの数が少ないので、本番前に Authentication > SMTP Settings で独自SMTP（Gmailなど）を設定する。

## 審査員評価・年間グランプリを使うとき

1. Supabaseの SQL Editor で `supabase/schema_judges.sql` を実行します（`schema.sql` の実行後に、1回だけ）。
2. 審査員を登録します。審査員になってもらう人が、一度アプリにログインしてから、次を実行します。

```sql
insert into public.judges (user_id)
select id from auth.users where email = '審査員のメールアドレス';
```

3. 審査員は、曲を開くと「審査員評価」が出ます。評価は本人にしか見えません。
4. 年末に、運営者のマイページの「結果を全員に公開する」を押すと、ランキングの「年間GP」が全員に見えるようになります。
5. 審査員評価の割合を変えるときは、次を実行します（0.5 が50%）。

```sql
update public.app_settings set value = '0.4'::jsonb where key = 'judge_weight';
```

## 利用規約などの文面

`public/terms.html` が、アプリの下部のリンク先になります。【 】の部分（運営者名、連絡先、裁判所など）を埋めて、先頭の「ひな形です」の行を削除してください。専門家の確認を受けてから公開してください。
