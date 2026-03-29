# Mail Notification Reuse

このリポジトリでは、GitHub Actions のメール通知を reusable workflow に切り出しています。

## 追加した reusable workflow

- `/.github/workflows/send-update-mail.yml`

他の workflow から job として呼び出します。

```yaml
notify:
  needs: sync
  if: needs.sync.outputs.updated == 'true'
  uses: ./.github/workflows/send-update-mail.yml
  with:
    enabled: ${{ secrets.MAIL_ENABLED == 'true' }}
    subject: my-repo updated to ${{ needs.sync.outputs.current_updated_at }}
    summary: my-repo was updated successfully.
    previous_updated_at: ${{ needs.sync.outputs.previous_updated_at }}
    current_updated_at: ${{ needs.sync.outputs.current_updated_at }}
    commit_sha: ${{ needs.sync.outputs.commit_sha }}
  secrets:
    SMTP_SERVER: ${{ secrets.SMTP_SERVER }}
    SMTP_PORT: ${{ secrets.SMTP_PORT }}
    SMTP_USERNAME: ${{ secrets.SMTP_USERNAME }}
    SMTP_PASSWORD: ${{ secrets.SMTP_PASSWORD }}
    MAIL_TO: ${{ secrets.MAIL_TO }}
    MAIL_FROM: ${{ secrets.MAIL_FROM }}
```

`needs.sync.outputs.*` は、通知元の job で `outputs:` に出しておきます。

## どこに何を入力するか

### 1. 共通で使い回す値

複数リポジトリで同じ SMTP 設定を使うなら、GitHub Organization Secrets に置くのが管理しやすいです。

GitHub の Organization で以下を登録します。

- `SMTP_SERVER`
- `SMTP_PORT`
- `SMTP_USERNAME`
- `SMTP_PASSWORD`
- `MAIL_FROM`
- `MAIL_TO`
- `MAIL_ENABLED`

画面の場所:

1. GitHub を開く
2. Organization を開く
3. `Settings`
4. `Secrets and variables`
5. `Actions`
6. `New organization secret`

Organization を使わずリポジトリ単位にしたい場合は、各リポジトリの以下に同名で登録します。

1. 対象リポジトリを開く
2. `Settings`
3. `Secrets and variables`
4. `Actions`
5. `New repository secret`

### 2. 各 workflow ごとに書く値

各 workflow では以下だけ決めます。

- いつ通知するか
- 件名をどうするか
- 本文先頭の要約をどうするか
- 通知に載せたい output をどの job から渡すか

つまり、SMTP 接続情報は共通化でき、workflow 側には通知を呼ぶ job だけを書けば済みます。

## 他リポジトリで使う方法

### 同じリポジトリ内の別 workflow で使う

そのまま `uses: ./.github/workflows/send-update-mail.yml` を書きます。

### 別リポジトリから使う

この workflow を置いたリポジトリを参照して呼びます。

```yaml
notify:
  needs: sync
  if: needs.sync.outputs.updated == 'true'
  uses: sironekotoro/zengin-data-mirror/.github/workflows/send-update-mail.yml@main
  with:
    enabled: ${{ secrets.MAIL_ENABLED == 'true' }}
    subject: other-repo updated to ${{ needs.sync.outputs.current_updated_at }}
    summary: other-repo was updated successfully.
    previous_updated_at: ${{ needs.sync.outputs.previous_updated_at }}
    current_updated_at: ${{ needs.sync.outputs.current_updated_at }}
    commit_sha: ${{ needs.sync.outputs.commit_sha }}
  secrets:
    SMTP_SERVER: ${{ secrets.SMTP_SERVER }}
    SMTP_PORT: ${{ secrets.SMTP_PORT }}
    SMTP_USERNAME: ${{ secrets.SMTP_USERNAME }}
    SMTP_PASSWORD: ${{ secrets.SMTP_PASSWORD }}
    MAIL_TO: ${{ secrets.MAIL_TO }}
    MAIL_FROM: ${{ secrets.MAIL_FROM }}
```

別リポジトリから使う場合も、secret の実体は呼び出し元リポジトリまたは Organization 側に置きます。
