---
title: "Next.16で改善されたキャッシュAPIを調査したんじゃ"
emoji: "✏️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["Next.js","cash"]
published: false
# published_at: 2025-11-01 21:30
publication_name: "genai"
---
# 初めに
気づいたら11月、今年ももう少しで終わりですね〜
どうでもいいんですけど、Fischer’sの増量中を会社でやりたいなと密かに思っています。（絶対にここで書く事ではないのだけど）
さて今回はNext.16で改善されたキャッシュAPI主にrevalidateTag・updateTag・refreshの使い分けや使い方を詳しく見ていこうと思います。

# revalidateTag：キャッシュデータのタグ無効化と再検証
revalidateTagは指定したタグに紐づくキャッシュデータをオンデマンドで「無効化（再検証対象にする）」する関数です。**主に多少の遅延が許容されるコンテンツ**に適しており、データをすぐ最新化する必要がないケース（ブログ記事、商品カタログ、ドキュメントなど）で利用するのが一般的です。
無効化後、ユーザーには現在のキャッシュ（古いデータ）が提供されつつ、バックグラウンドで新しいデータのフェッチ・更新が行われます。
### 使用できる場所
サーバーサイドのコンテキストである Server Action（サーバーアクション）や Route Handler（APIルート）内部で呼び出せます。クライアントコンポーネント内や直接ブラウザ上では使用できません。
### 使い方
デフォルトでは第二引数にキャッシュのライフタイムプロファイル`max`を指定して使用します。この`profile=max`を付けることで stale-while-revalidate が有効になり、**キャッシュを即座に削除せず「古いデータを見せながら裏で再取得」**する動作になります。ユーザーには遅延なく旧データが表示され、更新完了後に新データへ切り替わります。
第二引数を省略すると即時にキャッシュを無効化します（次のリクエスト時に強制的に再フェッチ・再レンダリングを行う）ですが、この一引数のみの使用法は非推奨になりました。Next.js 16では、即時再検証が必要な場合は後述のupdateTagへの移行が推奨されています。
タグ付きキャッシュデータを扱うため、あらかじめfetch関数のオプション`next: { tags: [...] }`でタグを付与するか、`'use cache'`指令を用いる関数内で`cacheTag('tagName')`を指定してデータにタグ付けしておく必要があります。
### SSRでの使用例
新規ユーザー情報更新後に該当データのタグを再検証（無効化）する例です。データ変更後に`revalidateTag('user', 'max')`を呼び出すことで、タグ"user"に関連するあらゆるページのキャッシュを stale 状態にし、次回アクセス時にバックグラウンドで最新データへ更新させます（ユーザーには旧データが瞬時に表示され続け、更新完了後に新データに差し替わります）。
こんな感じになるかな
```:ts
'use server';

import { revalidateTag } from 'next/cache';
import { updateUser } from '@/app/lib/user-store';

export type UpdateUserFormState = {
  status: 'idle' | 'success' | 'error';
  message: string;
  refreshedAt?: string;
};

const trimString = (value: FormDataEntryValue | null) => {
  return typeof value === 'string' ? value.trim() : '';
};

export async function updateUserAction(
  _prevState: UpdateUserFormState | undefined,
  formData: FormData,
): Promise<UpdateUserFormState> {
  const id = trimString(formData.get('id'));
  const name = trimString(formData.get('name'));
  const email = trimString(formData.get('email'));
  const bio = trimString(formData.get('bio'));
  const role = trimString(formData.get('role'));

  if (!id) {
    return {
      status: 'error',
      message: 'ユーザーIDが取得できませんでした。',
    };
  }

  const payload: Parameters<typeof updateUser>[1] = {};

  if (name) payload.name = name;
  if (email) payload.email = email;
  if (bio) payload.bio = bio;
  if (role === 'admin' || role === 'member') payload.role = role;

  if (Object.keys(payload).length === 0) {
    return {
      status: 'error',
      message: '変更内容を入力してください。',
    };
  }

  try {
    const result = await updateUser(id, payload);

    revalidateTag(`user:${id}`, 'max');

    return {
      status: 'success',
      message: 'ユーザー情報を更新しました。',
      refreshedAt: result.updatedAt,
    };
  } catch (error) {
    console.error(error);
    return {
      status: 'error',
      message: '更新処理でエラーが発生しました。',
    };
  }
}
```
個人的には`revalidateTag`を使用するなら後で記載する`updateTag`を使用するかなと思います。`revalidateTag`がデータをすぐに最新化する必要がない時に使用するとは書きましたが、最新の方がいいんじゃん？と思っています。

# updateTag：即時のキャッシュ無効化（Read-Your-Own-Writes 対応）
updateTagは Next.js 16で新たに導入された、**サーバーアクション内専用**のキャッシュ無効化関数です。特定のタグに紐づくデータのキャッシュを即座に期限切れにし、次のリクエストで必ず最新データを取得させるために使います。ユーザーがデータを投稿・更新した直後にその変更が画面に即反映される、「自分の行った変更を自分で直ちに確認できる」(read-your-own-writes)ための機能です。revalidateTagと異なり古いデータを見せる猶予を設けず、同じリクエスト中でデータを更新・再取得してUIに反映する動作を行います。
### 使用できる場所
くどいですが**Server Action内でのみ**利用可能です。Route Handler（APIルート）やクライアント側から呼ぶことはできず、もしServer Action外で呼び出すとエラーになります。Server Action内で動作させることで、Next.jsはキャッシュをただ無効にするだけでなく必要な再フェッチも即座に実行し、画面を更新します。
### 使い方
指定したタグのキャッシュエントリを即時に期限切れ（expired）状態にします。これにより次回そのタグ付きデータを要求する際は必ず新しいデータの取得が行われ、古いキャッシュは使用されません。言い換えれば、updateTag実行後の最初の再レンダリングは必ずブロッキングして最新データを取りに行き、ユーザーには更新済みの内容が表示されます。そのためユーザーのフォーム送信や設定変更など即時反映が求められるUI操作に向いています。
