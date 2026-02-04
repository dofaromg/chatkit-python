# ChatKit Python SDK

A Python backend SDK for building rich, interactive chat applications with OpenAI's ChatKit.

## Overview

ChatKit Python SDK provides the server-side components for building chat applications. It handles message streaming, thread management, store integration, and seamlessly integrates with OpenAI's Agents SDK for AI-powered responses.

## Features

- **Easy Integration**: Single endpoint handles all ChatKit requests
- **Flexible Storage**: Implement custom stores for any database (Postgres, MySQL, etc.)
- **Streaming Support**: Real-time message streaming with SSE (Server-Sent Events)
- **Agents SDK Integration**: Built-in helpers for OpenAI's Agents SDK
- **Type Safety**: Full type hints and runtime validation with Pydantic
- **Rich UI Components**: Support for widgets, annotations, and interactive elements

## Installation

```bash
pip install openai-chatkit
```

## Quick Start

### 1. Create a ChatKit Server

```python
from datetime import datetime
from typing import AsyncIterator

from fastapi import FastAPI, Request
from fastapi.responses import StreamingResponse, Response

from chatkit.server import ChatKitServer, StreamingResult
from chatkit.types import (
    AssistantMessageContent,
    AssistantMessageItem,
    ThreadItemDoneEvent,
    ThreadMetadata,
    ThreadStreamEvent,
    UserMessageItem,
)

app = FastAPI()

class MyChatKitServer(ChatKitServer[dict]):
    async def respond(
        self,
        thread: ThreadMetadata,
        input_user_message: UserMessageItem | None,
        context: dict,
    ) -> AsyncIterator[ThreadStreamEvent]:
        # Your response logic here
        yield ThreadItemDoneEvent(
            item=AssistantMessageItem(
                thread_id=thread.id,
                id=self.store.generate_item_id("message", thread, context),
                created_at=datetime.now(),
                content=[AssistantMessageContent(text="Hello, world!")],
            ),
        )

server = MyChatKitServer(store=MyChatKitStore())

@app.post("/chatkit")
async def chatkit(request: Request):
    result = await server.process(await request.body(), context={})
    if isinstance(result, StreamingResult):
        return StreamingResponse(result, media_type="text/event-stream")
    return Response(content=result.json, media_type="application/json")
```

### 2. Implement a Store

```python
from chatkit.store import Store, NotFoundError
from chatkit.types import ThreadMetadata, ThreadItem, Page

class MyChatKitStore(Store[dict]):
    async def load_thread(self, thread_id: str, context: dict) -> ThreadMetadata:
        # Load thread from your database
        pass
    
    async def save_thread(self, thread: ThreadMetadata, context: dict) -> None:
        # Save thread to your database
        pass
    
    # Implement other required methods...
```

### 3. Connect with Frontend

In your React app:

```tsx
import { ChatKit, useChatKit } from "@openai/chatkit-react";

export function App() {
  const chatkit = useChatKit({
    api: {
      url: "http://localhost:8000/chatkit",
      domainKey: "local-dev",
    },
  });

  return <ChatKit control={chatkit.control} />;
}
```

## Documentation

- **[Quick Start Guide](https://openai.github.io/chatkit-python/quickstart/)** - Get up and running in minutes
- **[Core Concepts](https://openai.github.io/chatkit-python/concepts/threads/)** - Understand threads, messages, and items
- **[Guides](https://openai.github.io/chatkit-python/guides/respond-to-user-message/)** - Step-by-step tutorials
- **[API Reference](https://openai.github.io/chatkit-python/api/chatkit/)** - Complete API documentation
- **[ChatKit.js Docs](https://openai.github.io/chatkit-js/)** - Frontend SDK documentation

## Key Guides

- [Respond to a User Message](https://openai.github.io/chatkit-python/guides/respond-to-user-message/)
- [Build Interactive Responses with Widgets](https://openai.github.io/chatkit-python/guides/build-interactive-responses-with-widgets/)
- [Handle User Feedback](https://openai.github.io/chatkit-python/guides/handle-feedback/)
- [Prepare Your App for Production](https://openai.github.io/chatkit-python/guides/prepare-your-app-for-production/)
- [Stream Generated Images](https://openai.github.io/chatkit-python/guides/stream-generated-images/)

## Examples

Check out the [starter app](https://github.com/openai/openai-chatkit-starter-app) for a complete working example with React frontend and Python backend.

## Requirements

- Python 3.10 or higher
- OpenAI API key (for Agents SDK integration)

## Development

### Install Dependencies

```bash
pip install -e ".[dev]"
```

### Run Tests

```bash
pytest
```

### Lint

```bash
ruff check .
ruff format .
```

## License

This project is licensed under the [Apache License 2.0](LICENSE).
