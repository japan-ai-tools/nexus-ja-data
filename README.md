# nexus-ja-data

<!-- mcp-name: io.github.japan-ai-tools/nexus-ja-data -->

**A local MCP server that lets AI agents clean Japanese business data with fixed rules instead of guessing.**

- Normalizes Japanese company names ((株) / ㈱ / 株式会社 / half-width kana), phone numbers, postal codes, emails, addresses and era dates (令和6年4月1日, R6.3.15 → 2024-04-01)
- Deduplicates Japanese customer lists (名寄せ). Flags similar-but-different companies (さくら商事 / さくら商会) for review instead of merging them
- Same input → same output. Returns the reason for each match
- Python standard library only. No network access. Data never leaves your machine
- MIT license

```json
{ "mcpServers": { "nexus-ja-data": { "command": "python", "args": ["/path/to/nexus-ja-data/run_server.py"] } } }
```

Tools: `ja_normalize_value`, `ja_same_company`, `ja_detect_column_types`, `ja_clean_and_dedupe_records`. See `llms.txt`.

---

# nexus-ja-data(日本語)

**AIエージェントのための、日本語ビジネスデータ整形・名寄せの道具(MCPサーバー)。**
ClaudeやChatGPTなどのAIに登録すると、AIが日本語の会社名・電話・住所・和暦を「推測」ではなく「決まったルール」で正確に直し、顧客名簿の重複をまとめられるようになる。

- Python標準ライブラリだけで動く(追加インストール不要)
- 外部通信なし。データはパソコンの外に出ない
- 名寄せ赤ペン(p004)と同じルール。JavaScript版と結果が一致することをテストで確認済み

## AIが使える道具(4つ)
| 名前 | できること |
|---|---|
| `ja_normalize_value` | 値1つを種類に合わせて直す(会社名・氏名・フリガナ・電話・郵便番号・メール・住所・日付) |
| `ja_same_company` | 2つの会社名が同じ会社かを判定((株)/株式会社/半角カナ/ひらがな の違いを吸収、似ている別会社は「別」) |
| `ja_detect_column_types` | 表の列の種類を推定 |
| `ja_clean_and_dedupe_records` | 名簿を正規化して重複をまとめる(名寄せ) |

## なぜAIに必要か
大きなAIでも、日本語の表記ゆれは毎回少しずつ違う直し方をしたり、「さくら商事」と「さくら商会」を同じ会社にしてしまったりする。この道具を使えば、同じ入力にいつも同じ答えを返し、判断の根拠(一致した条件・似ている度)も返す。

## 使い方(Claude Desktop の例)
設定ファイル `claude_desktop_config.json` に次を追加(パスは置いた場所に合わせる):
```json
{
  "mcpServers": {
    "nexus-ja-data": {
      "command": "python",
      "args": ["-m", "nexus_ja.server"],
      "cwd": "C:\\Users\\あなた\\Documents\\nexus-ja-data"
    }
  }
}
```
`cwd` が使えないアプリでは `"args": ["C:\\...\\nexus-ja-data\\run_server.py"]` を指定する。

MCP protocol: the stdio server speaks both the classic `initialize` flow (2025-06-18 and earlier) and the stateless 2026-07-28 revision (`server/discover`, per-request `_meta`). Tested in `tests/test_mcp_2026.py`.
MCP の新しい版(2026-07-28・server/discover)と従来の initialize の両方で動きます。

## 配布条件
MITライセンス(無料・商用利用可)。行数の制限なし。
人間が画面で使える同じルールの道具「名寄せ赤ペン」は BOOTH で販売中: https://nexus-tools.booth.pm/items/9051166

## 品質
- テスト11件(`python -m unittest discover -s tests`): 正規化の期待値、名寄せ、MCPとしての通し会話(初期化→一覧→呼び出し→エラー処理)、壊れた入力でも落ちない、外部通信コードなし、JavaScript版との一致
- 2026-10-27: 全件成功


## Related tool from the same author

- After you clean up a supplier list with this server, [invoice-watch](https://github.com/japan-ai-tools/invoice-watch) can compare that list with the NTA's published invoice-registration data and report only the suppliers whose registration was cancelled, expired or renamed.

関連(同じ作者): 取引先の一覧を名寄せで整えたら、[取引先インボイス見張り(invoice-watch)](https://github.com/japan-ai-tools/invoice-watch) がその一覧を国税庁の公表データと照らして、登録の取消・失効・名称の変化だけを知らせます。税務の助言ではありません(Not tax advice)。
