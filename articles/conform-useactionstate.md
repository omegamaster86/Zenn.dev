---
title: "Server Actionsに使用するConform（ライブラリ）とuseActionState"
emoji: "🗒️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "ecmascript"]
published: true
published_at: 2025-01-27 11:00
---

# 初めに
さて皆さん、インフルエンザが猛威を振るっておりますが、体調はいかがですか？
私は年末にA型インフルエンザにかかり、しんどい思いをしていました。
体調管理には気をつけてください！

# 今回書く内容と現状のPJ
皆さんはNext.jsのServer Actionsを使用していますか？
私は最近App RouterのPJ（去年の開発の続きのPJ）に参画しているのですが、
useSWRでデータ取得/更新をしたり、useEffectでデータ取得/更新しているPJです…
useEffectでデータ取得/更新は皆さんしていないと思いますが、useSWRでデータ取得/更新と言われて「おいおい！」
と言える方は何人いらっしゃいますか？
正直私も「そういや昔使っていたな〜」と思いながら開発をしていました。
しかし、App Routerを使用しているのにServer Actionsをしていないとは何事だ！
と思ったので、今回はServer Actionsを使用してデータ取得/更新をする方法を紹介します。

# そもそもServer Actionsとは
「バカにするな〜」という声が聞こえてきそうですが、私の復習のためにもお付き合いください！
一言で言うと「Server Actionsは、サーバー上で実行される非同期関数です。 Next.jsアプリケーションのフォーム送信やデータ変異を処理するために、サーバーコンポーネントやクライアントコンポーネントで呼び出すことができます。」
https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations

って言われてもどんな感じ？
となると思うので、実際はこんな感じです！
typeが二重定義になっているので、良い子の方はtypesフォルダにまとめましょう！

```:data.tsx
"use server";

type Product = { title: string };

export async function getRandomProduct(): Promise<Product> {
  const res = await fetch(
    `https://dummyjson.com/products/${Math.floor(Math.random() * 100) + 1}`,
  );
  const json = (await res.json()) as Product;
  return json;
}
```

```:page.tsx
"use client";

import { useActionState } from "react";
import { getRandomProduct } from "./data";

type Product = { title: string };

export default function Home() {
  const [product, dispatch, isPending] = useActionState(async () => {
    const product: Product = await getRandomProduct();
    return product;
  }, null);

  return (
    <div className="p-4">
      <form action={dispatch}>
        <button
          type="submit"
          disabled={isPending}
          className="bg-blue-500 px-4 py-2 text-white hover:bg-blue-700 disabled:opacity-50"
        >
          商品を取得する
        </button>
      </form>
      {product ? (
        <div className="mt-4 w-80 rounded-lg border bg-white p-6 shadow-lg">
          <h2 className="mb-4 text-2xl font-bold text-gray-800">
            {product.title}
          </h2>
        </div>
      ) : (
        <p>データ取得に失敗しました</p>
      )}
    </div>
  );
}
```

ここでの注目ポイントはデータ取得用の関数を`data.tsx`で記載して、クライアントではデータ取得用の関数を呼び出すだけです。
https://react.dev/reference/react-dom/hooks/useFormState

# じゃあ本題
formアクションなら[react-hook-form](https://react-hook-form.com/)と[zod](https://zod.dev/)で十分だと思われると思います。
しかし残念ながら、react-hook-formではServer Actionsをサポートしているか公式に書いてありませんでした。
（見逃していたら、教えていただけると幸いです）
やっている方はいらっしゃるようです。
https://medium.com/@ctrlaltmonique/how-to-use-react-hook-form-zod-with-next-js-server-actions-437aaca3d72d
react-hook-formでServerActionsをサポートしていなかった場合下記にお進みください。

# ServerActions対応なformライブラリのconform
[conform公式](https://ja.conform.guide/)に記載してあるの特徴
- プログレッシブエンハンスメント・ファーストな API
- 型安全なフィールド推論
- きめ細やかなサブスクリプション
- 組み込みのアクセシビリティ・ヘルパー
- Zod による自動型強制

ありがたいことに公式サイトにNext.jsとconformの実装例があります。
ここからは公式サイトのコードを拝借しながらやっていきたいと思います。
ソースコードもありました。
https://github.com/edmundhung/conform/tree/main/examples/nextjs

### インストール
zodを使う場合、以下2つをインストールします。
```
yarn add @conform-to/react @conform-to/zod
```

```:schema.ts
import { z } from 'zod';

export const loginSchema = z.object({
  email: z.string().email(),
  password: z.string(),
  remember: z.boolean().optional(),
});
```
Server Actionは@conform-to/zodのparseWithZodを利用することで下記のように記載できます。
parseWithZod関数を使用すると、Zodスキーマを使用してFormDataを解析し、フォームの入力値をバリデーションできます。
```:action.ts
'use server';

import { redirect } from 'next/navigation';
import { parseWithZod } from '@conform-to/zod';
import { loginSchema } from '@/app/schema';

export async function login(prevState: unknown, formData: FormData) {
  const submission = parseWithZod(formData, {
    schema: loginSchema,
  });

  if (submission.status !== 'success') {
    return submission.reply();
  }
  // データベース更新系の処理もここに記載可能
  // データ不整合の場合とかは下記のようにエラーメッセージを返せます。
  return submission.reply({
    formErrors: ["エラーメッセージ1", "エラーメッセージ2"],
  });

  redirect('/dashboard');
}
```
useFormStateは、フォーム送信結果やサーバーサイドで処理されたアクションの状態（成功・失敗など）を管理するためのフックです。
下記のコードでは、loginアクションの送信結果を取得し、その状態をlastResultとして保持しています。
フォームの送信が完了すると、lastResultが更新され、結果（エラーや成功メッセージ）が反映されます。
`lastResult`:サーバーアクションの結果（例: エラーメッセージや成功レスポンス）を保持。
`action`:サーバーアクション（login）をトリガーするためのaction属性値。

useFormは、フォームの状態管理やバリデーションロジックをクライアントサイドで実装するためのフックです。このフックを利用することで、フィールドの状態やエラーメッセージを動的に管理できます。
今回での挙動
フォームの状態を生成
useFormが返すformオブジェクトには、フォーム全体の状態やイベントハンドラ（例: onSubmit）が含まれています。

フィールドごとの状態管理
fieldsオブジェクトには、フォーム内の各フィールドのキー、名前、初期値、エラーメッセージが含まれています。

クライアントサイドバリデーション
onValidateで、parseWithZodを使ったZodスキーマバリデーションを実行。
バリデーション結果がエラーの場合、エラー情報がfieldsに反映されます。

再バリデーションのタイミング
`shouldValidate: 'onBlur'`: フィールドをフォーカスアウトしたときにバリデーションを実行。
`shouldRevalidate: 'onInput'`: 入力値が変更されたときに再バリデーションを実行。


```:form.tsx
'use client';

import { useForm } from '@conform-to/react';
import { parseWithZod } from '@conform-to/zod';
import { useFormState } from 'react-dom';
import { login } from '@/app/actions';
import { loginSchema } from '@/app/schema';

export function LoginForm() {
  const [lastResult, action] = useFormState(login, undefined);
  const [form, fields] = useForm({
    lastResult,

    // クライアントでバリデーション・ロジックを再利用する
    onValidate({ formData }) {
      return parseWithZod(formData, { schema: loginSchema });
    },

    shouldValidate: 'onBlur',
    shouldRevalidate: 'onInput',
  });

  return (
    <form id={form.id} onSubmit={form.onSubmit} action={action} noValidate>
      <div>
        <label>Email</label>
        <input
          type="email"
          key={fields.email.key}
          name={fields.email.name}
          defaultValue={fields.email.initialValue}
        />
        <div>{fields.email.errors}</div>
      </div>
      <div>
        <label>Password</label>
        <input
          type="password"
          key={fields.password.key}
          name={fields.password.name}
          defaultValue={fields.password.initialValue}
        />
        <div>{fields.password.errors}</div>
      </div>
      <label>
        <div>
          <span>Remember me</span>
          <input
            type="checkbox"
            key={fields.remember.key}
            name={fields.remember.name}
            defaultChecked={fields.remember.initialValue === 'on'}
          />
        </div>
      </label>
      <button>Login</button>
    </form>
  );
}
```

# 最後に
この記事が誰かの役にたてば幸いです。