# Async tool calling

> For the complete documentation index, see [llms.txt](/llms.txt). Markdown versions of documentation pages are available by appending `.md` to the page URL.

Async tool calling lets the model continue working after it calls a tool, without waiting for that tool's result. Use it to start slow lookup requests early, answer independent parts of a request, and provide results when your application has them.

## How async tools work

A normal [function call](https://developers.openai.com/api/docs/guides/function-calling) pauses the model's turn to wait for a tool response. Set `async: true` on a function or custom tool definition to let the model continue working after issuing that call, before your application returns the output.

Your application still executes the tool. Async tools don't move execution to
  OpenAI or manage your background jobs.

This differs from [Background mode](https://developers.openai.com/api/docs/guides/background), which runs response generation asynchronously. Async tool calling lets the model continue working while your application runs a tool.

When a job finishes, include its output in a later Responses request. Use the original API `call_id` to match the result to its call:

| Tool type | Call item          | Output item               |
| --------- | ------------------ | ------------------------- |
| Function  | `function_call`    | `function_call_output`    |
| Custom    | `custom_tool_call` | `custom_tool_call_output` |

## Call an async tool

Add `async: true` to the tool definition. The corresponding call items in `response.output` include `async: true`.

Run a weather lookup in the background

```javascript
const model = process.env.OPENAI_MODEL ?? "gpt-6-astra";

const tools = [
  {
    type: "function",
    name: "get_weather",
    description: "Read a demo weather snapshot for a city.",
    async: true,
    strict: true,
    parameters: {
      type: "object",
      properties: { city: { type: "string" } },
      required: ["city"],
      additionalProperties: false,
    },
  },
];
const apiKey = process.env.OPENAI_API_KEY;
if (!apiKey) {
  throw new Error("Set OPENAI_API_KEY before running this example.");
}
const baseURL = (
  process.env.OPENAI_BASE_URL ?? "https://api.openai.com/v1"
).replace(/\/$/, "");

async function postResponse(body) {
  const response = await fetch(`${baseURL}/responses`, {
    method: "POST",
    headers: {
      Authorization: `Bearer ${apiKey}`,
      "Content-Type": "application/json",
    },
    body: JSON.stringify(body),
  });
  if (!response.ok) {
    throw new Error(
      `Responses API ${response.status}: ${await response.text()}`
    );
  }
  return response.json();
}

async function getWeather(city) {
  const snapshots = {
    Paris: {
      city: "Paris",
      temperature_c: 22,
      condition: "Clear",
      source: "demo weather snapshot",
    },
  };
  if (typeof city !== "string" || !Object.hasOwn(snapshots, city)) {
    throw new Error(`No demo weather snapshot for ${city}.`);
  }
  return snapshots[city];
}
const instructions =
  "Start the weather lookup and answer the independent packing question " +
  "without waiting. Use the demo weather result when it arrives; never invent it.";

let response = await postResponse({
  model,
  tools,
  instructions,
  input:
    "Check the demo weather snapshot for Paris. Meanwhile, " +
    "list three essentials for any city trip.",
});

const call = response.output.find(
  (item) => item.type === "function_call" && item.name === "get_weather"
);
if (!call) {
  throw new Error("The response did not include a weather call.");
}
const { city } = JSON.parse(call.arguments);
let latestResponseId = response.id;

// Calling an async function starts the application's job immediately.
const job = getWeather(city).catch((error) => ({ error: error.message }));
if (!call.async) {
  // Ordinary synchronous calls must finish before the model resumes.
  await job;
}
console.log(response.output);
// Independent work or conversation turns can happen here.
// Update latestResponseId after each continuation.
const result = await job;
response = await postResponse({
  model,
  tools,
  instructions,
  previous_response_id: latestResponseId,
  input: [
    {
      type: "function_call_output",
      call_id: call.call_id,
      output: JSON.stringify(result),
    },
  ],
});
latestResponseId = response.id;
console.log(response.output);
```

```python
import json
import os
from concurrent.futures import ThreadPoolExecutor
from urllib.request import Request, urlopen


def post_response(body):
    base_url = os.environ.get("OPENAI_BASE_URL", "https://api.openai.com/v1")
    request = Request(
        f"{base_url.rstrip('/')}/responses",
        data=json.dumps(body).encode(),
        headers={
            "Authorization": f"Bearer {os.environ['OPENAI_API_KEY']}",
            "Content-Type": "application/json",
        },
        method="POST",
    )
    with urlopen(request, timeout=120) as response:
        return json.load(response)


def get_weather(city):
    # Demo data. Replace this function with your weather service.
    weather = {
        "Paris": {
            "city": "Paris",
            "temperature_c": 22,
            "condition": "Clear",
            "source": "demo weather snapshot",
        }
    }
    return weather[city]


worker = ThreadPoolExecutor()


def main():
    model = os.environ.get("OPENAI_MODEL", "gpt-6-astra")
    tools = [
        {
            "type": "function",
            "name": "get_weather",
            "description": "Read the demo weather snapshot for a city.",
            "async": True,
            "strict": True,
            "parameters": {
                "type": "object",
                "properties": {"city": {"type": "string"}},
                "required": ["city"],
                "additionalProperties": False,
            },
        },
    ]

    instructions = (
        "Start the weather lookup and answer the independent packing "
        "question without waiting. Use the actual tool result when it "
        "arrives; never invent it. Identify the weather as demo data."
    )
    response = post_response(
        {
            "model": model,
            "tools": tools,
            "instructions": instructions,
            "input": (
                "Check the demo weather in Paris. Meanwhile, "
                "list three essentials for any city trip."
            ),
        }
    )

    call = next(item for item in response["output"] if item["type"] == "function_call")
    arguments = json.loads(call["arguments"])
    if call["name"] != "get_weather" or arguments != {"city": "Paris"}:
        raise ValueError("Expected a weather lookup for Paris")

    latest_response_id = response["id"]
    if call.get("async", False):
        job = worker.submit(get_weather, **arguments)
        print(response["output"])
        # Independent work or conversation turns can happen here.
        # Update latest_response_id after each continuation.
        result = job.result()
    else:
        result = get_weather(**arguments)

    response = post_response(
        {
            "model": model,
            "tools": tools,
            "instructions": instructions,
            "previous_response_id": latest_response_id,
            "input": [
                {
                    "type": "function_call_output",
                    "call_id": call["call_id"],
                    "output": json.dumps(result),
                },
            ],
        }
    )
    print(response["output"])


if __name__ == "__main__":
    try:
        main()
    finally:
        worker.shutdown(wait=True)
```


The response can contain both the async call and an answer. If other conversation turns happen before the job finishes, update `latest_response_id` to continue from the latest response while keeping the original tool `call_id`.

For earlier dispatch with [streaming](https://developers.openai.com/api/docs/guides/streaming-responses), start the job when its complete call item arrives while you continue consuming the response.

## Add a wait tool

A wait tool lets the model choose when it needs a pending result. For example, it can launch two price lookup requests, work on something independent, and wait only when it's ready to compare prices.

Add a `task_handle` argument to each async tool. The model assigns a handle to each call, and your application binds it to the original API `call_id` and the running job. Keep handles unique throughout the conversation, including completed tasks and repeated lookup requests.

Define the wait tool as an ordinary synchronous function: omit `async` or set it to `false`. Its schema and behavior belong to your application. `wait_for_tasks` isn't a built-in Responses tool.

Use these definitions in the request's `tools` array:

```json
[
  {
    "type": "function",
    "name": "lookup_price",
    "async": true,
    "description": "Look up a product price in the background. Choose a fresh task_handle unique within this conversation, including completed tasks.",
    "strict": true,
    "parameters": {
      "type": "object",
      "properties": {
        "sku": { "type": "string" },
        "task_handle": { "type": "string" }
      },
      "required": ["sku", "task_handle"],
      "additionalProperties": false
    }
  },
  {
    "type": "function",
    "name": "wait_for_tasks",
    "description": "Wait for selected tasks whose results you need. Pass a nonempty list of distinct task_handles from your earlier lookup_price calls. Results arrive on their original calls; this tool returns status only. Do not wait again for results that have already arrived.",
    "strict": true,
    "parameters": {
      "type": "object",
      "properties": {
        "task_handles": {
          "type": "array",
          "items": { "type": "string" }
        }
      },
      "required": ["task_handles"],
      "additionalProperties": false
    }
  }
]
```

### Register each job

Register and start each launch before processing a dependent wait. Calls can arrive together or across responses. The following illustrative output items show two launches and a wait that depends on both:

```json
[
  {
    "type": "function_call",
    "name": "lookup_price",
    "async": true,
    "call_id": "call_widget",
    "arguments": "{\"sku\":\"WIDGET\",\"task_handle\":\"widget_price_1\"}"
  },
  {
    "type": "function_call",
    "name": "lookup_price",
    "async": true,
    "call_id": "call_gadget",
    "arguments": "{\"sku\":\"GADGET\",\"task_handle\":\"gadget_price_1\"}"
  },
  {
    "type": "function_call",
    "name": "wait_for_tasks",
    "call_id": "call_wait",
    "arguments": "{\"task_handles\":[\"widget_price_1\",\"gadget_price_1\"]}"
  }
]
```

Your application's registry binds each handle to its original call and running job:

| Task handle      | Original call ID | Job                 |
| ---------------- | ---------------- | ------------------- |
| `widget_price_1` | `call_widget`    | WIDGET price lookup |
| `gadget_price_1` | `call_gadget`    | GADGET price lookup |

Keep the registry for the entire conversation to prevent reuse of a completed task's handle.

### Deliver results before wait status

Resolve the requested handles in the registry and await only those jobs. Return each newly completed result on its original `call_id`, then return status on the wait call's own `call_id`. This order gives the model the results when it resumes.

For example, send these output items in the next request's `input` array. The prices are illustrative:

```json
[
  {
    "type": "function_call_output",
    "call_id": "call_widget",
    "output": "{\"task_handle\":\"widget_price_1\",\"price_cents\":1200,\"currency\":\"USD\"}"
  },
  {
    "type": "function_call_output",
    "call_id": "call_gadget",
    "output": "{\"task_handle\":\"gadget_price_1\",\"price_cents\":1500,\"currency\":\"USD\"}"
  },
  {
    "type": "function_call_output",
    "call_id": "call_wait",
    "output": "{\"status\":\"completed\",\"completed_task_handles\":[\"widget_price_1\",\"gadget_price_1\"]}"
  }
]
```

Set `previous_response_id` to the latest response ID, and include the tools and instructions in the continuation request. Your application can also deliver results as they become available, without a wait call. Only use the wait tool when the model's next step depends on results that haven't arrived.

## Compatibility

Async tool calling is supported by GPT-6 Astra and later models.

Async execution applies to function and custom tools that your application runs. It doesn't apply to hosted built-in tools. Use direct tool calls; don't configure async tools for [programmatic tool calling](https://developers.openai.com/api/docs/guides/tools-programmatic-tool-calling).

In [Multi-agent mode](https://developers.openai.com/api/docs/guides/responses-multi-agent), don't combine async tools with parallel tool calls.