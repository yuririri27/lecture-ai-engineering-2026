# 第3回 解答

## 演習

- 第1回（素のLLM）とエージェントの回答の違い：第1回の素のLLMでは、「3レベル」について一般的な講座の難易度構成から「基礎・初級」「応用・中級」「発展・上級」の3段階であると推測して回答した。しかし、実際の講座で使われる名称や、いずれか1つに取り組めば宿題が完了するというルールは回答できなかった。
第3回のエージェントでは、「基礎・実装・発展」の3段階であり、いずれか1つに取り組めば宿題が完了すると正しく回答した。

このことから、素のLLMは講座固有の情報を知らない場合でも一般的な知識からもっともらしい回答を生成するのに対し、エージェントは外部の用語集を利用することで、講座固有の情報に基づいた回答が可能になると考えられる。

- `--trace` で見えたツール呼び出し（ツール名と引数）：「LoRAとQLoRAの違いは？」という質問では、次の2回のツール呼び出しを確認した。

search_glossary({'query': 'LoRA'})
search_glossary({'query': 'QLoRA'})
2つの用語を別々に検索し、取得した定義を基に違いを説明していた。

また、「第2回の内容から確認問題を1問出して」という質問では、make_quiz({'lecture': 2}) が呼び出され、LoRAの意味を問う問題が生成された。

- 1つの質問での LLM 呼び出し回数（`requests`）：
「LoRAとQLoRAの違いは？」の場合：
requests：2回
tool_calls：2回

「第2回の内容から確認問題を1問出して」の場合：
requests：2回
tool_calls：1回

- 追加したツールと、docstring に書いた説明：
app/tools.py に make_quiz(lecture: int) -> str を追加した。

指定された講義回の用語を load_glossary() から取得し、ランダムに1つ選択して、用語の意味を問う問題と答えをJSON形式で返す機能を実装した。
docstringには、指定した回の用語集から確認問題を1問作成することと、「第2回の確認問題を出して」のような復習用の問題を求められたときに使用することを記述した。

## 宿題

- 取り組んだレベル： 実装 
- やったこと：app/tools.py に、学習内容を learning_log/ フォルダに保存する save_learning_log ツールを実装した。
このツールは学習テーマと内容を引数として受け取り、Markdownファイルに保存する。
また、app/agent.py で requires_approval=True を設定し、ツールを実行する前に利用者の承認を求めるようにした。
DeferredToolRequests で承認待ちの要求を受け取り、ターミナルで利用者に y/n を確認した後、DeferredToolResults に承認結果を設定してエージェントを再開する処理を実装した。

- 結果：
ツール登録とテスト

--list-tools で4つのツールが登録されていることを確認した。
uv run pytest -q を実行し、11件すべてのテストが成功した。

保存を拒否した場合
LoRAの学習内容を保存するよう依頼し、承認確認で n を入力した。
--trace に The tool call was denied. と表示され、保存処理が拒否されたことを確認した。

保存を承認した場合
同じ質問で承認確認に y を入力した。

--trace に save_learning_log の呼び出しと保存成功のメッセージが表示され、learning_log/ 内のMarkdownファイルへの保存が報告された。

以下、結果のログ
@yuririri27 ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ uv run python -m app.agent --trace "LoRAは少数のパラメータを学習する手法だと学びました。この内容を学習記録に保存してください。"
[model] gemini-3.5-flash-lite

--- 保存の承認確認 ---
ツール名: save_learning_log
引数:
{
  "content": "LoRAは事前学習済みの重みを固定し、低ランク行列の積として表される更新分（少数のパラメータ）のみを学習するPEFT手法であると学んだ。",
  "topic": "LoRAの学習"
}

この操作を実行してよいですか？ (y/n): y
ご指定の内容を学習記録に保存しました。

LoRAは事前学習済みの重みを固定したまま、低ランクの小さな行列だけを効率的に学習する重要なPEFT手法ですね。

（関連する回：第2回）

--- trace ---
[tool call]   search_glossary({'query': 'LoRA'})
[tool return] search_glossary -> [{"term": "LoRA", "lecture": 2, "definition": "事前学習済みの重みは固定したまま、重みの更新分を2つの小さな行列の積（低ランク行列）で表し、その小さな行列だけを学習するPEFT手法。学習するパラ…
[tool call]   save_learning_log({'content': 'LoRAは事前学習済みの重みを固定し、低ランク行列の積として表される更新分（少数のパラメータ）のみを学習するPEFT手法であると学んだ。', 'topic': 'LoRAの学習'})
[tool return] save_learning_log -> 学習記録を保存しました: /workspaces/lecture-ai-engineering-2026/learning_log/20261106_022220_ec779cd4.md…
[usage] RunUsage(cost=Decimal('0.0004356'), details={'text_prompt_tokens': 1027}, input_text_tokens=1027, input_tokens=1027, output_tokens=51, requests=1, tool_calls=1)
@yuririri27 ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ ls -l learning_log/
total 4
-rw-rw-rw- 1 vscode vscode 241 Nov  6 02:22 20261106_022220_ec779cd4.md
@yuririri27 ➜ /workspaces/lecture-ai-engineering-2026 (lec03) $ cat learning_log/20261106_022220_ec779cd4.md
# LoRAの学習

記録日時: 2026-11-06 02:22:20

LoRAは事前学習済みの重みを固定し、低ランク行列の積として表される更新分（少数のパラメータ）のみを学習するPEFT手法であると学んだ。


- 気づき・疑問：LLMがツールの呼び出しを判断することと、実際にツールが実行されることは別であると分かった。
ファイルの保存のように外部の状態を変更する操作では、LLMが保存を要求しても、利用者が承認するまで実行を停止できることを確認した。
このような承認の仕組みは、ファイルの削除や変更など、取り消しにくい操作をAIエージェントに実行させる場合にも重要だと考えられる。
