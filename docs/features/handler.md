# Handler

Handlerは基本となる機能で、TransformerやExtractorを登録してリッチエディタのHTMLコンテンツを処理します。

## Type Definition

```ts
import * as cheerio from "cheerio/slim";
import type { Extractor } from "./extractor/types";
import type { Transformer } from "./transformer/types";

type DefaultGenerics = Record<
 string,
 [
  extractor: Extractor<unknown>,
  options?: MicroCMSRichEditorHandlerExtractorOptions,
 ]
>;

type MicroCMSRichEditorHandlerExtractorOptions = {
 phase?: "before" | "after" | undefined;
};
type MicroCMSRichEditorHandlerOptions<T extends DefaultGenerics> = {
 transformers?: Transformer[] | undefined;
 extractors?: T | undefined;
};
type MicroCMSRichEditorHandlerReturn<T extends DefaultGenerics> = {
 html: string;
 data: {
  [K in keyof T]: T[K][0] extends Extractor<infer R> ? R : never;
 };
};

export const microCMSRichEditorHandler = async <T extends DefaultGenerics>(
 html: string,
 options: MicroCMSRichEditorHandlerOptions<T>,
): Promise<MicroCMSRichEditorHandlerReturn<T>>
```

extractorsに指定したキーに対応するするようにdataの型を決定するために少し複雑になっていますが、基本的には以下のように使います。

```js
import {
 microCMSRichEditorHandler,
 responsiveImageTransformer,
 tocExtractor
} from 'microcms-rich-editor-handler';

const main = async () => {
 const { html, data } = await microCMSRichEditorHandler(
  dataFromMicroCMS, // MicroCMSから取得したデータのリッチエディタのHTML文字列
  {
   transformers: [responsiveImageTransformer()],
   extractors: {
    toc: [tocExtractor(), { phase: "before" }],
   },
  },
 );
};
```

## Parameters

### options.transformers

第一引数に指定したHTML文字列を変換する関数の配列です。変換処理は配列に指定した順番に実行されるため、同じ要素に対して複数の変換処理を行う場合は順番に注意してください。

### options.extractors

第一引数に指定したHTML文字列からデータを抽出する関数のオブジェクトです。キーには抽出したいデータの名前を指定し、値には関数の配列を指定します。  
配列の１つ目の要素には抽出処理を行う関数を指定し、２つ目の要素には抽出処理を行うタイミングを指定します。

行うタイミングは`before`、`after`のいずれかを指定します。指定しない場合は`after`がデフォルト値として適用されます。  
このタイミングはTransformerの処理が実行される前に抽出処理を行うか、後に行うかを指定します。データの抽出処理がHTMLの変換処理の影響を受ける場合は`before`を指定してください。
