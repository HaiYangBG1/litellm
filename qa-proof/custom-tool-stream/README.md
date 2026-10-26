# Custom tool input stream proof

These captures show an interactive API inspection client consuming live Responses streams from two isolated source checkouts. They are not Codex desktop or LiteLLM Admin UI captures

Before is `dddbaa803d6e9fcc862ea7a5412f771cb1ae040e`. After is `4c905957e5b24b40df74194fa9334688385c9929`

Both runs use the same request and a real Anthropic-compatible upstream. No provider responses were mocked. The latest pair is shown, with a second successful repetition recorded in `verification.json`

Before returns HTTP 200 with a completed custom tool item, but emits 10 function argument deltas and no custom input events. After returns one custom input delta and one custom input done event, with the assembled input matching the completed item

The correction buffers the wrapped arguments until completion, then unwraps the input. This is not incremental JSON parsing

`before-response.json` and `after-response.json` retain the observed event fields. Credentials and upstream addresses are omitted

## Reproduce the API request

Configure `bridged-haiku` as a chat-backed model on the proxy and use your own gateway key. Save `request.json` beside this command

```sh
printf 'Authorization: Bearer %s\nContent-Type: application/json\n' "$LITELLM_MASTER_KEY" |
  curl --no-buffer --silent --show-error --header @- \
    --data-binary @request.json http://localhost:4000/v1/responses
```

Listen for `response.custom_tool_call_input.delta` and `response.custom_tool_call_input.done`. Their concatenated input should match the completed `custom_tool_call.input`

The 18 regression cases also cover tool names arriving after IDs and initial argument fragments. The two affected unit test files pass all 439 cases, and all 53 changed executable statements are covered
