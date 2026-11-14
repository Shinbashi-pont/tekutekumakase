# てくてくまかせ

キャラは勝手に歩きます。あなたは道具を3つ、持たせるだけ。
自動生成されるステージをどこまで進めるか挑む、エンドレスのパズルアクションです。

このフォルダの `index.html` 1つでゲーム全体が動きます。みんなのランキングは Supabase に保存されます。

## GitHub Pages で公開する手順

1. GitHub で新しいリポジトリを作る（例：`tekuteku`、Public）。
2. 「Add file」→「Upload files」で `index.html` と `README.md` をアップロードし、「Commit changes」。
3. リポジトリの「Settings」→「Pages」を開く。
4. 「Build and deployment」の Source を「Deploy from a branch」、Branch を `main` / `/ (root)` にして「Save」。
5. 1〜2分待つと、同じ画面に公開URL（`https://ユーザー名.github.io/tekuteku/`）が表示される。

## ランキングの動作確認

1. 公開URLを開き、タイトル画面の「みんなのランキング」を押す。「まだ登録がありません」と出れば接続できています。
2. 1回遊んでゲームオーバーにし、スコア画面で名前を入れて「登録する」を押す。
3. Supabase のダッシュボードで「Table Editor」→ `scores` を開くと、登録された行が見えます。

うまくいかないときは、ブラウザの開発者ツール（Console）に出るエラーを確認してください。
「ランキングサーバーに接続できませんでした（401）」なら鍵、「（404）」なら表の名前、「（403）」なら行レベルセキュリティのポリシーを見直します。

## Supabase 側の設定（作成済みのもの）

```sql
create table public.scores (
  id bigint generated always as identity primary key,
  name text not null check (char_length(name) between 1 and 12),
  score int not null check (score between 0 and 1000000),
  stage int not null check (stage between 1 and 999),
  created_at timestamptz not null default now()
);
alter table public.scores enable row level security;
create policy "anyone can read" on public.scores for select to anon using (true);
create policy "anyone can add"  on public.scores for insert to anon with check (true);
create index scores_score_idx on public.scores (score desc);
```

- ゲームに書かれている `sb_publishable_...` の鍵は、公開してよい鍵です。できるのは上のポリシーで許した「読む」「追加する」だけです。
- `sb_secret_...` の鍵は、絶対にファイルに書かないでください。
- 不適切な名前やおかしなスコアは、ダッシュボードの「Table Editor」から行を選んで削除できます。
- ブラウザの開発者ツールを使えば偽のスコアを送れてしまう仕組みです。気になる場合は、サーバー側でゲームを再生して確かめる方式に切り替えられます。
