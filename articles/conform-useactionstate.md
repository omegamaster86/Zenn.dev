---
title: "Server Actionsに使用するConform（ライブラリ）とuseActionState"
emoji: "🗒️"
type: "tech" # tech: 技術記事 / idea: アイデア
topics: ["javascript", "ecmascript"]
published: false
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
- Progressive enhancement first APIs
- Type-safe field inference
- Fine-grained subscription
- Built-in accessibility helpers
- Automatic type coercion with Zod

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
Server Actionは@conform-to/zodのparseWithZodを利用することで以下のように書くことができます。
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

  redirect('/dashboard');
}
```

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