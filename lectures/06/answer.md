# 第6回 解答

## 演習

<!-- [tool call] の行は、表示された行をそのまま貼り付けてください。終わらなかった STEP は、空欄のままでかまいません。 -->

- STEP 1 のツールの名前：glossary_search_glossary
- STEP 2 の [tool call]：[tool call]   glossary_search_glossary({'query': '3レベル'})
- STEP 3 の [tool call]：[tool call]   fetch_fetch({'url': 'https://modelcontextprotocol.io/'})
- STEP 4 の [tool call]：[tool call]   aws_aws___search_documentation({'topics': ['general'], 'search_phrase': 'Amazon S3 バージョニング 概要'})
[tool call]   aws_aws___search_documentation({'topics': ['general'], 'search_phrase': 'Using versioning in S3 buckets overview'})
- 気づき・疑問：STEP 3とSTEP 4では、どちらも英語の情報を取得しているにもかかわらず、ツールの呼び出し回数が異なることに疑問を持った。そこで、質問言語の変更やURLの直接指定による比較を行った。その結果、同じ質問でも検索キーワードや呼び出し回数が変化することが分かった。検索キーワードの生成方法や情報取得方法を適切に指定することで、呼び出し回数やトークン消費量を削減できるのではないかと考えた。

## 宿題

<!-- 基礎・実装・発展のどれか1つに取り組めば完了です。 -->

- 取り組んだレベル：基礎 
- やったこと：app/mcp_server.pyにserver.tool()(tools.get_lecture_summary)を追加し、既存の講義概要取得関数をMCPツールとして公開した。
- 結果：利用可能なツールが7個から8個に増加した。「第2回では何を学びますか？」と質問したところ、glossary_get_lecture_summary({'lecture': 2})が呼び出され、講義概要を取得できた。また、pytestでは21件すべてのテストに合格した。
- 気づき・疑問：エージェント側のプログラムを変更せず、MCPサーバー側に1行追加するだけで、新しいツールを利用できることが分かった。また、モデルがツールの説明文や引数の情報を基に、質問内容に適したツールを選択できることを確認した。
