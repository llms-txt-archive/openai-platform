# MCP for Deep Research

> For the complete documentation index, see [llms.txt](/llms.txt). Markdown versions of documentation pages are available by appending `.md` to the page URL.

This is a minimal example of a Deep Research style MCP server for searching and fetching files from the OpenAI file storage service.

For a reference of _how_ to call this service from the Responses API, with Deep Research see [this cookbook](https://developers.openai.com/cookbook/examples/deep_research_api/introduction_to_deep_research_api). To see how to call the MCP server with the Agents SDK, checkout [this cookbook](https://developers.openai.com/cookbook/examples/deep_research_api/how_to_use_deep_research_api_agents)!

The Deep Research agent relies specifically on Search and Fetch tools. Search should look through your object store for a set of specfic, top-k IDs. Fetch, is a tool that takes objectIds as arguments and pulls back the relevant resources.

## Set up & run

Store your internal file(s) in [OpenAI Vector Storage](https://platform.openai.com/storage/vector_stores/)

Python setup:

```shell
python3 -m venv env
source env/bin/activate
pip install -r requirements.txt
```

Run the server:

```shell
python main.py
```

The server will start on `http://0.0.0.0:8000/sse/` using SSE transport. If you want to reach the server from the public internet, there are a variety of ways to do that including with ngrok:

```shell
brew install ngrok 
ngrok config add-authtoken <your_token>
ngrok http 8000
```

You should now be able to reach your local server from your client. 

## Files

- `main.py`: [Main server code](https://github.com/openai/openai-cookbook/blob/main/examples/deep_research_api/how_to_build_a_deep_research_mcp_server/main.py)

## Example Flow diagram for MCP Server

![/cookbook/assets/images/mcp_dr.png](https://developers.openai.com/cookbook/assets/images/mcp_dr.png)

## Example request

```python
# system_message includes reference to internal file lookups for MCP.
system_message = """
You are a professional researcher preparing a structured, data-driven report on behalf of a global health economics team. Your task is to analyze the health question the user poses.

Do:
- Focus on data-rich insights: include specific figures, trends, statistics, and measurable outcomes (e.g., reduction in hospitalization costs, market size, pricing trends, payer adoption).
- When appropriate, summarize data in a way that could be turned into charts or tables, and call this out in the response (e.g., "this would work well as a bar chart comparing per-patient costs across regions").
- Prioritize reliable, up-to-date sources: peer-reviewed research, health organizations (e.g., WHO, CDC), regulatory agencies, or pharmaceutical earnings reports.
- Include an internal file lookup tool to retrieve information from our own internal data sources. If you've already retrieved a file, do not call fetch again for that same file. Prioritize inclusion of that data.
- Include inline citations and return all source metadata.

Be analytical, avoid generalities, and ensure that each section supports data-backed reasoning that could inform healthcare policy or financial modeling.
"""

user_query = "Research the economic impact of semaglutide on global healthcare systems."

response = client.responses.create(
  model="o3-deep-research-2025-06-26",
  input=[
    {
      "role": "developer",
      "content": [
        {
          "type": "input_text",
          "text": system_message,
        }
      ]
    },
    {
      "role": "user",
      "content": [
        {
          "type": "input_text",
          "text": user_query,
        }
      ]
    }
  ],
  reasoning={
    "summary": "auto"
  },
  tools=[
    {
      "type": "web_search_preview"
    },
    { # ADD MCP TOOL SUPPORT
      "type": "mcp",
      "server_label": "internal_file_lookup",
      "server_url": "http://0.0.0.0:8000/sse/", # Update to the location of *your* MCP server
      "require_approval": "never"
    }
  ]
)